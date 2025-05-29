---
title: "VS Codeで自動でtype-only importする"
emoji: "🛵"
type: "tech"
topics: ["TypeScript", "VSCode"]
published: true
---

## 結論

以下の設定を`settings.json`に追加する。

```jsonc
{
  // ...
  "typescript.preferences.preferTypeOnlyAutoImports": true,
  // ...
}
```

これにより型をimportする際以下のように`type`キーワードが自動で付与される。

```typescript
import type { User } from './user';
```

## 背景

Biomeのrecommendedなルールとして[useImportType](https://biomejs.dev/linter/rules/use-import-type/)があり、型をimportする際はtype-only importしないと怒られる。なおBiomeのドキュメントでもこの設定が推奨されている。
> You may also want to enable the editor setting typescript.preferences.preferTypeOnlyAutoImports from the TypeScript LSP. This setting is available in Visual Studio Code. It ensures the type is used when the editor automatically imports a type.

`typescript.preferences.preferTypeOnlyAutoImports`はVS Code 1.85（2023年11月リリース）で追加されている。
<https://code.visualstudio.com/updates/v1_85#_prefer-using-type-for-auto-imports>

type-only importはTypeScript 3.8で追加されている。type-only importすることで、型のみに使用されるインポートがコンパイル時にJavaScriptコードから完全に除去され、バンドルサイズが削減される。
<https://devblogs.microsoft.com/typescript/announcing-typescript-3-8/#type-only-imports-and-exports>

昔はESLintの`@typescript-eslint/consistent-type-imports`を用いてtype-only importするのが一般的だったっぽい。
<https://qiita.com/derasado/items/20da03ea0d5dd3fceb07>
