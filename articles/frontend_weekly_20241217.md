---
title: "Baseline2024のふりかえりなど: Cybozu Frontend Weekly (2024-12-17号)"
emoji: "⛄️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CybozuFrontendWeekly", "frontend"]
published: false
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社フロントエンドエキスパートチームの [@nus3\_](https://twitter.com/nus3_) です。

## はじめに

サイボウズでは毎週火曜日に Frontend Weekly という「1 週間の間にあったフロントエンドニュースを共有する会」を社内で開催しています。

今回は、2024 年 12 月 17 日 の Frontend Weekly で取り上げた記事や話題を紹介します。

## コピペってなんだろう？Clipboard 編

https://zenn.dev/irico/articles/d2782f5ad615f0

- 弊社フロントエンドエンジニアの[irico](https://x.com/poyoyofever)の記事
- OS ごとのクリップボードの大まかな仕組みを解説
- この後も２本後続の記事を執筆予定とのこと

## Next.js 15.1

https://nextjs.org/blog/next-15-1

- Next.js 15.1 がリリース
- React19 の Stable に対応
- Stream 完了時点での処理を保証する `after()`
- `notFound()` の類似系で `forbidden()` `unauthorized()` の Experimental サポート

## Node v23.4.0

https://nodejs.org/en/blog/release/v23.4.0

- Node.js v23.4.0 がリリース
- assert.partialDeepStrictEqual が試験的に追加
- `--trace-env`、`--trace-env-js-stack`、および `--trace-env-native-stack` CLI オプションが導入

## Next.js PPR と比較して理解する Astro Server Islands

https://zenn.dev/morinokami/articles/astro-server-islands-vs-nextjs-ppr

- Astro と Next.js のレンダリングモデルがサンプルコード交えて詳しく解説されている
- Astro Server Islands は動的コンテンツをページ部分ごとにレンダリングする機能であり、Next.js Partial Prerendering (PPR) と仕組みが似ているが、動的なコンテンツを取得する主体がブラウザであるという点で異なっている

## Form Validation That Doesn't Annoy Users: CSS :user-valid and :user-invalid

https://www.trevorlasn.com/blog/css-user-valid-and-user-invalid-pseudo-classes

- ユーザのインタラクション後に発火する:user-valid, :user-invalid について、従来の:invalid, :valid との挙動を比較しながら説明している記事
- @support による、安全なオプトイン方法についても述べられている

## Baseline 2024: more tools to help web developers

https://web.dev/blog/baseline-project-2024

- Baseline2024 のふりかえり
- Baseline のデータソースである web-features の各機能のマッピングが 81%完了
- [Web Platform Status](https://webstatus.dev/)
- `<baseline-status>`Web コンポーネント
- Baseline プロジェクトの公式ページとロゴを作成
- [RUM Insights](https://rumarchive.com/insights/)の Baseline support

## The 2024 Web Almanac

https://almanac.httparchive.org/en/2024/

- Web Almanac 2024 のレポートが公開

## Build Qwik with Deno

https://x.com/deno_land/status/1868691986175431120

- Deno で Qwik を使ってアプリを作成するガイドがドキュメントに追加
- さまざまなフレームワーク・ライブラリ・環境で Deno を使うためのガイドが追加されている
  - https://docs.deno.com/examples/

## Astro 5.0

https://astro.build/blog/astro-5/

- Astro 5.0 がリリース
- Content Layer
  - Astro プロジェクトでコンテンツを定義、ロード、アクセスするためのタイプセーフな API
- Server Islands
- Vite 6 対応
- SVG コンポーネントが experimental で実装

## google.com の証明書の誤発行が CT で見つかる

https://bugzilla.mozilla.org/show_bug.cgi?id=1934361

- 1934361 - ICP-Brasil: Mis-issued certificate
- ブラジルの CA が google.com の証明書を誤って発行する
- 発行した履歴を保存する CT (Certificate Transparency)の監視によって発覚した
- 発行した CA は音信不通
- CT を行う最大のモチベーションが誤発行の発見なので、典型的な事例の一つ

## A Note from our Executive Director - Let's Encrypt

https://letsencrypt.org/2024/12/11/eoy-letter-2024/

- 2025 は Let's Encrypt 10 周年、 5 億サイトが採用
- 来年、有効期限が 6 日の新しい証明書サービスを始める予定
- 現在の 20 倍、 500 万枚/日の発行に耐えうる準備を進めている
- 証明書がめっちゃ短い時代が到来する可能性がある

## ECMA-426 1st edition – Source map format specification

https://ecma-international.org/publications-and-standards/standards/ecma-426/

- 新しく JavaScript、WebAssembly、および CSS のソースマップ形式の仕様が策定された
