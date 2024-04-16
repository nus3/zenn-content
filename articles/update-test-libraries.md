---
title: "「実装例から見る React のテストの書き方」をアップデートする"
emoji: "🆙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react", "vitest", "storybook"]
published: false
publication_name: "cybozu_frontend"
---

[社内の人](https://twitter.com/shisama_)から、自分が以前書いた次の記事が「便利で助かった！書いた時から何かアップデートある？」ってメッセージがきた。

https://blog.cybozu.io/entry/2022/08/29/110000

そんな便利だなんてどうもありがとうございますウフフ、と思いながら書いた日を見てみると **2022-08-09** だった。もうすぐ 2 年経とうとしてる。時の流れが早い。怖い。

当時、この記事に書かれた実装例は[リポジトリ](https://github.com/nus3/react-test-examples)にまとめていたんだけど、当然、何かメンテをしていたわけもなく、2022 年当時の状態がそのまま残っていた。

せっかく便利に思ってくれる人がいたので、内容をアップデートする。

## アップデート方針

ライブラリの変更、バージョンアップを次のように行なった。

- Yarn → pnpm
- ESLint, Prettier → Biome
- Jest, @swc/jest → Vitest
- MSW: v0 → v2
- @testing-library/react: v13 → v15
- Storybook: v6 → v8

テストに関わるもの(MSW, testing-library, Storybook)に関しては単純にバージョンアップ、テストランナー(Vitest)の変更理由に関しては後述する。

それ以外(pnpm, Biome)は「面白そう」ぐらいの軽い気持ちで移行してる。

次の PR は実際に今回作業したものだ。
https://github.com/nus3/react-test-examples/pull/12

## Yarn → pnpm

最近は使用時の体験が良いので、個人のプロジェクトだともっぱら pnpm を使ってる。

今回の対象のリポジトリは依存するパッケージがそこまで多くなく、イレギュラーなこともしてないので、Yarn v3 から pnpm への移行は簡単だった。

- (モノレポの場合) pnpm-workspace.yaml の作成
- [`pnpm import`](https://pnpm.io/ja/cli/import)を使い、yarn.lock から pnpm-lock.yaml を生成
- Yarn 関連のファイルを削除
- GitHub Actions で pnpm を使うように変更
  - https://pnpm.io/ja/continuous-integration#github-actions

上記、作業の他に GitHub Actions での pnpm のセットアップが冗長だったので[composite action](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action) を使ったりもした。

### 実際に作業したプルリク

https://github.com/nus3/react-test-examples/pull/10

## ESLint, Prettier → Biome

[前回の記事](https://blog.cybozu.io/entry/2022/08/29/110000#%E8%A3%9C%E8%B6%B3-Testing-Library-%E3%81%AE%E8%A8%98%E6%B3%95%E3%82%92%E3%83%81%E3%82%A7%E3%83%83%E3%82%AF%E3%81%97%E3%81%A6%E3%81%8F%E3%82%8C%E3%82%8Beslint-plugin-testing-library)で紹介した testing-library 用の Plugin は Biome では[対応してない](https://biomejs.dev/linter/rules-sources/#eslint-plugin-barrel-files)が、そのほかのケースには大体対応してくれそうだったのと、ESlint 関連のパッケージの依存を減らせることもあって Biome に移行した。

大まかな作業内容としては次になる。

- @biomejs/biome のインストール
- `pnpm biome init`
- [使っているエディタで拡張機能のインストール](https://biomejs.dev/guides/integrate-in-editor/)、設定
- Biome のコマンドを追加
- Biome に引っかかるコードを修正
- ESLint、Prettier 関連のファイル、パッケージを削除

### `pnpm biome init`

Biome では`pnpm biome init`を実行すると、何をすればいいのかステップごとに表示される。とても親切。

:::details `pnpm biome init`で表示される内容

```shell
Files created

  - biome.json
    Your project configuration. See https://biomejs.dev/reference/configuration

Next Steps

  1. Setup an editor extension
     Get live errors as you type and format when you save.
     Learn more at https://biomejs.dev/guides/integrate-in-editor/

  2. Try a command
     biome check  checks formatting, import sorting, and lint rules.
     biome --help displays the available commands.

  3. Migrate from ESLint and Prettier
     biome migrate eslint   migrates your ESLint configuration to Biome.
     biome migrate prettier migrates your Prettier configuration to Biome.

  4. Read the documentation
     Find guides and documentation at https://biomejs.dev/guides/getting-started/

  5. Get involved with the community
     Ask questions and contribute on GitHub: https://github.com/biomejs/biome
     Seek for help on Discord: https://discord.gg/BypW39g6Yc
```

:::

### Biome のコマンドを追加

lint と format 用のコマンドは次のように追加、変更した。

```diff: package.json
-    "lint": "eslint ./ --ext ts,tsx,js,jsx",
+    "fix": "biome check --apply .",
+    "lint": "biome lint .",
```

このコマンドの指定だと`.`の中に node_modules などが入ってると、大量のファイルに lint や format がかかるのでは？となったが、[v1.5.0](https://biomejs.dev/internals/changelog/#150-2024-01-08) から CLI で実行時に`.gitignore`が考慮されるようになっている。素敵。

> The CLI now takes in consideration the .gitignore in the home directory of the user, if it exists. Contributed by @ematipico

### tab or space?

Biome ではインデントがデフォルトで tab になっているので、己の信仰に合わせて変更が必要かもしれない。

```json:biome.json
{
  "formatter": {
    "indentStyle": "tab" or "space",
  }
}

```

### 実際に作業したプルリク

https://github.com/nus3/react-test-examples/pull/14

## Jest, @swc/jest → Vitest

元の記事を書いた 2022-08-09 は Vitest のバージョンが [v0.21.1](https://github.com/vitest-dev/vitest/releases/tag/v0.21.1) であり、ドキュメントにまだ experimental だよみたいなことが書かれていた気がするので、当時は Jest を採用することが多かった。

その後、Vitest のメジャーバージョンがリリースされたこともあり、今は Vite を使っているプロダクトであればビルドツールを統一できる Vitest を採用するのが自然かなと思う。

そんな背景もあり今回は Jest から Vitest に移行した。

- Vitest のインストール、Vite を v5 にアップデート
- vite.config.ts にテスト用の設定を追加
- [マイグレーションガイド](https://vitest.dev/guide/migration.html#migrating-from-jest)を元に jest→Vitest へテストコードを修正する

### vite.config.ts にテスト用の設定を追加

```diff ts:vite.config.ts
+ /// <reference types="vitest" />
// ...
export default defineConfig({
  plugins: [react()],
+  test: {
+    globals: true,
+    environment: 'jsdom',
+    setupFiles: './test/setup.ts',
+    css: true
+  }
});
```

### Jest→Vitest へテストコードを修正

次のような関数を vitest から import するように変更する.

- `afterAll`や`afterEach`などの hook 系のメソッド
- `test`や`describe`
- `jest.fn`、`jest.mock`などは`vi.fn`、`vi.mock`に変更

https://github.com/nus3/react-test-examples/pull/13/commits/bff36b208e9d41a100c485ade53d193d339c22ac

### `@vitest/ui`と VSCode 拡張

`@vitest/ui`を追加し、`vitest --ui`を実行することで、テストの詳細情報がブラウザ上で確認できる。X にもポストしたがテスト対象の Module Graph が見れたりする。すごい。

https://x.com/nus3_/status/1780096722041225522

また Vitest の VSCode 拡張も開発が進んでおり、VSCode 上でテストの実行、デバッグ、監視ができるようになっている。
https://marketplace.visualstudio.com/items?itemName=vitest.explorer

### 実際に作業したプルリク

https://github.com/nus3/react-test-examples/pull/13

## あとがき

全体のコード量は多くはなかったが、それでも作業時間は合計すると半日ぐらいかかった気がする。日頃の(計画的な)アップデートは大事だなと改めて実感。

## jest/swc→vitest

https://github.com/nus3/react-test-examples/pull/11

- Vitest が 1.0 になったので Vitest を使おう
- Vitest 1.0 requires Vite >=v5.0.0 and Node >=v18.0.0
- jest から vitest への移行方法
  - https://vitest.dev/guide/migration.html#migrating-from-jest
- vite に掲載されてる react + ts のバージョンに買うライブラリを揃える
  - https://stackblitz.com/edit/vitejs-vite-8tkjpr?file=package.json&terminal=dev
  - tsconfig、tsconfig.node.json も揃える
- vitest で msw, testing-library, react の example があったのでそれを参考にする
  - https://stackblitz.com/edit/vitest-dev-vitest-yf7kxm?file=src%2Futils%2Ftest-utils.tsx&initialPath=__vitest__/
  - testing-library/jest-dom を使う場合
  - https://github.com/testing-library/jest-dom?tab=readme-ov-file#with-vitest
- vitest が提供するメソッドをテストファイルで使うように
- vitest、ui もいいし、VSCode 拡張も開発進んでていい感じよ

## msw -> v2 対応

https://mswjs.io/docs/migrations/1.x-to-2.x/

- rest を http に変更
  - Response を返すように
- setupServer の引数に handlers を渡すように変更

## eslint, prettier → biome

https://biomejs.dev/blog/biome-v1-7/

- biome をインストールして、対象のパッケージで pnpm biome init

```
Files created

  - biome.json
    Your project configuration. See https://biomejs.dev/reference/configuration

Next Steps

  1. Setup an editor extension
     Get live errors as you type and format when you save.
     Learn more at https://biomejs.dev/guides/integrate-in-editor/

  2. Try a command
     biome check  checks formatting, import sorting, and lint rules.
     biome --help displays the available commands.

  3. Migrate from ESLint and Prettier
     biome migrate eslint   migrates your ESLint configuration to Biome.
     biome migrate prettier migrates your Prettier configuration to Biome.

  4. Read the documentation
     Find guides and documentation at https://biomejs.dev/guides/getting-started/

  5. Get involved with the community
     Ask questions and contribute on GitHub: https://github.com/biomejs/biome
     Seek for help on Discord: https://discord.gg/BypW39g6Yc
```

- その後に`pnpm dlx @biomejs/biome migrate`を実行した
- vscode 拡張をワークスペースに適用
- koba さんが biome に移管したプルリク
  - https://github.com/koba04/swr-devtools/pull/131/files

```json
		"fix": "biome check --apply .",
		"fix:unsafe": "biome check --apply-unsafe .",
		"lint": "biome lint .",
```

- biome が無視するファイル
  - https://biomejs.dev/guides/how-biome-works/#protected-files
- node_moduels もデフォルトで無視されてそうではある
  - .gitignore を考慮してくれる
  - The CLI now takes in consideration the .gitignore in the home directory of the user, if it exists.
  - https://biomejs.dev/internals/changelog/#new-features-13
- indent を tab から space に変更する

## testing-library -> v15 対応

- 対象のライブラリをアプデした
- 1 つテストが落ちた
- 大量のテストで以下の warning が出た

```
Warning: An update to SelectBox inside a test was not wrapped in act(...).

When testing, code that causes React state updates should be wrapped into act(...):

act(() => {
  /* fire events that update state */
});
/* assert on the output */

This ensures that you're testing the behavior the user would see in the browser. Learn more at https://reactjs.org/link/wrap-tests-with-act
    at SelectBox
```

- 多分、v14 のリリースが影響してそう
  - https://github.com/testing-library/react-testing-library/releases/tag/v14.0.0
- 過去に v14 対応した時のプルリク
  - https://github.dev.cybozu.co.jp/kintone/kintone/pull/32308
- `await userEvent`してるのに warning が出るのは何故か
  - https://qiita.com/naoki96/items/fd9ac220d158a4b33b14
- @testing-library/dom のバージョンが揃ってないかららしい

```shell
❯ pnpm why @testing-library/dom
Legend: production dependency, optional only, dev only

react-vitest@0.0.0 /Users/nus3/dev/playground/react-test-examples/apps/react-vitest

devDependencies:
@storybook/testing-library 0.0.11
├── @testing-library/dom 8.13.0
└─┬ @testing-library/user-event 13.5.0
  └── @testing-library/dom 8.13.0 peer
@testing-library/react 15.0.2
└── @testing-library/dom 10.0.0
@testing-library/user-event 14.5.2
└── @testing-library/dom 8.13.0 peer
```

- @storybook/testing-library が依存してるから storybook の v8 対応も進める
- storybook のバージョンアップ対応したら、warning は解消された
  - @testing-library/react と@testing-library/user-event で@testing-library/dom のバージョンが揃ったから

```shell
❯ pnpm why @testing-library/dom
Legend: production dependency, optional only, dev only

react-vitest@0.0.0 /Users/nus3/dev/playground/react-test-examples/apps/react-vitest

devDependencies:
@storybook/addon-interactions 8.0.8
└─┬ @storybook/test 8.0.8
  ├── @testing-library/dom 9.3.4
  └─┬ @testing-library/user-event 14.5.2
    └── @testing-library/dom 9.3.4 peer
@storybook/test 8.0.8
├── @testing-library/dom 9.3.4
└─┬ @testing-library/user-event 14.5.2
  └── @testing-library/dom 9.3.4 peer
@testing-library/react 15.0.2
└── @testing-library/dom 10.0.0
@testing-library/user-event 14.5.2
└── @testing-library/dom 10.0.0 peer
```

- 全部直ると、useFakeTimers 使ってるところで落ちる
  - `vi.useFakeTimers({ shouldAdvanceTime: true });`がいいらしい
  - https://github.com/testing-library/react-testing-library/issues/1198#issuecomment-1489224361
- https://vitest.dev/api/vi.html#vi-usefaketimers
  - The implementation is based internally on @sinonjs/fake-timers.
  - 実装は@sinonjs/fake-timers に基づいている
    - https://github.com/sinonjs/fake-timers
  - shouldAdvanceTime
    - https://github.com/sinonjs/fake-timers?tab=readme-ov-file#api-reference
    - 実システムのモック時間を自動的にインクリメントするように指示する
- composeStories と play 関数は混ぜない方がいいかも
  - 同じ warning が出る

```shell
Warning: An update to Toast inside a test was not wrapped in act(...).

When testing, code that causes React state updates should be wrapped into act(...):

act(() => {
  /* fire events that update state */
});
/* assert on the output */
```

- vitest 側と、storybook 側の@testing-library/dom のバージョンが違うことが原因に感じる

```shell
❯ pnpm why @testing-library/dom

devDependencies:
@storybook/addon-interactions 8.0.8
└─┬ @storybook/test 8.0.8
  ├── @testing-library/dom 9.3.4
  └─┬ @testing-library/user-event 14.5.2
    └── @testing-library/dom 9.3.4 peer
@storybook/test 8.0.8
├── @testing-library/dom 9.3.4
└─┬ @testing-library/user-event 14.5.2
  └── @testing-library/dom 9.3.4 peer
@testing-library/react 15.0.2
└── @testing-library/dom 10.0.0
@testing-library/user-event 14.5.2
└── @testing-library/dom 10.0.0 peer
```

- ということで play 関数のテストは storybook のテストランナーを使うことにしましょう

## storybook

https://storybook.js.org/docs/migration-guide

- `pnpm dlx storybook@latest upgrade`やってみた

```shell
❯ pnpm dlx storybook@latest upgrade
Packages: +599
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Progress: resolved 621, reused 457, downloaded 142, added 599, done
╭───────────────────────────────────────────────────────────────╮
│                                                               │
│   Upgrading Storybook from version 6.5.6 to version 8.0.8..   │
│                                                               │
╰───────────────────────────────────────────────────────────────╯
info Checking for upgrade blockers...
╭──────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│                                                                                                          │
│   Storybook has found potential blockers in your project that need to be resolved before upgrading:      │
│                                                                                                          │
│   StoryStoreV7 feature must be removed from your Storybook configuration.                                │
│   This feature was removed in Storybook 8.0.0.                                                           │
│   Please see the migration guide for more information:                                                   │
│   https://github.com/storybookjs/storybook/blob/next/MIGRATION.md#storystorev6-and-storiesof-is-deprec   │
│   ated                                                                                                   │
│                                                                                                          │
│   In your Storybook configuration we found storyStoryV7 feature defined. For instance:                   │
│                                                                                                          │
│   export default = {                                                                                     │
│   features: {                                                                                            │
│   storyStoreV7: false, <--- remove this line                                                             │
│   },                                                                                                     │
│   };                                                                                                     │
│                                                                                                          │
│   You need to remove the storyStoreV7 property.                                                          │
│                                                                                                          │
│   ─────────────────────────────────────────────────                                                      │
│                                                                                                          │
│   Fix the above issues and try running the upgrade command again.                                        │
│                                                                                                          │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯

attention => Storybook now collects completely anonymous telemetry regarding usage.
This information is used to shape Storybook's roadmap and prioritize features.
You can learn more, including how to opt-out if you'd not like to participate in this anonymous program, by visiting the following URL:
https://storybook.js.org/telemetry
```

- `storyStoreV7`を消せと
- 以下のコードを削除

```ts
	features: {
		storyStoreV7: true,
	},
```

- もう一回`pnpm dlx storybook@latest upgrade`を実行

```shell
devDependencies:
- @storybook/addon-docs 6.5.6
+ @storybook/addon-docs 8.0.8
- @storybook/addon-essentials 6.5.6
+ @storybook/addon-essentials 8.0.8
- @storybook/addon-interactions 6.5.6
+ @storybook/addon-interactions 8.0.8
- @storybook/addon-links 6.5.6
+ @storybook/addon-links 8.0.8
- @storybook/builder-vite 0.1.35
+ @storybook/builder-vite 8.0.8
- @storybook/react 6.5.6
+ @storybook/react 8.0.8
```

```shell
🔎 checking possible migrations..

🔎 found a 'new-frameworks' migration:
╭ Automigration detected ──────────────────────────────────────────────────────────────────────────────────╮
│                                                                                                          │
│   We've detected your project is not fully setup with the new framework format, which was introduced     │
│   in Storybook 7.                                                                                        │
│                                                                                                          │
│   Storybook 7 introduced the concept of frameworks, which abstracts configuration for renderers (e.g.    │
│   React, Vue), builders (e.g. Webpack, Vite) and defaults to make integrations easier.                   │
│                                                                                                          │
│   Your project should be updated to use Storybook's framework: @storybook/react-vite. We can attempt     │
│   to do this for you automatically.                                                                      │
│                                                                                                          │
│   Here are the steps this migration will do to migrate your project:                                     │
│   - Remove the following dependencies:                                                                   │
│   - * @storybook/builder-vite                                                                            │
│   - Add the following dependencies:                                                                      │
│   - * @storybook/react-vite                                                                              │
│   - Remove the core.builder field in .storybook/main.ts.                                                 │
│                                                                                                          │
│                                                                                                          │
│   To learn more about the new framework format, see:                                                     │
│   https://github.com/storybookjs/storybook/blob/next/MIGRATION.md#new-framework-api                      │
│                                                                                                          │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯
✔ Do you want to run the 'new-frameworks' migration on your project? … yes
✅ Removing dependencies: @storybook/builder-vite
✅ Installing new dependencies: @storybook/react-vite
✅ Updating main.js
✅ Updating "framework" field
✅ Removing "core.builder" field
✅ Removing "core" field
✅ ran new-frameworks migration
```

```shell
🔎 found a 'storybook-binary' migration:
╭ Automigration detected ──────────────────────────────────────────────────────────────────────────────────╮
│                                                                                                          │
│   We've detected you are using Storybook 8.0.8 without Storybook's storybook binary. Starting in         │
│   Storybook 7.0, it has to be installed.                                                                 │
│                                                                                                          │
│                                                                                                          │
│   More info: https://github.com/storybookjs/storybook/blob/next/MIGRATION.md#start-storybook--build-st   │
│   orybook-binaries-removed                                                                               │
│                                                                                                          │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯
✔ Do you want to run the 'storybook-binary' migration on your project? … yes

✅ Adding 'storybook' as dev dependency

✅ ran storybook-binary migration
```

```shell
 found a 'sb-scripts' migration:
╭ Automigration detected ──────────────────────────────────────────────────────────────────────────────────╮
│                                                                                                          │
│   We've detected you are using Storybook 8.0.8 with scripts from previous versions of Storybook.         │
│   Starting in Storybook 7, the start-storybook and build-storybook binaries have changed to storybook    │
│   dev and storybook build respectively.                                                                  │
│   In order to work with Storybook 8.0.8, your storybook scripts have to be adjusted to use the binary.   │
│   We can adjust them for you:                                                                            │
│                                                                                                          │
│   storybook                                                                                              │
│   from:                                                                                                  │
│   start-storybook -p 1234                                                                                │
│   to:                                                                                                    │
│   storybook dev -p 1234                                                                                  │
│                                                                                                          │
│   build-storybook                                                                                        │
│   from:                                                                                                  │
│   build-storybook                                                                                        │
│   to:                                                                                                    │
│   storybook build                                                                                        │
│                                                                                                          │
│   In case this migration did not cover all of your scripts, or you'd like more info: https://github.co   │
│   m/storybookjs/storybook/blob/next/MIGRATION.md#start-storybook--build-storybook-binaries-removed       │
│                                                                                                          │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯
✔ Do you want to run the 'sb-scripts' migration on your project? … yes
✅ Updating scripts in package.json


✅ ran sb-scripts migration
```

```shell
🔎 found a 'remove-jest-testing-library' migration:
╭ Automigration detected ──────────────────────────────────────────────────────────────────────────────────╮
│                                                                                                          │
│   Attention: We've detected that you're using the following packages which are known to be               │
│   incompatible since Storybook 8:                                                                        │
│                                                                                                          │
│   - @storybook/testing-library                                                                           │
│                                                                                                          │
│   We will uninstall them for you and install @storybook/test instead.                                    │
│                                                                                                          │
│   Also, we can help you migrate your stories to use the new package.                                     │
│                                                                                                          │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯
✔ Do you want to run the 'remove-jest-testing-library' migration on your project? … yes
✔ Please enter the glob for your stories to migrate to @storybook/test … ./src/**/*.stories.*
```

```shell
🔎 found a 'autodocsTrue' migration:
╭ Automigration detected ────────────────────────────────────────────────────────────────────────────╮
│                                                                                                    │
│   We've changed the configuration of autodocs (previous docsPage), so now the value:               │
│     - docs.autodocs: true -- means automatically create docs for every CSF file                    │
│     - docs.autodocs: 'tag' -- means only create autodocs for CSF files with the 'autodocs' tag     │
│     - docs.autodocs: false -- means never create autodocs                                          │
│                                                                                                    │
│   Based on your prior configuration,  we can set the `docs.autodocs` to keep your old behaviour:   │
│                                                                                                    │
│   docs: { autodocs: true }                                                                         │
│                                                                                                    │
│   More info: https://github.com/storybookjs/storybook/blob/next/MIGRATION.md#autodocs-changes      │
│                                                                                                    │
╰────────────────────────────────────────────────────────────────────────────────────────────────────╯
✔ Do you want to run the 'autodocsTrue' migration on your project? … yes
✅ Setting 'docs.autodocs' to true in main.js
✅ ran autodocsTrue migratio
```

```shell
🔎 found a 'upgradeStorybookRelatedDependencies' migration:
╭ Automigration detected ──────────────────────────────────────────────────────────────────────────────────╮
│                                                                                                          │
│   You're upgrading to the latest version of Storybook. We recommend upgrading the following packages:    │
│   - @storybook/testing-react: 1.3.0 => 2.0.1                                                             │
│                                                                                                          │
│   After upgrading, we will run the dedupe command, which could possibly have effects on dependencies     │
│   that are not Storybook related.                                                                        │
│   see: https://docs.npmjs.com/cli/commands/npm-dedupe                                                    │
│                                                                                                          │
│   Do you want to proceed (upgrade the detected packages)?                                                │
│                                                                                                          │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯
✔ Do you want to run the 'upgradeStorybookRelatedDependencies' migration on your project? … yes
Installing dependencies...

Scope: all 2 workspace projects
 WARN  deprecated @storybook/testing-react@2.0.1: In Storybook 7, this package has been promoted to a first-class Storybook functionality. This means that you no longer need it! Instead, you can import the same utilities, but from the @storybook/react package. Please migrate when you can.
../..                                    |   +6  -29 +---
../..                                    | Progress: resolved 958, reused 896, downloaded 6, added 6, done

devDependencies:
- @storybook/testing-react 1.3.0
+ @storybook/testing-react 2.0.1 deprecated

 WARN  Issues with peer dependencies found
apps/react-vitest
└─┬ @storybook/testing-react 2.0.1
  └── ✕ unmet peer @storybook/react@"^7.0.0-beta.0 || ^7.0.0-rc.0 || ^7.0.0": found 8.0.8

Done in 1.9s

We upgraded 1 packages:
- @storybook/testing-react: 1.3.0 => 2.0.1

✅ ran upgradeStorybookRelatedDependencies migration
```

```shell
╭ Migration check ran successfully ────────────────────────────────────────────────────────────────────────╮
│                                                                                                          │
│   Successful migrations:                                                                                 │
│                                                                                                          │
│   new-frameworks, storybook-binary, sb-scripts, remove-jest-testing-library, autodocsTrue,               │
│   wrap-require, upgradeStorybookRelatedDependencies                                                      │
│                                                                                                          │
│   Skipped migrations:                                                                                    │
│                                                                                                          │
│   visual-tests-addon                                                                                     │
│                                                                                                          │
│   ─────────────────────────────────────────────────                                                      │
│                                                                                                          │
│   If you'd like to run the migrations again, you can do so by running 'npx storybook automigrate'        │
│                                                                                                          │
│   The automigrations try to migrate common patterns in your project, but might not contain everything    │
│   needed to migrate to the latest version of Storybook.                                                  │
│                                                                                                          │
│   Please check the changelog and migration guide for manual migrations and more information:             │
│   https://storybook.js.org/docs/8.0/migration-guide                                                      │
│   And reach out on Discord if you need help: https://discord.gg/storybook                                │
│                                                                                                          │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯
```

- バージョンが固定化されてなかったのと、dependencies に`@storybook/test`が入ってたのでそれを移動
- Storybook は esm 対応してるっぽいので main.ts を esm で書けるようにする
  - https://github.com/storybookjs/storybook/issues/9854#issuecomment-1534669700
- main.ts で getAbsolutePath を使う理由
  - https://storybook.js.org/docs/faq#how-do-i-fix-module-resolution-in-special-environments
  - monorepo 構成の場合に必要そう
- @storybook/testing-react は@storybook/react から利用できるようになったのでいらない
  - https://github.com/storybookjs/testing-react?tab=readme-ov-file#%EF%B8%8F-attention
- composeStories からのテストコードでの play 関数の実行はこれで対応する
  - https://github.com/storybookjs/storybook/issues/21969#issuecomment-1638924687
- @storybook/addon-actions の対応
  - `@storybook/test`の fn でモックすれば良き
    - https://storybook.js.org/docs/essentials/actions#via-storybooktest-fn-spy-function
  - めんどくさい場合は一括で指定できる
    - https://storybook.js.org/docs/essentials/actions#automatically-matching-args
    - が play 関数との兼ね合いで推奨はしてないって
- Storybook の型指定を satisfies のやつに変えたよ
- @storybook/test の userEvent が promise 返すようになってた

### Storybook のテストランナー

- @storybook/test-runner を追加
- デフォルトポートが 6006 なので storybook 側もそれに合わせる
- `pnpm dlx playwright install`を使ってブラウザをインストール
