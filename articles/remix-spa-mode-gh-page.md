---
title: "SPA モードのRemixを GitHub Pages にデプロイする"
emoji: "💿"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["remix", "githubactions", "githubpages"]
published: true
publication_name: "cybozu_frontend"
---

SPA モードの Remix を GitHub Pages にデプロイする方法が Remix のアカウントでポストされていたので、ふとその[リポジトリ](https://github.com/brookslybrand/remix-gh-pages)を見てみると、GitHub Actions を使った GitHub Pages へのデプロイが簡単に行えるようになっていて驚いた。

https://x.com/remix_run/status/1768640090010468568?s=20

このポストを見ると GitHub Actions から提供されている Action など知らなかったものがいくつかあったので、Remix SPA モードのデプロイ方法など含め、内容を紹介したい。

## 動作確認した環境

関連がありそうな依存を記す。

```json
  "dependencies": {
    "@remix-run/node": "^2.8.1",
    "@remix-run/react": "^2.8.1",
    "@remix-run/serve": "^2.8.1",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
  },
  "devDependencies": {
    "@remix-run/dev": "^2.8.1",
    "vite": "^5.1.0",
    "vite-tsconfig-paths": "^4.2.1"
  },
```

## Vite からの Remix のセットアップ

Vite の[プロジェクトを作成するコマンド](https://vitejs.dev/guide/#scaffolding-your-first-vite-project)久しぶりに実行してみると Remix が選択肢に増えてる。
[Remix が Vite をサポート](https://remix.run/blog/remix-vite-stable)したのちょっと前だった気がするけども、Vite のセットアップの選択肢の中にも増えてる。仕事が早い。

```shell
pnpm create vite
✔ Project name: … hoge
✔ Project name: … hoge
✔ Select a framework: › React
? Select a variant: › - Use arrow-keys. Return to submit.
    TypeScript
    TypeScript + SWC
    JavaScript
    JavaScript + SWC
❯   Remix ↗
# Remix増えとるやんか
```

### SPA モードの設定

Vite で Reimix を選択しプロジェクトを作成したら、生成される`vite.config.ts`を変更することで SPA モードが利用できる。`ssr`を`false`にするだけ。簡単。

```diff typescript: vite.config.ts
export default defineConfig({
-   plugins: [remix(), tsconfigPaths()],
+   plugins: [remix({
+     ssr: false
+   }), tsconfigPaths()]
});
```

### GitHub Pages の設定を追加

GitHub Pages は公開先の URL が`https://{github-account-name}.github.io/{repository-name}`になるので、その対応を `vite.config.ts` に追加する。

```diff typescript: vite.config.ts
+ import { copyFileSync } from "node:fs";
+ import { join } from "node:path";

export default defineConfig({
+ base: "/repository-name/",
  plugins: [remix({
    ssr: false,
+   basename: "/repository-name/",
+   buildEnd(args) {
+     if (!args.viteConfig.isProduction) return;
+     const buildPath = args.viteConfig.build.outDir;
+     copyFileSync(
+       join(buildPath, "index.html"),
+       join(buildPath, "404.html"),
+     );
+   }
  }), tsconfigPaths()]
});
```

[Remix が Vite をサポートしたことで Basename にも対応してる](https://remix.run/blog/remix-vite-stable#basename-support)。

また、ポストで紹介されていたリポジトリでも[コメント](https://github.com/brookslybrand/remix-gh-pages/blob/ce5981bbdf29fa0c4adbdb1634f8ae05c6af9609/vite.config.ts#L16-L23)されていたが、GitHub Pages の場合、全てが index.html に行かない場合があるので同様の内容を 404.html としてコピーする処理を`buildEnd`で追加している。

## SPA モードの Remix を GitHub Pages にデプロイする

GitHub のリポジトリの Settings から Pages を選択し、Build and deployment の Source を GitHub Actions に設定すれば、あとは、ポストで紹介されている次の yaml ファイルを追加することで SPA モードの Remix を GitHub Pages にデプロイできる。

https://github.com/brookslybrand/remix-gh-pages/blob/main/.github/workflows/build-deploy.yml

んだが、久しぶりに触った GitHub Actions で自分の中でいくつかアップデートがあったので、その内容をまとめる。

### `permissions`

https://docs.github.com/en/rest/authentication/permissions-required-for-github-apps?apiVersion=2022-11-28#repository-permissions-for-pages

```yaml
permissions:
  contents: read
  pages: write
  id-token: write
```

`pages`はその名前通り、GitHub Pages を扱う上で必要な権限。許可される[詳細の Endpoint もドキュメントに記載](https://docs.github.com/en/rest/authentication/permissions-required-for-github-apps?apiVersion=2022-11-28#repository-permissions-for-pages)されてる。`id-token`はドキュメント読む感じ、`write`にすることで GitHub の OIDC プロパイダから JWT 受け取れるようになる権限なのかな。

`pages`と`id-token`は後に紹介する`actions/deploy-pages`で必要な権限になる。
https://github.com/actions/deploy-pages?tab=readme-ov-file#security-considerations。

### `pnpm/action-setup`

```yaml
- uses: pnpm/action-setup@v3
  name: Install pnpm
  with:
    version: 8
    run_install: false
```

今回は`pnpm`を使っているが、pnpm ではセットアップ用の action が提供されているので、ドキュメント通りに使うことで GitHub Actions 上でもすぐに pnpm を利用できる。

cache を使ってインストール時間を短縮する方法も記載されてる親切設計。

https://github.com/pnpm/action-setup?tab=readme-ov-file#use-cache-to-reduce-installation-time

```yaml
- name: Get pnpm store directory
  shell: bash
  run: |
    echo "STORE_PATH=$(pnpm store path --silent)" >> $GITHUB_ENV

- uses: actions/cache@v4
  name: Setup pnpm cache
  with:
    path: ${{ env.STORE_PATH }}
    key: ${{ runner.os }}-pnpm-store-${{ hashFiles('**/pnpm-lock.yaml') }}
    restore-keys: |
      ${{ runner.os }}-pnpm-store-
```

### `actions/configure-pages`

https://github.com/actions/configure-pages

README には GitHub Pages を有効にすると書かれている。この有効にするの詳細は実装を見ると、GitHub Pages のサイトを作成するための API が呼ばれていた。

https://github.com/actions/configure-pages/blob/983d7736d9b0ae728b81ab479565c72886d7745b/src/api-client.js#L9-L13

https://docs.github.com/ja/rest/pages/pages?apiVersion=2022-11-28#create-a-github-pages-site

なので、この Action を使っていれば前述した次の手順はもしかしたら必要ないかも知れない。

> GitHub のリポジトリの Settings から Pages を選択し、Build and deployment の Source を GitHub Actions に設定すれば

### `actions/upload-pages-artifact`

```yaml
- name: Upload artifact
  uses: actions/upload-pages-artifact@v3
  with:
    path: "build/client"
```

https://github.com/actions/upload-pages-artifact

GitHub Pages にデプロイするために必要なファイルを artifact にアップロードしてくれる Action。`path`の`"build/client"`は Remix の SPA モードでビルドされたファイルが出力されるディレクトリ。

この設定をすることで、`path`で指定した対象が`github-pages`という名前で artifact にアップロードされる。

### `actions/deploy-pages`

```yaml
- name: Deploy to GitHub Pages
  id: deployment
  uses: actions/deploy-pages@v4
```

https://github.com/actions/deploy-pages

`actions/upload-pages-artifact`で artifact にアップロードされたファイルを使って GitHub Pages にデプロイする Action。なので、[デフォルト値は`actions/upload-pages-artifact`のものが使われている](https://github.com/actions/deploy-pages?tab=readme-ov-file#inputs-)（artifact の名前は`github-pages`がデフォルト値）。

`id: deployment`の指定は Context から参照したいためなのかな。
https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsid

action の実装みたけど id を使ってるとこはないように見えたけど、step の id を指定することで何かしらが識別できるのかな。
https://github.com/actions/deploy-pages/blob/d6db90164ac5ed86f2b6aed7e0febac5b3c0c03e/src/internal/api-client.js#L111-L136

## あとがき

自分は以前まで、[github-pages-deploy-action](https://github.com/JamesIves/github-pages-deploy-action)を使い、GitHub Pages 用のブランチを作って、そのブランチをデプロイするみたいな方法をとっていたが、今は GitHub Actions の公式が提供する Action を使うことで、もっと簡単に、かつ、素早いデプロイができるようになっていた。

もちろん初期段階でコード量が多くないのもあるが、今回紹介した方法では GitHub Pages へのデプロイの実行時間は 24 秒しかかからなかった。速い。すごい。

![現状はGitHub ActionsでのBuildとDeployが24秒しかかかってない](/images/remix-spa-mode-gh-page/deploy-time.png)
