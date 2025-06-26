---
title: "Baseline2024のふりかえりなど: Cybozu Frontend Weekly (2025-06-24号)"
emoji: "🐸"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["CybozuFrontendWeekly", "frontend"]
published: false
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社フロントエンドエキスパートチームの [@nus3\_](https://twitter.com/nus3_) です。

## はじめに

サイボウズ社内では毎週火曜日に Frontend Weekly と題し「一週間の間にあったフロントエンドニュースを共有する会」を開催しています。

今回は、2025/06/24 の Frontend Weekly で取り上げた記事や話題を紹介します。

##　 Roo Code 3.21.4 Release Notes

https://docs.roocode.com/update-notes/v3.21.4

- Roo Code 3.21.4 がリリース
- Claude Code Provider を介して Claude Max プランを Roo Code で利用できるように

## guide to Scroll-driven Animations with just CSS | WebKit

https://webkit.org/blog/17101/a-guide-to-scroll-driven-animations-with-just-css/

- Scroll-driven Animations のガイドが WebKit のブログから公開
- Scroll-driven Animations は Safari 26 beta で利用可能
- プログレスバーや画像のスライドインなど実例も交えて紹介

## Firefox Nightly 142.0a1 Release Notes

https://www.mozilla.org/en-US/firefox/142.0a1/releasenotes/#note-790833

- Firefox 139 の Nightly build から View Transitions API がサポート

## Intent to prototype: Framebusting Intervention

https://groups.google.com/a/mozilla.org/g/dev-platform/c/G0GDGqJCGpM

- iframe から top level のページ遷移を制限する実装のプロトタイプが Gecko で進められいる

## Announcing Vitest 3.2

https://vitest.dev/blog/vitest-3-2.html

- Vitest 3.2 がリリース
- workspace が deprecated になり、projects に統一
- 新しい annotate API が追加され、GitHubActions やレポート上でテストに紐づけて、メッセージや画像を指定できるように
- project ごとに色を指定できるように
- などなど

## @msw/playwright がリリース

https://github.com/mswjs/playwright

- Playwright で API をモックする際により良い開発者体験を提供する MSW 統合パッケージ
- @msw/playwright を使うことで、playwright のテストの中で worker にアクセスできるように

## Linux Foundation Launches the Agent2Agent Protocol Project to Enable Secure, Intelligent Communication Between AI Agents

https://www.linuxfoundation.org/press/linux-foundation-launches-the-agent2agent-protocol-project-to-enable-secure-intelligent-communication-between-ai-agents

- Agent2Agent Protocol Project という、AI Agent 間の相互互換性を向上させようというプロジェクトが Google から Linux Foundation に寄贈
- ベンダ非依存のプロジェクトになった
- AWS や MS, Salesforce など大手サービスベンダが協力して、プロジェクトを進める

## Cloudflare Containers - Open Beta Launch Event

https://containers.cloudflare.com/

- Cloudflare Containers のパプリックベータがリリース

## Prettier 3.6: Experimental fast CLI and new OXC and Hermes plugins! · Prettier

https://prettier.io/blog/2025/06/23/3.6.0

- Prettier 3.6 がリリース
- `--experimental-cli`フラグを有効化することで experimental なハイパフォーマンスな CLI を利用可能に
- @prettier/plugin-oxc と @prettier/plugin-hermes という公式プラグインがリリース

## Playwright 1.53.0

https://github.com/microsoft/playwright/releases/tag/v1.53.0

- Playwright 1.53.0 がリリース
- Trace Viewer と HTML レポーターで確認できる`locator.describe()`が追加

## The New Separation of Concerns

https://bradfrost.com/blog/post/the-new-separation-of-concerns/

- Brad Frost 氏が CSSDay で提唱した、Design Token System の記事
- 色やタイポグラフィなど Aesthetic なスタイルを Design Token System、そのほかの機能的な部分を Component System に分離
- 関心を分離することで、さまざまなプロダクトやブラントに応じた Theming 可能なデザインシステムが作りやすくなるという考え方

## TSKaigi Hokuriku 2025 キックオフ

https://typescript-jpc.connpass.com/event/359888/

- TSKaigi Hokuriku 2025 のスタッフ希望者向けの説明会とキックオフイベントが開催

## Bluesky Likes Web Components • Lea Verou

https://lea.verou.me/blog/2025/bluesky-likes/

- Bluesky のいいねを表示するために Web Components を作った
- Web Components で開発する際の注意点や苦労した点を紹介

## Intent to Prototype: Customized built-in elements via elementInternals.type

https://groups.google.com/a/chromium.org/g/blink-dev/c/6srW418CDgs

- Custom Elements でビルトイン要素を拡張可能にする仕組みの実装が進んでいる

## アップル、ＥＵから新たな制裁金の可能性－ＤＭＡ違反継続なら

https://www.bloomberg.co.jp/news/articles/2025-06-16/SXXXRNT0G1KW00

- アップルが EU のデジタル市場法（DMA）違反で 4 月に 5 億ユーロの制裁金を科され、速やかに是正しなければ新たな制裁金の可能性
- EU は AppStore 以外の割安な商品やキャンペーンに開発業者がユーザーを誘導することを認めるよう要請している

## Biome v2

https://biomejs.dev/blog/biome-v2/

- Biome v2 がリリース
- 複数ファイルを跨ぐ情報が必要なルールや、型情報が必要なルールなどに対応
- モノレポのサポートが改善
- HTML Formatter が完成
- GritQL でカスタムルールを作成できる Linger Plugin

## WebAIM: Up and Coming ARIA

https://webaim.org/blog/up-and-coming-aria/

- ARIA 1.3 (Editor's) を中心に注目/知名度の低い機能を取り上げ、各 AT の実装状況と合わせて紹介

## Stylable select is here!

https://www.gwhitworth.com/posts/2025/styleable-select-is-here/

- stylable な select 要素の実現が近いことを OpenUI の chair で[Customizable Select Element](https://open-ui.org/components/customizableselect/)を提案した Greg Whitworth 氏が紹介
