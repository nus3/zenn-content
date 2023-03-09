---
title: "Denoでpackage.jsonがサポートされたなど : Cybozu Frontend Weekly (2023-03-07号)"
emoji: "⛓️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CybozuFrontendWeekly", "frontend"]
published: true
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社フロントエンドエキスパートチームの [@nus3\_](https://twitter.com/nus3_) です。

# はじめに

サイボウズでは毎週火曜日に Frontend Weekly という「1 週間の間にあったフロントエンドニュースを共有する会」を社内で開催しています。

今回は、2023/03/07 の Frontend Weekly で取り上げた記事や話題を紹介します。

# 取り上げた記事・話題

## Bun has docs now

https://twitter.com/colinhacks/status/1629310598004772865?s=20

Bun の[ドキュメントサイト](https://bun.sh/docs)が公開されました。

このドキュメントサイトでは、次の内容が記載されています。

- Bun の概要、インストール方法
- CLI
- Runtime について
- 各種エコシステムとの互換性
- 提供する API など

Quick start なども記載されているので、気軽に Bun を試せそうです。

## I've joined @rometools🚀

https://twitter.com/nissy_dev/status/1631222211980955649

弊社の[@nissy_dev](https://twitter.com/nissy_dev)が Rome のメンテナーになりました 🎉。

Rome の活動の大半を締めていた Micha Reiser 氏が [Rome を離れる](https://twitter.com/MichaReiser/status/1613474278808162304?s=20)など、今後の動向が気になっていた Rome ですが、2 月ぐらいから OSS として活動が継続されているそうです。近いうちに[12.0.0 のリリース](https://github.com/rome/tools/pull/4002)を予定しているそうです。

## Connect for Node.js is now available

https://buf.build/blog/connect-node-beta

Connect for Node.js がベータ版としてリリースされました。このリリースにより、Connect-Web と合わせてクライアントサイド・サーバーサイドの両方で TypeScript での実装が可能になりました。

## Getting Started with Style Queries

https://developer.chrome.com/blog/style-queries/

コンテナ要素のスタイルに合わせて、子要素のスタイルを指定できる Style Queries が Chrome111 から試せるようになります。Chrome111 では、次のサンプルコードのようにカスタムプロパティを利用する場合のみ動くようです。

```css
/* 親要素の--themeがwarmの場合に適応されるスタイル */
@container style(--theme: warm) {
  .card {
    background-color: wheat;
    ...;
  }
}
```

## Today on web.dev we've launched Learn Privacy

https://twitter.com/ChromiumDev/status/1628391117870825475

[web.dev](https://web.dev/)に[Learn Privacy](https://web.dev/learn/privacy/)のコースが追加されました。

Web アプリケーションを構築する上でなぜプライバシーを学ぶことが重要なのか、データの管理・運用方法、フィンガープリントについてなどを交えつつ、プライバシーを保護するためのベストプラクティスが紹介されています。

また、[Learn HTML](https://web.dev/learn/html/) も全てのページが公開されました。新たに、Forms や Focus、Dialog などが追加されています。

> I've been writing HTML for a very long time, and I learned things while editing this course. There really is a lot more to HTML than you might think, and it's definitely worth revisiting some of the things you think you already know.

これは[All of Learn HTML! is available](https://web.dev/learn-html-available/)の最後部分です。2022 年に主要なブラウザ全てで実装された `<dialog>` もそうですが、HTML について自分が知っていると思っていた部分でも、新たな仕様が追加されていたりするので、改めてこのコースで見直してみるのも良さそうと社内では話していました。

## Deno 1.31: package.json support

https://deno.com/blog/v1.31

Deno の 1.31 で package.json がサポートされました。これにより、package.json に記載された依存をコードや `scripts` で利用できます。

また、このほかに、指定した依存関係をバンドルする `deno bundle` が非推奨になりました。Deno がこれからも npm や Node.js の組み込みモジュールをサポートし続けることで、すでにある優れたバンドラーが利用できるので、Deno のランタイムからこの機能を削除することを決めたそうです。

## Next.js 13.2

https://nextjs.org/blog/next-13-2

Next.js 13.2 がリリースされました。

`app` ディレクトリ内で静的、動的な metadata を定義できる新しい API や、`<Link>` コンポーネントのリンクの型付け、Next.js Cache のベータ版が追加されています。

社内では、コンポーネント(React Server Components)単位で ISR が設定できる Next.js Cache が話題になりました。

## Moving From Vue 1 To Vue 2 To Vue 3: A Case Study Of Migrating A Headless CMS System — Smashing Magazine

https://www.smashingmagazine.com/2023/03/vue-case-study-migrating-headless-cms-system/

Headless CMS の [storyblok](https://www.storyblok.com/) が実施した Vue1→Vue2→Vue3 のマイグレーションの話です。

Vue1 から Vue2 と Vue2 から Vue3 では全く違うアプローチをとっており、Vue1→Vue2 の移行では UI も含めフルリニューアルし、Vue2→Vue3 の移行では UI の変更なしに行われたそうです。それぞれのアプローチでよかったことや全体を通して大事だったことが紹介されています。

具体的には次のようなことが挙げられています。

- フルリニューアルを経て、プロダクトのアーキテクチャや品質、方向性に直接関われるように
- 新しいデザインや機能を段階的にゆっくりと導入することで、ユーザーに徐々に慣れてもらう
  - また、利用するユーザーからのフィードバックをもらえる
- 移行を成功させるためにはオンボーディングは不可欠
- UI を変更しない Vue2 から Vue3 の移行には E2E や Testing Library が役立った

ちなみに、Vue1 から Vue2 のフルリニューアルが完了するまで約 2 年ほどかかり、Vue2 から Vue3 への移行は約 8 ヶ月ほどかかったそうです。

# あとがき

TypeScript のみで gRPC ベースでのアプリケーションの実装が可能になる Connect for Node.js と Connect-Web がとても気になりました。Connect for Node.js を使えばクライアントサイドとサーバーサイドの両方を TypeScript で実装できるので、普段フロントエンドだけを開発してる人でもとっつきやすそうで良さそうですね。

---

フロントエンドエキスパートチームでは毎月、最終火曜日の 17 時から Frontend Monthly というイベントを YouTube Live で開催しています。その月のフロントエンド注目ニュースやゲストを呼んでの対談などフロントエンドに関する発信していますので是非どうぞ！

https://cybozu.github.io/frontend-monthly/

またフロントエンドエキスパートチームでは、一緒にサイボウズのフロントエンドを最高にする仲間を募集しています。興味ある方はこちら ↓ から！

https://cybozu.github.io/frontend-expert/
