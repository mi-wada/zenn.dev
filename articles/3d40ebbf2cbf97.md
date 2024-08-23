---
title: "active_model_serializersをRails標準のActiveModel::Serializers::JSONで置きかえる"
emoji: "🥞"
type: "tech"
topics:
  - "rails"
  - "ruby"
published: true
published_at: "2024-02-02 19:16"
---

## 動機

[active_model_serializers](https://github.com/rails-api/active_model_serializers)は十分にメンテナンスされていないので、乗り換え先を探していた。しかし、あまり良さげな乗り換え先がなかった。

そんな時以下の記事で、Railsが標準で実装している[`ActiveModel::Serializers::JSON`](https://github.com/rails/rails/blob/1c2529b9a6ba5a1eff58be0d0373d7d9d401015b/activemodel/lib/active_model/serializers/json.rb)の存在を知った。なのでこれに乗り換えることにした。

https://product.st.inc/entry/2023/12/22/170502

## 要件

* active_model_serializersと同じくらいの書き心地を目指す
  * とはいえ`has_many`とかは妥協する

## 1. `BaseSerializer`を実装

各`Serializer`が継承する基底クラスを実装する。

```rb
class BaseSerializer
  include ActiveModel::Serializers::JSON

  def initialize(*_args)
    # ここでkeyに対応するmethodが実装されていることを求められるので、各インスタンス変数のattr_readerを定義。（もっと簡単に書く方法が知りたい）
    # https://github.com/rails/rails/blob/1c2529b9a6ba5a1eff58be0d0373d7d9d401015b/activemodel/lib/active_model/serialization.rb#L172-L176
    define_attr_readers
  end

  private

  def define_attr_readers
    instance_variable_names.each do |instance_variable_name|
      self.class.send(:attr_reader, instance_variable_name) unless self.class.method_defined?(instance_variable_name)
    end
  end

  # @return [Array<String>] all instance variable names
  def instance_variable_names
    instance_variables.map { |ivar| ivar.to_s.delete_prefix('@') }
  end

  # override `ActiveModel::Serialization#attribute_names_for_serialization`
  # https://github.com/rails/rails/blob/1c2529b9a6ba5a1eff58be0d0373d7d9d401015b/activemodel/lib/active_model/serialization.rb#L152-L154
  def attribute_names_for_serialization
    instance_variable_names
  end
end
```

<details>
  <summary>テストコード</summary>

```rb
RSpec.describe BaseSerializer do
  let(:klass) do
    Class.new(described_class) do
      def initialize(record1:, record2:)
        @record1 = Record1Serializer.new(record1)
        @record2 = Record2Serializer.new(record2)

        super
      end

      class Record1Serializer < BaseSerializer
        def initialize(record1)
          @str = record1[:str]
          @number = record1[:number]

          super
        end
      end

      class Record2Serializer < BaseSerializer
        def initialize(record2)
          @str = record2[:str]
          @bool = record2[:bool]

          super
        end
      end
    end
  end

  it do
    expect(
      klass.new(
        record1: { str: 'str1', number: 1 },
        record2: { str: 'str2', bool: true }
      ).as_json
    ).to eq(
      {
        'record1' => { 'str' => 'str1', 'number' => 1 },
        'record2' => { 'str' => 'str2', 'bool' => true }
      }
    )
  end
end

```

</details>

## 2. `ActiveModel::Serializer`を`BaseSerializer`で置き換える

### Before

```rb
class UsersController
  def show
    users = User.find(id)
    render json: user
  end
end
```

```rb
class UserSerializer < ActiveModel::Serializer
  attribute :id
  attribute :name
end
```

### After

```rb
class UsersController
  def show
    user = User.find(id)
    render json: UserSerializer.new(user)
  end
end
```

```rb
class UserSerializer < BaseSerializer
  # @param [User] user
  def initialize(user)
    @id = user.id
    @name = user.name

    super
  end
end
```

## おわり

* そこまで書き心地を損なわずに移行できた
* 依存を一つ減らせたので満足
* 一部エンドポイントでは少しだけ早くなった
