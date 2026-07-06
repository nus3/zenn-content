---
title: "Vitestで始めるシンプルなVRT"
emoji: "🖼️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vitest", "vrt"]
published: true
publication_name: "cybozu_frontend"
---

この記事は、[CYBOZU SUMMER BLOG FES '26](https://summer-blog-fes.cybozu.io/2026/)の記事です。

## はじめに

[Vitest 4.0](https://voidzero.dev/posts/announcing-vitest-4#additional-updates)でVisual Regression Testing（VRT）がサポートされました。このVitest（Browser Mode）でのVRTとGitHub Actionsを組み合わせることで、実際のプロダクトにシンプルなVRTを導入できたので、その方法を紹介します。

今回紹介する方法を導入した導入したサンプルのリポジトリを作ったので、詳細な実装が知りたい方はご覧ください。
https://github.com/nus3/vitest-vrt

## VRT実行の流れ

今回は次の流れでVRTを実行します。

1. VitestによるVRTをGitHub Actionsで手動、もしくはスケジュールで実行
2. 初回実行時は対象コンポーネントのスクリーンショットを保存し、gitで管理
3. 2回目以降のVRTでは差分が発生すると、ベースの画像を更新するプルリクエストを作成
4. 人間がプルリクエスト上で画像の差分を確認
5. 差分に問題なければプルリクエストをマージし、ベースの画像を更新

## VRT導入のためのパッケージを追加

VitestのVRTは[Browser Mode](https://vitest.dev/guide/browser/why.html)で実行されるので、まずはBrowser Modeに必要な次のパッケージを追加しましょう。

- `@vitest/browser`
- `@vitest/browser-playwright`
- `playwright`
- `vitest-browser-react`

### Browser Modeのproviderの指定

Vitestで[Browser Modeを利用する場合、必ずproviderの指定が必要](https://vitest.dev/guide/browser/#manual-installation)です。providerには`playwright`と`webdriverio`を選択できます。今回は`playwright`を利用するので、パッケージも`@vitest/browser-playwright`と
`playwright`を追加します。

WebdriverIOを採用しているプロダクトであればBrowser Modeのproviderにも`webdriverio`を採用した方が良いかもしれませんが、そうでない場合はPlaywrightをproviderとして利用するのが[Vitestのドキュメント](https://vitest.dev/config/browser/webdriverio.html)では推奨されています。

> If you do not already use WebdriverIO in your project, we recommend starting with Playwright as it is easier to configure and has more flexible API.

### Browser Modeでコンポーネントを描画する

Browser Modeでは次のようにReactやVue、SvelteのコンポーネントをBrowser Modeで描画するためのパッケージがVitestから提供されています。

- [`vitest-browser-react`](https://vitest.dev/api/browser/react.html)
- [`vitest-browser-vue`](https://vitest.dev/api/browser/vue.html)
- [`vitest-browser-svelte`](https://vitest.dev/api/browser/svelte.html)

テスト時のコンポーネントの描画には、よく[`@testing-library/react`](https://github.com/testing-library/react-testing-library)のような[Testing Library](https://testing-library.com/)が利用されます。これらのパッケージはVitestのBrowser Modeでも引き続き利用できます。

引き続き利用できるんですが、[`vitest-browser-react`](https://vitest.dev/api/browser/react.html)などのVitestが提供するパッケージにはBrowser Modeの機能と連携するAPIが用意されてるので、移行にコストがかかるなど特段の理由がなければ、こちらのパッケージを利用するのをおすすめします。

### Browserのインストール

`provider`には`playwright`を指定したので、VitestのBrowser Modeで利用するブラウザをインストールする必要があります。CI（GitHub Actions）上でもブラウザのインストールは必要なので、npm scriptsに次のコマンドを追加しておきましょう。

```jsonc
// package.json
{
  "scripts": {
    "dev:setup": "playwright install --with-deps chromium",
  },
}
```

[`--with-deps`](https://playwright.dev/agent-cli/installation#install-options) は Linuxに必要な依存ライブラリも一緒にインストールしてくれます。

#### Browserバイナリのキャッシュについて

Playwrightでインストールしたブラウザのバイナリをキャッシュすることは[推奨されていません](https://playwright.dev/docs/ci#caching-browsers)。

> Caching browser binaries is not recommended, since the amount of time it takes to restore the cache is comparable to the time it takes to download the binaries.

基本的には`--with-deps`を使ったブラウザのインストールをCIに組み込みましょう。ただ、（自分たちのプロダクトでもたまに発生したんですが）`--with-deps`を使ったブラウザのインストール時にタイムアウトやエラーが発生するケースがあります。その場合は次の記事に記載されているようにPlaywrightが提供する公式のDocker Imageを採用するのを検討しても良いかもしれません。

https://tech.newmo.me/entry/playwright-ci-optimization

## VRT用の設定を追加

必要なパッケージが追加できたら、Vitestの[Projects](https://vitest.dev/guide/projects)を使い、VRT用のprojectを追加します。

```ts: vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import { playwright } from "@vitest/browser-playwright";

export default defineConfig({
  test: {
    projects: [
      {
        plugins: [react()],
        test: {
          name: "vrt",
          setupFiles: ["./vitest-vrt-setup.ts"],
          include: ["tests/vrt/**/*.test.{ts,tsx}"],
          browser: {
            provider: playwright(),
            // Browser Modeを有効にする
            enabled: true,
            headless: true,
            instances: [{ browser: "chromium" }],
            viewport: { width: 1280, height: 720 },
          },
        },
      },
    ],
  },
});
```

`"vitest --project vrt"`のようにVitest実行時に[`--project`](https://vitest.dev/guide/cli.html#project)オプションをつけることで指定したprojectの設定でテストが実行されます。今回は次のような設定を追加しています。

- テスト実行時に`./vitest-vrt-setup.ts`を実行する
- テスト対象は`tests/vrt/**/*.test.{ts,tsx}`に限定
- Browser Modeを有効化し、Chromiumでheadlessに実行する
- ビューポートのサイズを1280x720に指定する

VRTを実行する際のベストプラクティスがVitestの[公式ドキュメント](https://vitest.dev/guide/browser/visual-regression-testing.html#best-practices)に記載されているので、いくつか紹介します。

### アニメーションの無効化

アニメーションがある画面やコンポーネントをVRTの対象にすると、スクリーンショットのタイミングによって毎回、差分が検出されてしまうことがあります。
次のようなsetupファイルを追加することで、アニメーションを無効化することができます。

```ts: vitest-vrt-setup.ts
import { beforeAll } from "vitest";

beforeAll(() => {
  const style = document.createElement("style");
  style.textContent = `
    *, *::before, *::after {
      animation-duration: 0s !important;
      animation-delay: 0s !important;
      transition-duration: 0s !important;
      transition-delay: 0s !important;
    }
  `;
  document.head.appendChild(style);
});
```

https://vitest.dev/guide/browser/visual-regression-testing.html#disable-animations

### VRTでの閾値を調整する

プロジェクト全体とコンポーネントごとで必要に応じて見た目の差分の閾値（`allowedMismatchedPixelRatio`）を調整できます。

プロジェクト全体で設定する場合

```ts diff: vite.config.ts
test: {
  browser: {
+   expect: {
+     toMatchScreenshot: {
+       comparatorOptions: {
+         allowedMismatchedPixelRatio: 0.1,
+       },
+     },
+   },
  },
},
```

コンポーネントごとに設定する場合

```ts
await expect(page.getByTestId("article-summary")).toMatchScreenshot({
  comparatorName: "pixelmatch",
  comparatorOptions: {
    allowedMismatchedPixelRatio: 0.1,
  },
});
```

https://main.vitest.dev/guide/browser/visual-regression-testing.html#set-appropriate-thresholds

## VRTを実行するテストコードを追加

VRT用の設定を追加できたらVRT用のテストケースを追加しましょう。

VRT用のテストケースでは、対象のコンポーネントを`render`し、[`toMatchScreenshot`](https://vitest.dev/api/browser/assertions.html#tomatchscreenshot)を実行します。この`toMatchScreenshot`は次のような挙動になります。

- 初回は対象コンポーネントのスクリーンショットを保存
- 2回目以降は前回保存したスクリーンショットと見た目を比較、差分がある場合にテストが失敗する

```tsx
import { describe, expect, it } from "vitest";
import { page } from "vitest/browser";
import { render } from "vitest-browser-react";

import { Counter } from "../../src/components/Counter";

describe("Counter (VRT)", () => {
  it("初期表示のスクリーンショット", async () => {
    const { container } = await render(<Counter initialCount={5} />);
    await expect.element(page.getByTestId("count")).toBeVisible();

    // 対象のコンポーネントのスクリーンショットを撮影し、前回のスクリーンショットと比較する
    await expect(container).toMatchScreenshot("counter-initial.png");
  });
});
```

## CIでのVRTの実行

GitHub Actions上でVRTを実行するために、次の二つのコマンドをnpm scriptsに追加しましょう。

```jsonc: package.json
{
  "scripts": {
    "test:vrt": "vitest --project vrt",
    "test:vrt:update": "vitest --project vrt --update",
  },
}
```

`test:vrt`でVRTを実行し、差分が発生（テストが失敗）した場合は、[`--update`](https://vitest.dev/guide/snapshot.html#updating-snapshots)オプションを追加した`test:vrt:update`を実行することで、前回のスクリーンショットを更新します。

### 変更差分の確認

実際にVRTが失敗した場合には、どういった差分が発生したのかを目で見て確認したいです。この差分の確認に今回はGitHubのプルリクエスト上のFiles changedを利用します。Files changedでは次のように変更があった画像をSwipeやOnion Skinで確認することができます。

![GitHubのFiles changedでの画像の差分を確認できる](/images/vitest-simple-vrt/github-image-changed.gif)

今回は、このGitHub上のFiles changedでの差分を確認する方法を採用します。もちろん差分を簡単に確認できるレポートなどあった方が便利なので、運用していく中でGitHubのプルリクエスト上で差分を確認するのがしんどくなってきた場合はChromatic等のVRTのレポート機能を提供しているツールやライブラリを検討するのも良いと思います。

### GitHub Actionsを使ってVRTを実行する

次のようなワークフローを作成することで、VRTの実行から差分が発生した際にプルリクエストを作成するまでをGitHub Actionsで完結できます。

1. 手動実行、もしくはスケジュールでVRTのワークフローを実行
2. VRTの実行(テストが失敗しても後続のjobを実行する)
3. `--update`によるVRTのベース画像を更新
4. ベース画像を更新するプルリクエストを作成

ワークフロー全体は次のファイルに記載しています。（Vite+を試してみたくて`vp`がセットアップには使われてますが適宜読み替えてもらえると..）  
https://github.com/nus3/vitest-vrt/blob/main/.github/workflows/vrt.yml

実際に、このワークフローによって自動で作成されたプルリクが次になります。  
https://github.com/nus3/vitest-vrt/pull/6/changes

サンプルリポジトリのワークフローではプルリクの自動作成まででしたが、実際のプロダクトでは、このワークフローにslackへの通知を追加しています。

![SlackにVRTの実行結果を通知する](/images/vitest-simple-vrt/slack-notify.png)

## Vitestでのtips

このほかにもVitestを導入したプロダクトでのtipsを次にまとめました。

### VitestでのFlakyテストの検出

Vitestの[Custom Reporters](https://vitest.dev/guide/reporters.html#custom-reporters)を使い、Browser Modeとjsdom環境の両方でFlakyなテストを検出する仕組みを導入しています。

https://github.com/nus3/vitest-vrt/blob/e5e3a147738df65eb41bc5abec2a5e1ee1dfb1dd/vitest-flaky-reporter.ts#L22-L43

このCustom Reportersの中で[`diagnostic`](https://vitest.dev/api/advanced/test-case.html#diagnostic)を使うことで対象のテストケースがretryでパスするようなflakyなテストかどうかを判定しています。

サンプルリポジトリでは、flakyなテストが検出された場合に、[プルリクエスト上でコメント](https://github.com/nus3/vitest-vrt/pull/5#issuecomment-4888505156)するようにしています。

![プルリクエスト上でflakyなテストをコメントする](/images/vitest-simple-vrt/pr-comment.png)

実際のプロダクトでも最初はプルリクエスト上でコメントしていたのですが、多くのチームが関わっているプロダクトで、特に全体に影響があるようなファイルを変更した際に自チームと関係がないflakyなテストを通知してしまうケースが多発しました。現在は専用のチャンネルをSlack上に作り、そこにflakyなテストがあったことを通知する運用に変更しています。

![Slack上でflakyなテストを通知する](/images/vitest-simple-vrt/flaky-test-report.png)

### PlaywrightのTrace Viewerを使ったBrowser Modeでのテストのデバッグ

[Vitest 4.0](https://voidzero.dev/posts/announcing-vitest-4#additional-updates)のリリースでは、Browser ModeでのVRT以外に[PlaywrightのTrace Viewer](https://playwright.dev/docs/trace-viewer)がサポートされています。

次のようにVitestのBrowser Modeの設定で`trace: "on"`にすることで、Browser Modeでのテスト実行時に実際にどのようにブラウザ上でテストが実行されているかを保存したzipファイルが生成されます。

https://github.com/nus3/vitest-vrt/blob/e5e3a147738df65eb41bc5abec2a5e1ee1dfb1dd/vite.config.ts#L68-L69

`playwright show-trace path/to/trace.zip`で生成されたzipファイルを指定することで、Trace Viewerが起動します。サンプルリポジトリでは、Trace Viewer用のnpm scriptを追加しているので以下のコマンドでもTrace Viewerを起動できます。

```bash
pnpm test:trace path/to/trace.zip
# 例
pnpm test:trace tests/browser/__traces__/ContactForm.test.tsx/browser--chromium--ContactForm--browser---------------------0-0.trace.zip
```

Trace Viewerでは、[StorybookのPlay function](https://storybook.js.org/docs/writing-stories/play-function)のようにテストのステップごとに、どのような操作が行われたかを視覚的に確認できるのでBrowser Modeでflakyなテストが発生した場合のデバッグに役立ちます。

実際にTrace Viewerを使って、フォームの送信ボタンをクリックした際にエラーメッセージが表示されることを確認するテストをデバッグしている様子が次になります。

![Trace Viewerでステップごとにデバッグする](/images/vitest-simple-vrt/trace-viewer.gif)

## 最後に

今回紹介した方法はVitestのBrowser ModeとGitHub ActionsのみでシンプルにVRTが組み込める点が良いなと感じています。

Vitestを採用していて、VRTの導入できてないなぁというプロダクトがあれば、ぜひ試してみてください！
