---
title: "Next.jsのCacheを網羅したドキュメントの公開など : Cybozu Frontend Weekly (2023-08-01号)"
emoji: "💴"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CybozuFrontendWeekly", "frontend"]
published: false
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社フロントエンドエキスパートチームの [@nus3\_](https://twitter.com/nus3_) です。

## はじめに

サイボウズでは毎週火曜日に Frontend Weekly という「1 週間の間にあったフロントエンドニュースを共有する会」を社内で開催しています。

今回は、2023/08/01 の Frontend Weekly で取り上げた記事や話題を紹介します。

## How React 18 Improves Application Performance – Vercel

https://vercel.com/blog/how-react-18-improves-application-performance

パフォーマンスの向上させるReact18 で新しく導入された transition, suspense, react server components などの機能の解説です。React17以前とReact18の`concurrent rendering`でのレンダリングの違いや、React18で導入された新機能が図やサンプルコードと合わせて紹介されています。

## i feel bittersweet sharing i’m leaving my job at meta in a few weeks.

https://twitter.com/dan_abramov/status/1682029195843739649?s=20

ReactチームにいたDanさんがmeta社を離れるとのことです。引き続きReactチームには関わっていくことや、ここ最近は新しいReactのドキュメント作成やコミュニティ周りの活動をされていたことがツイートで紹介されています。

## How Turborepo is porting from Go to Rust – Vercel

https://vercel.com/blog/how-turborepo-is-porting-from-go-to-rust

[Turborepo を Go から Rust へ移行する記事](https://vercel.com/blog/turborepo-migration-go-rust)の続編です。TurborepoではGoからRustに徐々に移行する戦略を採用しており、この移行による学びを共有していくれています。良かった点として新しい機能をリリースしながら徐々に移行作業が出来たこと、大変だった点としてOSなどさまざまなコンテキストすべててRustとGoのバイナリが動作するか確認する必要があったことが挙げられています。

## Node.jsのv16が9/11でEOLになります

https://endoflife.date/nodejs

## Bunのv0.7.0のリリース

https://bun.sh/blog/bun-v0.7.0

Bunのv0.7.0がリリースされ、Vite, WebWorker, structuredClone, AsyncLocalStorage などが使えるようになりました。

## PartyKit

https://partykit.io/

リアルタイムやマルチプレイヤー向けのアプリケーションを作るための SDK が公開されました。このSDKでは、Edge も含めた多様なランタイムや、Yjs のサポート入っています。

## Security alert: social engineering campaign targets technology industry employees - The GitHub Blog

https://github.blog/2023-07-18-security-alert-social-engineering-campaign-targets-technology-industry-employees/

GitHubが「パッケージやリポジトリを利用した攻撃」について注意するアナウンスを出しています。具体的には、開発者や採用担当者になりすました攻撃者が、企業の従業員などを悪意のあるパッケージを含んだリポジトリへ招待し、そのリポジトリをクローンし実行することでウイルスに感染させるという手法のようです。GitHubが行なっている調査や対策、自分たちができること、悪意のあるパッケージやアカウントのリストなどが紹介されています。

## Astro 2.9

https://astro.build/blog/astro-290/

Astro 2.9がリリースされました。2.9で実装されたサポートされていないブラウザのことも考慮しつつ、View Transitions APIが使えるView Transitionsについて特に社内では話していました。View Transitionsについてはこちらの記事が参考になります。

https://zenn.dev/cybozu_frontend/articles/astro-view-transitions

## Introducing Valibot, a < 1kb Zod Alternative

https://www.builder.io/blog/introducing-valibot

Zodよりもファイルサイズが抑えられるValibotの紹介です。Zodとは違い、バリデーションの機能をチェーンメソッドではなく関数で提供することでTree Shakingが有効になり、ビルド後のファイルサイズを抑えられるそうです。社内では試しにZodを使ったプロダクトで代わりにValibotを使うとファイルサイズが10分の1程度になったことなどを話していました。


## Jotaiで快適フロントエンド開発 | 株式会社ヌーラボ(Nulab inc.)

https://nulab.com/ja/blog/nulab/react-jotai/

Jotaiの基本や、RecoilからJotaiに移行によるJotaiとRecoilの比較が紹介されています。Recoilでは現状、主要なメンテナーがレイオフの対象になって開発が止まっているとのことで、状態管理の技術選定に影響がありそうですね。

## Arc from The Browser Company

https://arc.net/

[ブラウザのArcがリリース](https://twitter.com/arcinternet/status/1683841503544897538?s=20)され、利用できるようになりました。ナビゲーションにはサイドバーがメインで使われたりSpaceやEasel、ユーザープライバシーを重視していることが特徴として挙げられます。

## Building Your Application: Caching

https://nextjs.org/docs/app/building-your-application/caching

Next.jsのCacheを説明したページがドキュメントに追加されました。Next.jsで利用できるCacheの種類や、どのAPIがどのCacheに影響を与えるのかが図や表で網羅されています。Next.jsでのCacheは種類がいくつかあるので、全体を理解したいときに参考になりそうです。

## Build and Ship Astro Sites with Deno and Deno Deploy

https://deno.com/blog/astro-on-deno

AstroのサイトをDenoとDeno Deployを使ってリリースする方法が紹介されています。AstroのSSRを使う場合は[テンプレート](https://github.com/denoland/deno-astro-template)が用意されており、このテンプレートを使うとDeno KVなどがすぐ使えるようになっています。
