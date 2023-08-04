---
title: "Denoのモジュールをdntでnpmパッケージとして、お手軽に公開しよう"
emoji: "🦖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["deno", "npm"]
published: false
publication_name: "cybozu_frontend"
---

## 書きたいこと

- Deno で実装したモジュールが、CJS、ESM、TypeScript に対応した npm パッケージとして公開できる
  - Deno で実装したモジュールに対して、[dnt](https://github.com/denoland/dnt) を使ってビルドするとできる
  - import 時に`npm:`や esm.sh を使ってる場合は、依存関係に追加される
- Deno だと Linter、Formatter、Test のライブラリ入れなくて良い
- Deno のグローバルなオブジェクトとかは shim を使うことで対応
- ビルドした npm パッケージの ESM、CJS 両方で Node.js でテストを実行してくれる
- ビルド後のパッケージのタイプチェックもしてくれる
- GitHub Action で CICD 作ってみて、改めて Deno は楽でいいな〜
- ビルド後のファイルからどんな感じに Dual Packages になってるのか調べる
  - 生成されたファイルの type しか記載されていない package.json とか

## 3 行まとめ

## Deno のモジュールを npm パッケージに変換する`dnt`

https://deno.com/blog/publish-esm-cjs-module-dnt

Deno の[`dnt`](https://github.com/denoland/dnt)モジュールを使うと、Deno で実装したモジュールを CommonJS(CJS) と ESM、TypeScript に対応した npm パッケージとして公開できます。

実際、今回 Deno と`dnt`を試してみましたが、Deno には Linter、Formatter、Test などが組み込まれているので、すぐに npm パッケージの作成に取り掛かれるのが良かったです。また、CICD の自動化(GitHub Actions)もとてもシンプルに実装できた印象がありました。

また、`dnt`ではビルド時に生成された CJS、ESM のファイルに対して、それぞれ Node.js でテストを実行してくれたりもします。

## 実際にモジュールを npm に公開する

実際に Deno と`dnt`を使って、npm パッケージを作成してみましょう。

次のリポジトリに今回実装したコードがあります。

https://github.com/nus3/deno-npm-package

### Deno でモジュールを実装

まず、[`cowsay`](https://www.npmjs.com/package/cowsay)を使ったシンプルなモジュールを作ります。

```ts
import cowsay from "npm:cowsay@1.5.0";

export function cowsayNus3() {
  return cowsay.say({ text: "Hello nus3!" });
}
```

このモジュールでは、[`npm:`specifiers](https://deno.land/manual@v1.36.0/node/npm_specifiers)を使って`cowsay`パッケージをインポートしています。`npm:`specifiers や[Skypack](https://www.skypack.dev/)、[esm.sh](https://esm.sh/)経由で使用されるパッケージは、`dnt`によって次のように依存関係に追加されます。

```json:package.json
{
  "dependencies": {
    "cowsay": "1.5.0"
  }
}
```

### ビルドして npm パッケージを作成

次に、`dnt`を使って Deno で実装したモジュールを npm パッケージとしてビルドします。

```ts
import { build, emptyDir } from "https://deno.land/x/dnt@0.37.0/mod.ts";

await emptyDir("./npm");

await build({
  entryPoints: ["./mod/index.ts"],
  outDir: "./npm",
  shims: {
    deno: true,
  },
  package: {
    name: "@nus3/cowsay-nus3",
    version: Deno.args[0],
    description: "String function that returns where the cow says Hello nus3",
    license: "MIT",
    repository: {
      type: "git",
      url: "git+https://github.com/nus3/deno-npm-package.git",
    },
    bugs: {
      url: "https://github.com/nus3/deno-npm-package/issues",
    },
  },
  postBuild() {
    Deno.copyFileSync("LICENSE", "npm/LICENSE");
    Deno.copyFileSync("README.md", "npm/README.md");
  },
});
```

### GitHub Action で CICD を整える

## あとがき

- エコシステムが用意された状態で、簡単に npm パッケージを作れる
- npm の互換性も上がってる
- cjs、esm 両方でテストしてくれる
- 使うしかない

```

```
