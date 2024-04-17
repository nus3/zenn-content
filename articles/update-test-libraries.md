---
title: "「実装例から見る React のテストの書き方」をアップデートする"
emoji: "🆙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react", "vitest", "storybook", "msw", "testinglibrary"]
published: true
publication_name: "cybozu_frontend"
---

[社内の人](https://twitter.com/shisama_)から、自分が以前書いた次の記事が「便利で助かった！書いた時から何かアップデートある？」ってメッセージがきた。

https://blog.cybozu.io/entry/2022/08/29/110000

そんな便利だなんてどうもありがとうございますウフフ、と思いながら書いた日を見てみると **2022-08-09** だった。もうすぐ 2 年経とうとしてる。時の流れが早くて怖い。

この記事に書かれた実装例は[リポジトリ](https://github.com/nus3/react-test-examples)にまとめていたんだけど、当然、何かメンテをしていたわけもなく、2022 年当時の状態がそのまま残っていた。

せっかく便利に思ってくれる人がいたので、内容をアップデートする。

## アップデートまとめ

- メジャーバージョンのリリースやビルドツールの統一の観点で Jest から Vitest に移行
- `useFakeTimers({ shouldAdvanceTime: true })`
- `@testing-library/react`を v15 にバージョンアップ
- MSW を v2 にバージョンアップ
- Storybook を v8 にバージョンアップ
- `composeStories`による Storybook の story を使ったテストでは play 関数を呼ばない
- play 関数を使ったテストは`@storybook/test-runner`で行う

## アップデート方針

ライブラリの変更、バージョンアップを次のように行なった。

- Yarn → pnpm
- ESLint, Prettier → Biome
- Jest, `@swc/jest` → Vitest
- `@testing-library/react`: v13 → v15
- Storybook: v6 → v8
- MSW: v0 → v2

テストに関わるもの(MSW, testing-library, Storybook)に関しては単純にバージョンアップ、テストランナー(Vitest)の変更理由に関しては後述する。

それ以外(pnpm, Biome)は「面白そう」ぐらいの軽い気持ちで移行してる。

今回作業した内容は次の PR にまとめている。
https://github.com/nus3/react-test-examples/pull/12

## Yarn → pnpm

最近は使用時の体験が良いので、個人のプロジェクトだと pnpm を使うことが多い。

[対象のリポジトリ](https://github.com/nus3/react-test-examples)は依存するパッケージがそこまで多くなく、イレギュラーなこともしてないので、Yarn v3 から pnpm への移行は簡単だった。

- (モノレポの場合) pnpm-workspace.yaml の作成
- [`pnpm import`](https://pnpm.io/ja/cli/import)を使い、yarn.lock から pnpm-lock.yaml を生成
- Yarn 関連のファイルを削除
- GitHub Actions で pnpm を使うように変更
  - https://pnpm.io/ja/continuous-integration#github-actions

上記、作業の他に GitHub Actions での pnpm のセットアップが冗長だったので[composite action](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action) を使ったりもした。

### 該当プルリク

https://github.com/nus3/react-test-examples/pull/10

## ESLint, Prettier → Biome

[元の記事](https://blog.cybozu.io/entry/2022/08/29/110000#%E8%A3%9C%E8%B6%B3-Testing-Library-%E3%81%AE%E8%A8%98%E6%B3%95%E3%82%92%E3%83%81%E3%82%A7%E3%83%83%E3%82%AF%E3%81%97%E3%81%A6%E3%81%8F%E3%82%8C%E3%82%8Beslint-plugin-testing-library)で紹介した testing-library 用の Plugin は Biome では[対応してない](https://biomejs.dev/linter/rules-sources/#eslint-plugin-barrel-files)が、そのほかのケースには大体対応してくれそうだったのと、ESlint 関連のパッケージの依存を減らせることもあって Biome に移行した。

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
    "enabled": true,
    "indentStyle": "tab" or "space",
  }
}

```

### モノレポの場合

Biome の VSCode 拡張では、[ワークスペースのルートディレクトリにある`biome.json`を自動的に読み込む](https://biomejs.dev/ja/reference/vscode/#%E8%A8%AD%E5%AE%9A%E3%81%AE%E8%AA%AD%E3%81%BF%E8%BE%BC%E3%81%BF)。

なので対象のプロジェクトがモノレポの場合は、ワークスペースのルートディレクトリに`biome.json`がないと、指定したルールが適用されないので注意が必要。ルート直下にベースの`biome.json`を作り、モノレポで管理している各パッケージはルート直下の config を[`extends`](https://biomejs.dev/guides/how-biome-works/#the-extends-option)を使って読み込むのも良いかも。

### Biome v1.7

https://biomejs.dev/blog/biome-v1-7/

Biome v1.7 で ESLint や Prettier の移行をサポートするコマンドがリリースされたので、既存プロジェクトの Biome への移行はもっと簡単になりそう。

### 該当プルリク

https://github.com/nus3/react-test-examples/pull/14

## Jest, `@swc/jest` → Vitest

[元の記事](https://blog.cybozu.io/entry/2022/08/29/110000)を書いた 2022-08-09 は Vitest のバージョンが [v0.21.1](https://github.com/vitest-dev/vitest/releases/tag/v0.21.1) であり、ドキュメントにまだ experimental だよみたいなことが書かれていた気がするので、当時は Jest を採用することが多かった。

その後、Vitest のメジャーバージョンがリリースされたこともあり、今は Vite を使っているプロダクトであればビルドツールを統一できる Vitest を採用するのが自然な流れかもしれない。

そんな背景もあり今回は Jest から Vitest に移行した。

- Vitest のインストール、Vite を v5 にアップデート
- vite.config.ts にテスト用の設定を追加
- `whatwg-fetch`の削除
- [マイグレーションガイド](https://vitest.dev/guide/migration.html#migrating-from-jest)を元に jest→Vitest へテストコードを修正する
- Jest や`@swc/jest`など関連するライブラリの削除

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

- [`globals`](https://vitest.dev/config/#globals): Jest のように globals API として Vitest の API を使いたいかどうか
  - [マイグレーションガイド](https://vitest.dev/guide/migration.html#globals-as-a-default)に記載があるが、この設定を false にすると testing-library は DOM のクリーンアップをしてくれないので true にしている
- [`css`](https://vitest.dev/config/#css): CSS ファイルを処理するかどうか

`@testing-library/jest-dom`が提供する Custom matchers は使いたいので、次のように、setup のファイルでは`@testing-library/jest-dom`を import する。

```ts: test/setup.ts
import "@testing-library/jest-dom";
```

### `whatwg-fetch`の削除

[元の記事](https://blog.cybozu.io/entry/2022/08/29/110000)では Node.js v16 でテスト実行していたので [`whatwg-fetch`](https://github.com/whatwg/fetch)を使って`fetch`を polyfill していたが、Node.js の v18 からフラグなしで`fetch`が使えるようになったので削除。

> v18.0.0: No longer behind --experimental-fetch CLI flag.
> [https://nodejs.org/docs/v20.12.1/api/globals.html#fetch](https://nodejs.org/docs/v20.12.1/api/globals.html#fetch)

また、Node.js の v21 から `fetch` は stable になる。
[https://nodejs.org/en/blog/release/v21.0.0](https://nodejs.org/en/blog/release/v21.0.0)

### Jest→Vitest へテストコードを修正

次のような関数を Vitest から import するように変更。

- `afterAll`や`afterEach`などの hook 系のメソッド
- `test`や`describe`
- `jest.fn`、`jest.mock`、`jest.useFakeTimers`などは`jest`部分を`vi`(`vi.fn`)に変更する

config で`globals`を true にしているので、上記の関数やオブジェクトは import しなくてもテストは問題なく実行されるが、Vitest でのデフォルトの挙動を尊重し、明示的に import している。(本音は testing-library での DOM のクリーンアップしてくれるなら`globals`は false にしたい)

### `@vitest/ui`と VSCode 拡張

`@vitest/ui`を追加し、`vitest --ui`を実行することで、テストの詳細情報がブラウザ上で確認できる。X にもポストしたがテスト対象の Module Graph が見れたりもする。すごい。

https://x.com/nus3_/status/1780096722041225522

また、Vitest の VSCode 拡張も開発が進んでおり、VSCode 上でテストの実行、デバッグ、監視ができるようになっている。
https://marketplace.visualstudio.com/items?itemName=vitest.explorer

### 該当プルリク

https://github.com/nus3/react-test-examples/pull/13

## `@testing-library/react`: v13 → v15

[元の記事](https://blog.cybozu.io/entry/2022/08/29/110000)から`@testing-library/react`のバージョンが 13.3.0 から 15.0.2 に上がっていたので、次のような対応をした

- `@testing-library/react`、`@testing-library/user-event`のバージョンアップ
- `Warning: An update to SelectBox inside a test was not wrapped in act(...).`の対応
- `vi.useFakeTimers`を使ってるテストが落ちる問題の対応

### `Warning: An update to SelectBox inside a test was not wrapped in act(...).`の対応

`@testing-library/react`と`@testing-library/user-event`をそれぞれ最新バージョンにアップデートしたところ、テスト実行時に親の顔より見た次の Warning が大量に出力された。

```shell
Warning: An update to SelectBox inside a test was not wrapped in act(...).

When testing, code that causes React state updates should be wrapped into act(...):

act(() => {
  /* fire events that update state */
});
/* assert on the output */

This ensures that you're testing the behavior the user would see in the browser. Learn more at https://reactjs.org/link/wrap-tests-with-act
```

`@testing-library/react`と`@testing-library/user-event`がそれぞれ依存する`@testing-library/dom`のバージョンが揃ってないことでこの Warning が発生するという情報が[`@testing-library/user-event`の Discussions](https://github.com/testing-library/user-event/discussions/906#discussioncomment-2522468)にあったので、調べてみる。

```shell
❯ pnpm why @testing-library/dom
Legend: production dependency, optional only, dev only

devDependencies:
@storybook/testing-library 0.0.11
├── @testing-library/dom 8.13.0
└─┬ @testing-library/user-event 13.5.0
  └── @testing-library/dom 8.13.0 peer
@testing-library/react 15.0.2
└── @testing-library/dom 10.0.0 # バージョンが揃ってへんやんけ
@testing-library/user-event 14.5.2
└── @testing-library/dom 8.13.0 peer # バージョンが揃ってへんやんけ
```

`@storybook/testing-library`の影響を受けて、`@testing-library/dom`のバージョンが揃っていなかった。後述する Storybook の v8 へのバージョンアップの過程で、`@storybook/testing-library`は`@storybook/test`に移行できる。この移行により`@testing-library/dom`のバージョンを揃えることができる。

次の出力結果は Storybook の v8 へのバージョンアップ後のもので、`@testing-library/react`と`@testing-library/user-event`それぞれで`@testing-library/dom`のバージョンが揃っている。

```shell
❯ pnpm why @testing-library/dom
Legend: production dependency, optional only, dev only

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
└── @testing-library/dom 10.0.0 # キレイにバージョンが揃ってるわね
@testing-library/user-event 14.5.2
└── @testing-library/dom 10.0.0 peer # キレイにバージョンが揃ってるわね
```

`@testing-library/dom`のバージョンが揃うことで、テスト実行時に Warning は発生しなくなる。

### `vi.useFakeTimers`を使ってるテストが落ちる問題の対応

`vi.useFakeTimers`を使っているテストで、`vi.advanceTimersByTime`を使ってタイマーを進めようとしても、テスト実行時にはタイマーが進んでおらず、テストが落ちるというケースがあった。

[それっぽい Issue](https://github.com/testing-library/react-testing-library/issues/1198#issuecomment-1489224361)を読むと、`vi.useFakeTimers({ shouldAdvanceTime: true })`という引数を渡せるようになっている。[Vitest のドキュメント](https://vitest.dev/api/vi.html#vi-usefaketimers)を読むと、`vi.useFakeTimers`の内部では[`@sinonjs/fake-timers`](https://github.com/sinonjs/fake-timers)が使われているとのこと。

この`@sinonjs/fake-timers`の[APIReference](https://github.com/sinonjs/fake-timers?tab=readme-ov-file#api-reference)を見ると、`shouldAdvanceTime`パラメーターの説明が次のように記載されていた。

> tells FakeTimers to increment mocked time automatically based on the real system time shift (e.g. the mocked time will be incremented by 20ms for every 20ms change in the real system time)

説明では`based on the real system time shift`で、今回のケースには当てはまらないような気もしたが、実際に`vi.useFakeTimers({ shouldAdvanceTime: true })`を指定してテストを実行すると、意図した挙動になった。

### 該当プルリク

https://github.com/nus3/react-test-examples/pull/15

## Storybook: v6 → v8

Storybook も testing-library と同様に、[元の記事](https://blog.cybozu.io/entry/2022/08/29/110000)からバージョンが 6.5.6 から 8.0.8 に上がっていたので、次のような対応をした

- `pnpm dlx storybook@latest upgrade`
- Story の型定義を変更
- `@storybook/testing-react`の削除
- `@storybook/addon-actions`を`@storybook/test`の`fn`に置き換える
- `composeStories`による Story を使った Vitest のテストで play 関数を使わないように
- `@storybook/test-runner`を追加

### `pnpm dlx storybook@latest upgrade`

今回のアップデート作業で一番驚いた部分。

Storybook には自動で v8 にアップデートするためのコマンドが用意されており、このコマンドを実行することで関連するライブラリのアップデートや移行作業を CLI を通して自動で行なってくれる。
https://storybook.js.org/docs/migration-guide#automatic-upgrade

作業ごとにどういったことを Storybook がしてくれるのか懇切丁寧なメッセージを出力してくれる。出力結果を次のアコーディオンにまとめたので興味がある人はぜひ見てほしい。バージョンアップ作業によるユーザーの負担を減らすための Storybook の思いやりを感じるはずだ。

:::details storyStore の削除

[v7 から storyStore がデフォルトで使われている](https://github.com/storybookjs/storybook/blob/next/MIGRATION.md#storystorev7-enabled-by-default)ので、オプションで指定した場合は削除する。

```shell
❯ pnpm dlx storybook@latest upgrade
╭───────────────────────────────────────────────────────────────╮
│                                                               │
│   Upgrading Storybook from version 6.5.6 to version 8.0.8..   │
│                                                               │
╰───────────────────────────────────────────────────────────────╯
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
```

:::

:::details 関連するライブラリのバージョンアップ

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

:::

:::details @storybook/react-vite への移行

```shell
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

:::

:::details storybook-binary への移行

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

:::

:::details storybook のコマンド変更

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

:::

:::details @storybook/testing-library の削除

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

:::

:::details autodocs の有効化

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

:::

### Story の型定義を変更

次のように[`satisfies`](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html#the-satisfies-operator)を使って Story の定義をするように変更。

```ts
import type { Meta, StoryObj } from "@storybook/react";

import { Button, ButtonProps } from "./Button";

const meta = {
  component: Button,
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Primary: Story = {
  args: {
    primary: true,
    label: "Button",
  },
};
```

https://storybook.js.org/docs/writing-stories/args#story-args

### `@storybook/testing-react`の削除

Storybook で定義した Story がテストでも使える`composeStories`は`@storybook/react`で提供されるようになったので、`@storybook/testing-react`は削除。

https://github.com/storybookjs/testing-react?tab=readme-ov-file#%EF%B8%8F-attention

### `@storybook/addon-actions`を`@storybook/test`の`fn`に置き換える

以前は`@storybook/addon-actions`を使って、`onClick`などのイベントを Storybook 上で確認できるようにしていたが、同様のことが`@storybook/test`の`fn`でできるようになった。

https://storybook.js.org/docs/essentials/actions#via-storybooktest-fn-spy-function

```diff ts
- import { action } from "@storybook/addon-actions";
+ mport { fn } from "@storybook/test";

export const Default: Story = {
	args: {
		children: "label",
-		onClick: action("onClick"),
+		onClick: fn(),
	},
};
```

### `composeStories`による Story を使った Vitest のテストで play 関数を使わないように

次のテストのように、`composeStories`から Story を取得しつつ、Vitest 側で play 関数を実行すると、`Warning: An update to SelectBox inside a test was not wrapped in act(...).`問題が再発する。

```ts
import { composeStories } from "@storybook/react";
import * as ToastStories from "./Toast.stories";

const { Default } = composeStories(ToastStories);

test("should be show and hide toast in story", async () => {
  const { container } = render(<Default />);
  await Default.play({ canvasElement: container });
  expect(screen.getByRole("alert")).toBeInTheDocument();
});
```

これは`@storybook/test`と`@testing-library/react`で依存する`@testing-library/dom`のバージョンが揃っていないのが原因。

```shell
❯ pnpm why @testing-library/dom
Legend: production dependency, optional only, dev only

devDependencies:
@storybook/addon-interactions 8.0.8
└─┬ @storybook/test 8.0.8
  ├── @testing-library/dom 9.3.4
  └─┬ @testing-library/user-event 14.5.2
    └── @testing-library/dom 9.3.4 peer
@storybook/test 8.0.8
├── @testing-library/dom 9.3.4
└─┬ @testing-library/user-event 14.5.2
  └── @testing-library/dom 9.3.4 peer # testing-library/reactとバージョンが揃ってへんやんけ!
@testing-library/react 15.0.2
└── @testing-library/dom 10.0.0
@testing-library/user-event 14.5.2
└── @testing-library/dom 10.0.0 peer
```

`@storybook/test`と`@testing-library/react`の依存関係のアップデートは必ずしも同時に行われるわけではないだろうし、play 関数を含めたテストがしたい場合はテストランナーを分ける(`@storybook/test-runner`を使う)方が運用は安定しそう。

なので、今回は`composeStories`を使ってテストでは play 関数を実行しないようにし、`@storybook/test-runner`で play 関数のテストを実行できるようにした。

### その他

- `@storybook/test`の`useEvent`は Promise を返してくれる
- [v7 から`main.ts`は ESM 形式で定義できるようになった](https://github.com/storybookjs/storybook/blob/next/MIGRATION.md#esm-format-in-mainjs)
- [モノレポ構成の場合は Storybook 実行時にモジュール解決に問題が出るかもしれないので、`getAbsolutePath`で addon などを追加している](https://storybook.js.org/docs/faq#how-do-i-fix-module-resolution-in-special-environments)
- `@storybook/test-runner`のセットアップで`pnpm dlx playwright install`
- [`@storybook/test-runner`のデフォルトポートは 6006](`@storybook/test-runner`)

### 該当コミット

https://github.com/nus3/react-test-examples/pull/15/files/b232a187065e85b952554f2ba85aa22130890f72..b18c7d649cb143510ff17da4e9ba50da5f792845

## MSW: v0 → v2

MSW は[元の記事](https://blog.cybozu.io/entry/2022/08/29/110000)からバージョンが 0.44.2 から 2.2.13 に上がっていたので、[マイグレーションガイド](https://mswjs.io/docs/migrations/1.x-to-2.x/)を参考に、`rest`を`http`に変更した。

```diff ts
- import { rest } from 'msw';
+ import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';

+ const mockHandlers = [
+   http.get('/examples', () => {
+     return HttpResponse.json(
+       {
+         examples: [
+           { id: '1', name: 'nus1' },
+           { id: '2', name: 'nus2' },
+           { id: '3', name: 'nus3' }
+         ]
+       },
+       { status: 200 }
+     );
+   })
+ ];
+ const server = setupServer(...mockHandlers);

describe('/examples', () => {
-  const server = setupServer();
-  beforeEach(() => {
-    server.use(
-      rest.get('/examples', async (_req, res, ctx) => {
-        return res(
-          ctx.status(200),
-          ctx.json<GetExamplesResponse>({
-            examples: [
-              { id: '1', name: 'nus1' },
-              { id: '2', name: 'nus2' },
-              { id: '3', name: 'nus3' }
-            ]
-          })
-        );
-      })
-    );
-  });
})
```

MSW では v2 のアップデートで`http`をつかった際にレスポンスには標準の`Response`を使えるようになったが、今回はレスポンスを定義する上で便利なメソッドが提供されていて、利用が推奨されている`HttpResponse`で定義した。

https://mswjs.io/docs/basics/mocking-responses#using-httpresponse-class

### 該当コミット

https://github.com/nus3/react-test-examples/pull/13/commits/245b1bdd867f12be58295c2b3d7adca19cc2a59f

## あとがき

全体のコード量は多くなかったのに、それでも作業時間は合計すると半日ぐらいかかった。実際に運用しているプロダクトであればもっと時間がかかったんだろうな。

リポジトリの内容をがっつり変えてしまったので、[元の記事](https://blog.cybozu.io/entry/2022/08/29/110000)で参照している部分が軒並みリンク切れになってしまった。直すます。
