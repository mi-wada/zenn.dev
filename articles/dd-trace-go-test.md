---
title: "dd-trace-goのmocktracerはテスト時に便利"
emoji: "🐶"
type: "tech"
topics: ["Go", "Datadog", "Test"]
published: true
---

## はじめに

自動テスト時に動かすtracerを探していたところ[mocktracer](https://pkg.go.dev/gopkg.in/DataDog/dd-trace-go.v1/ddtrace/mocktracer)を見つけたのでそれを紹介する。

そのようなtracerを探すに至った経緯はこちら（読み飛ばして良い）：
dd-trace-goの[v1.72.0](https://github.com/DataDog/dd-trace-go/releases/tag/v1.72.0)のリリースでtracerが起動していない場合`tracer.SpanFromContext`の第二返り値はfalseが返るようになった。
> There is a minor behaviour breaking change in tracer.SpanFromContext. Now, it also returns false in the second return value when the span from the context is a no-op span, which only happens when spans are created without a tracer being started (previously it used to return true). This change shouldn't have any effect in applications that correctly start the tracer before creating spans.

それに伴いいくつかのテストが落ちたので、テスト時もtracerを動かす必要があった。

## 結論

テスト時は[mocktracer](https://pkg.go.dev/gopkg.in/DataDog/dd-trace-go.v1/ddtrace/mocktracer)が使える。mocktracerならばtraceの送信などはスキップされる。以下のように使う。

```go
func TestXxx(t *testing.T) {
  mt := mocktracer.Start()
  defer mt.Stop()

  // spanを必要とするロジックのテストを書く
}
```

`TestMain`などで起動しておくのも便利。

```go
func TestMain(m *testing.M) {
  mt := mocktracer.Start()
  exitCode := m.Run()
  mt.Stop()

  os.Exit(exitCode)
}
```

## おわり

Datadogやotel周りのGo packageはメジャーバージョンアップでなくともガンガンBREAKING CHANGEが入る印象がある。
