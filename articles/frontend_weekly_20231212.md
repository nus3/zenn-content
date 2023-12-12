---
title: "Safari 17.2のリリースなど: Cybozu Frontend Weekly (2023-12-12号)"
emoji: "🦓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CybozuFrontendWeekly", "frontend"]
published: false
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社フロントエンドエキスパートチームの [@nus3\_](https://twitter.com/nus3_) です。

## はじめに

サイボウズでは毎週火曜日に Frontend Weekly という「1 週間の間にあったフロントエンドニュースを共有する会」を社内で開催しています。

今回は、2023 年 12 月 12 日 の Frontend Weekly で取り上げた記事や話題を紹介します。

## StyleX のコードが公開

https://github.com/facebook/stylex

[StyleX](https://stylexjs.com/) のリリースに合わせて、GitHub で StyleX のコードが公開されました。

参加したメンバーが[調査した内容](https://zenn.dev/b4h0_c4t/scraps/771984492e7185)も踏まえて、StyleX がどういった API を提供しているのか話しました。

## ECMAScript 考古学をするときに

https://zenn.dev/cybozu_frontend/articles/ecmascript-archeology

先日開催された JSConf JP 2023 で [@sajikix](https://twitter.com/sajikix) が[「Intl の今までとこれから」](https://speakerdeck.com/sajikix/intlnojin-madetokorekara)という発表をしました。この記事は Intl の策定の歴史を調べる過程で得た「ECMAScript の仕様の歴史を追う方法」をまとめた記事です。

## dnd-kit: future of library & maintenance

https://github.com/clauderic/dnd-kit/issues/1194

dnd-kit に対して、メンテナンスは継続してやっていくのか、これからのロードマップがどうなるのか確認する issue が作られていました。最近 issue のトリアージや PR が捌けてないのは重要なリファクタリングをしているからとのことです。

11 月にリリースもされており、これからもメンテナンスは継続するようです。

## Web Performance Calendar

https://calendar.perfplanet.com/2023/

パフォーマンスに関するアドベントカレンダーが今年も開催されました。このアドベントカレンダーの中から次の記事について話しました。

- [Benchmarking, Profiling, and Optimizing JavaScript libraries](https://calendar.perfplanet.com/2023/benchmarking-profiling-and-optimizing-javascript-libraries/)
  - JS ライブラリのベンチマーク、プロファイリング、最適化について
- [Yielding to the Main Thread: How Breaking Up Tasks Can Fix INP](https://calendar.perfplanet.com/2023/yielding-main-thread-breaking-up-tasks-fix-inp/)
  - INP を向上させるためにブラウザのメインスレッドで実行する JS のタスクを分割する方法について
- [Fastest Way of Passing State to JavaScript, Re-visited](https://calendar.perfplanet.com/2023/fastest-way-passing-state-javascript-revisited/)
  - サーバーサイドレンダリングされた状態の初期値をクライアントサイドに渡す最速の方法について

## Deno Cron

https://deno.com/blog/cron

スケジュールされたジョブを簡単に作成できる Deno Cron がリリースされました。

Deno Deploy にデプロイしている場合、`Deno.cron()`を宣言することで簡単にジョブをスケジュールできます。

## CSS Snapshot 2023

https://www.w3.org/TR/2023/NOTE-css-2023-20231207/

CSS Snapshot 2023 が Note として公開されました。

この Note の中では、CSS の各モジュールの仕様が 2023 現在どういう状態なのかまとめられています。

## Safari 17.2

https://webkit.org/blog/14787/webkit-features-in-safari-17-2/

Safari 17.2 がリリースされました。

[Custom Highlights API](https://developer.mozilla.org/en-US/docs/Web/API/CSS_Custom_Highlight_API)など新機能の追加や、Interop 2023 の活動の一環として大量の CSS の修正がこのリリースに含まれています。

特に Interop 2023 による修正によって重点分野となっている CSS の新機能など、主要ブラウザの互換性が向上するのはとても良いですね。
