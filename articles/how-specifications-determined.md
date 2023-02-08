---
title: "その仕様はどこから？"
emoji: "📜"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["html", "css", "javascript", "ecmascript"]
published: false
publication_name: "cybozu_frontend"
---

## HTML

### 未整理メモ

- 現在では、WHATWG が策定する HTML Living Standard が HTML の標準仕様になっている
  - https://html.spec.whatwg.org/
- HTML に関しては、Living Standard なので、下記リポジトリのコミットを見ればいい
  - https://github.com/whatwg/html

#### https://html.spec.whatwg.org/multipage/introduction.html#introduction

- 1990~1995 年、最初は CERN でホストされ、次に IETF でホストされ、拡張されてきた
- W3C 発足と共に HTML3.2 や HTML4 の策定が続いた
- ちょっと停滞した
- HTML5 の策定作業中に Mozilla と Opera の共同で W3C に提案したけど却下された。W3C は XML ベースの代替品の開発を継続
- WHATWG と呼ばれる組織が誕生、Apple、Mozilla、Opera 共同で継続する意向を発表
- 2006 年に W3C が HTML5 の開発に興味を示して、WHATWG と W3C は協力して、開発してた
- WHATWG は仕様をプラットフォームの進化に合わせて新しい機能を追加、Living Standard としたかった
- W3C は HTML5 の完成バージョンを公開したかった
- 2019 年に WHATWG と W3C は、今後 HTML の単一バージョンに関して共同作業を行う署名をした

#### https://whatwg.org/faq

WHATWG の FAQ を見る。

- 活動のほとんどは GitHub 上で行われている
  - https://github.com/whatwg
- Apple、Google、Microsoft、および Mozilla が WHATWG のポリシーを監督する運営グループを作った
- 誰かが仕様を提案する、コミュニティ参加者が推進、編集者がつく、運営グループが調整する
  - 必要に応じて運営グループにはブラウザエンジンの開発する組織からメンバーが参加する
- WHATWG 標準に新しい機能を提案する方法
  - https://whatwg.org/faq#adding-new-features
  - GitHub の[WHATWG の html の issue](https://github.com/whatwg/html/issues)に追加したい内容の要件を追加する
  - ブラウザベンダにフィードバックを求める
  - 2 つ以上のブラウザエンジンが機能を実装して shipping するのを約束する
  - 新しい機能のテストを作成する

#### https://github.com/whatwg/html/blob/main/FAQ.md

WHATWG の HTML の FAQ を見る。

#### https://www.tohoho-web.com/html/memo/htmlls.htm

- HTML1.0~HTML2.9 は IETF
- HTML3.2~5.2 は W3C が標準化を進める
- W3C とは別に Appple, Mozilla, Opera の開発者らが設立した WHATWG という団体で策定が進んでいるのが HTML Living Standard ど呼ばれる仕様
  - バージョン番号がなく、日々、改版が進められている
- 現在では W3C による HTML5 の仕様策定は中止され、HTML Living Standard が HTML の標準仕様に
- W3C の方針、Web 開発者/利用者のニーズを取り込むことに消極的など、が不満
- 開発者のニーズを手折り込んだ HTML 開発の再開を提案したが却下
  - これを機に 2004 年 6 月、Apple, Opera, Mozilla の開発者達は WHATWG を設立
- WHATWG は W3C とは別に仕様を策定
- W3C と WHATWG は共同作業してた
- ただ、細部で分裂
  - WHATWG は HTML Living Standard を開始
  - W3C は HTML5 を独自に勧告
- HTML Living Standard と HTML5 で標準仕様が分裂
- MS は W3C ベースだったが、Google、Mozila、Apple、Opera は HTML Living Standard を採用
- 唯一の Edge も Chromium ベースになったことで、HTML Living Standard に完全に統一された
- 2021 年 1 月 28 日、W3C の HTML 関連の仕様はすべて廃止され、WHATWG の HTML Living Standard に完全統一されました。

#### https://blog.jxck.io/entries/2020-11-19/how-to-track-web-standards.html

- 仕様は IETF, WHATWG, TC39 を主とし、実装は Chrome, Firefox と Safari Edge の 4 ブラウザを主とした。

#### https://blog.jxck.io/entries/2018-07-18/how-to-logging-monthly-web.html

- WHATWG/W3C 動向
- W3C のドラフトにはステージがある
- WHATWG では Living Standard なので、ステージを持たない
  - 多くは GitHub で管理されているので、スペックのリンクのリポジトリを追えばいい
- Mailing List には標準化に関する提案、議論、Meeting の議事録などが上がる

#### W3C との関係

- working group のリスト
  - https://www.w3.org/groups/wg/
- HTML Working Group のリスト
  - https://www.w3.org/groups/wg/htmlwg
- CSS の Working Group のリスト
  - https://www.w3.org/groups/wg/css

#### W3C と WHATWG の HTML の関係性

- https://www.mitsue.co.jp/knowledge/blog/frontend/201905/29_1711.html
  - HTML と DOM の仕様策定は共同で行い、管理自体は WHATWG のリポジトリで行われる？
  - これは 2019 年の記事、2021 年には W3C が HTML Review Draft を W3C 勧告として承認することに
    - https://www.w3.org/blog/news/archives/8909
    - HTML Review Draft は Living Standard のスナップショット
  - [詳細の記事](https://www.w3.org/blog/2021/01/whatwg-review-drafts-of-html-and-dom-endorsed-as-w3c-recommendations/)
    - The W3C HTML Working Group was rechartered to assist the W3C community in raising issues and proposing solutions for those specifications.
    - W3C の HTML ワーキンググループは W3C コミュニティが問題を提起し、仕様に関する解決策を提案するのをアシストするために設立
    - HTML ワーキンググループは年に 1 回 HTML と DOM の WHATWG の Review Draft を W3C 勧告に持ち込む予定

W3C 勧告とは国際的に標準化された技術となること
https://wemo.tech/795

メーリングリストで watch しているリポジトリです。

- https://github.com/w3c/html-aam
- https://github.com/w3c/html-aria
- https://github.com/w3c/html-extensions
- https://github.com/w3c/htmlwg
- https://github.com/w3c/webcomponents
- https://github.com/whatwg/html
- https://github.com/whatwg/dom

## CSS

## JavaScript
