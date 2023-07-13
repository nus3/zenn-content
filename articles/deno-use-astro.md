---
title: "Deno v1.35でAstroが使えるように"
emoji: "🦕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["deno", "astro"]
published: true
publication_name: "cybozu_frontend"
---

## 3 行まとめ

- Deno v1.35 で npm と Node の互換性がさらに向上
- `npm:astro` を使って Deno で Astro が使える
- Astro の Server⁠-⁠side Rendering の機能ももちろん使える

## Deno v1.35 のリリース

https://deno.com/blog/v1.35

Deno v1.35 のリリースし、その中には npm と Node の互換性の向上が含まれています。

> This month we made great improvements to compatibility of http, https and zlib modules.

これらモジュールの互換性の向上により、`npm:astro` が Deno で使えるようになりました。

## Deno で Astro を使ってみよう

実際に執筆時点(2023/07/13)で latest である `astro` v2.8.1 を Deno で使ってみましょう。

次のリンクからデモを確認できます。
[https://nus3-deno-astro.deno.dev/](https://nus3-deno-astro.deno.dev/)

また、[nus3/deno-astro](https://github.com/nus3/deno-astro)でコードも確認できます。

### VSCode で Deno を書くための準備をする

必須ではないですが、エディタに VSCode を使っている場合は、Deno 用の `settings.json` と `extensions.json` を追加しておくと、Deno の Linter や Formatter などの設定をわざわざしなくても良くなります。

```json: extensions.json
{
  // ついでにAstroの拡張も
  "recommendations": ["denoland.vscode-deno", "astro-build.astro-vscode"]
}
```

```json: settings.json
{
  "deno.enable": true,
  "deno.lint": true,
  "editor.defaultFormatter": "denoland.vscode-deno"
}
```

### Astro を使う

Deno v1.35 時点では、npm パッケージを利用するには以下の方法が[ドキュメント](https://deno.land/manual@v1.35.1/node)には記載されています。

> - Using `npm:` specifiers and `node:` specifiers
> - package.json compatibility
> - Using CDNs

今回は、ブログでのリリース内容にもある、`npm:` specifiers (`npm:astro`)を使って `astro` パッケージをインストールします。

#### `deno task` を定義

まず、Astro を起動するのに必要な `deno task` を `deno.json` に定義します。

```json:deno.json
{
  "tasks": {
    "dev": "deno run -A --node-modules-dir npm:astro@2.8.1 dev",
    "start": "deno run -A --node-modules-dir npm:astro@2.8.1 dev",
    "build": "deno run -A --node-modules-dir npm:astro@2.8.1 build",
    "preview": "deno run -A --node-modules-dir npm:astro@2.8.1 preview",
    "astro": "deno run -A --node-modules-dir npm:astro@2.8.1"
  }
}
```

`npm:astro@2.8.1` で `astro` の v2.8.1 を使用すること指示し、`--node-modules-dir` フラグを使って npm パッケージが node_modules ディレクトリから実行するようにします。

参考: [`--node-modules-dir` flag](https://deno.land/manual@v1.35.1/node/npm_specifiers#--node-modules-dir-flag)

#### `astro` パッケージを import している箇所に `npm:` specifiers を追記

以下のファイルで `astro` を import しているので `npm:` specifiers を追記します。

- astro.config.mjs
- src/env.d.ts

```js: astro.config.mjs
import { defineConfig } from "npm:astro@2.8.1/config";

export default defineConfig();
```

```ts: src/env.d.ts
/// <reference types="npm:astro@2.8.1/client" />
```

#### `index.astro` を追加

表示する画面を [`src/pages/index.astro`](https://github.com/nus3/deno-astro/blob/main/src/pages/index.astro)で追加し、`deno task dev` を実行することで localhost で Astro が起動します。

```bash
❯ deno task dev
Task dev deno run -A --node-modules-dir npm:astro@2.8.1 dev
  🚀  astro  v2.8.1 started in 61ms

  ┃ Local    http://localhost:3000/
  ┃ Network  use --host to expose
```

### Astro で Server⁠-⁠side Rendering し、Deno Deploy にデプロイする

Astro には [Server⁠-⁠side Rendering](https://docs.astro.build/en/guides/server-side-rendering/) の機能があり、また、[デプロイ先として Deno Deploy を選択](https://docs.astro.build/en/guides/deploy/deno/)できます。

せっかくなので Deno Deploy にデプロイしつつ、Astro で SSR してみましょう。

#### Astro で Server⁠-⁠side Rendering する

Deno で SSR するために Astro には [`@astrojs/deno`](https://docs.astro.build/en/guides/integrations-guide/deno/) という adapter が用意されています。

`astro.config.mjs` で Deno で SSR するための設定と adapter を指定します。

```diff js: astro.config.mjs
import { defineConfig } from "npm:astro@2.8.1/config";
+ import deno from "npm:@astrojs/deno@4.2.0";

export default defineConfig({
+   output: "server",
+   adapter: deno(),
});
```

また、`deno task` の `preview` コマンドも変更します。

```diff json: deno.json
{
  "tasks": {
-   "preview": "deno run -A --node-modules-dir npm:astro@2.8.1 preview",
+   "preview": "deno run --allow-net --allow-read --allow-env ./dist/server/entry.mjs",
  }
}
```

SSR されているかどうかを確認するために、`index.astro` にサーバーサイドで実行する処理を追加しましょう。

```html
---
// サーバーサイドで値を取得
// (リクエストを送るごとに毎回異なるユーザー情報がレスポンスとして返ってくるAPI)
const personResponse = await fetch('https://randomuser.me/api/');
const personData = await personResponse.json();
const randomPerson = personData.results[0];
---

<html lang="en">
  <body>
    <section class="profile">
      <div>
        <!-- サーバーサイドで取得したデータを使用 -->
        <p>Name: {randomPerson.name.first}</p>
        <p>age: {randomPerson.dob.age}</p>
      </div>
      <img src="{randomPerson.picture.medium}" alt="user picture" />
    </section>
  </body>
</html>
```

#### Deno Deploy にデプロイする

Astro のドキュメントに紹介されている [GitHub Actions を使った Deno Deploy へのデプロイ方法](https://docs.astro.build/en/guides/deploy/deno/#github-actions-deployment)を参考にします。

この Astro のドキュメントでは npm を使った依存関係のダウンロードやビルドが想定された GitHub Actions の実装が紹介されていますが、今回はそれら全てを Deno で行えるので、どの部分を変更します。

```yml: .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: main
  pull_request:
    branches: main

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Clone repository
        uses: actions/checkout@v3

      # Denoのセットアップ
      - uses: denoland/setup-deno@v1
        with:
          deno-version: v1.35.0

      # Denoを使ってAstroをビルド
      - name: Build Astro
        run: deno task build

      # Deno Deployにデプロイするためのアクションを実行
      - name: Upload to Deno Deploy
        uses: denoland/deployctl@v1
        with:
          project: "nus3-deno-astro"
          entrypoint: server/entry.mjs
          root: dist
```

以上で Astro の Server⁠-⁠side Rendering を使ったプロジェクトが Deno Deploy にデプロイできます。

実際に[デモ](https://nus3-deno-astro.deno.dev/)をみてもらうと、レスポンスで返ってくる HTML にはサーバーサイドで生成した値が含まれていることが確認できます。

## あとがき

v2.0 での[Hybrid Rendering](https://astro.build/blog/hybrid-rendering/)もそうですが、Astro では Server⁠-⁠side Rendering の機能が強化されている印象があるので、それらの機能を Linter や Formatter、[Permissions](https://deno.land/manual@v1.35.1/basics/permissions)が包括されている Deno と、Deno 環境を簡単にデプロイできる Deno Deploy で使えるようになるのは嬉しいですね。
