---
title: "その仕様はどこから？"
emoji: "📜"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["html", "css", "javascript", "ecmascript"]
published: false
publication_name: "cybozu_frontend"
---

## HTML(DOM)

- WHATWG では 6 ヶ月に一度、Living Standard のスナップショットを Review Draft として残す
  - https://html.spec.whatwg.org/review-drafts/
  - https://dom.spec.whatwg.org/review-drafts/
- W3C の HTML Working Group では年に 1 回、この Review Draft を W3C 勧告に持っていく
  - この過程で W3C のレビューを受けれたり、実装レポートが作られる。また、W3C 勧告までいくと、特許ポリシーのもと、技術を利用できる様になる

## 未整理メモ

- 現在では、WHATWG が策定する HTML Living Standard が HTML の標準仕様になっている
  - https://html.spec.whatwg.org/
- HTML に関しては、Living Standard なので、下記リポジトリのコミットを見ればいい
  - https://github.com/whatwg/html

### https://html.spec.whatwg.org/multipage/introduction.html#introduction

- 1990~1995 年、最初は CERN でホストされ、次に IETF でホストされ、拡張されてきた
- W3C 発足と共に HTML3.2 や HTML4 の策定が続いた
- ちょっと停滞した
- HTML5 の策定作業中に Mozilla と Opera の共同で W3C に提案したけど却下された。W3C は XML ベースの代替品の開発を継続
- WHATWG と呼ばれる組織が誕生、Apple、Mozilla、Opera 共同で継続する意向を発表
- 2006 年に W3C が HTML5 の開発に興味を示して、WHATWG と W3C は協力して、開発してた
- WHATWG は仕様をプラットフォームの進化に合わせて新しい機能を追加、Living Standard としたかった
- W3C は HTML5 の完成バージョンを公開したかった
- 2019 年に WHATWG と W3C は、今後 HTML の単一バージョンに関して共同作業を行う署名をした

### https://whatwg.org/faq

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

### https://github.com/whatwg/html/blob/main/FAQ.md

WHATWG の HTML の FAQ を見る。

- 標準の変更を追うには？
  - twitter の[フィード](https://twitter.com/htmlstandard)
  - GitHub のコミットログ
- [WHATWG と W3C HTML WG について](https://github.com/whatwg/html/blob/main/FAQ.md#whatwg-and-the-w3c-html-wg)
  - それぞれのグループは目標が違うので統合することはない
  - W3C HTML WG は WHATWG の HTML レビュードラフトを W3C 勧告として承認すること
    - [詳細はこちら](https://www.w3.org/2019/04/WHATWG-W3C-MOU.html)
  - WHATWG ではすべての参加者と組織からのフィードバックを受け入れる
  - WHATWG の editor は[WHATWG Working Mode](https://whatwg.org/working-mode)に従って、Livigin Standard にマージするかを決定する
  - W3C HTML WG を含むコミュニティの参加者が editor に対して異議を唱える場合は、[WHATWG Steering Group(運営)とディスカッションするための issue を提起できる](https://whatwg.org/workstream-policy#appeals)
  - W3C HTML WG の場合は、[懸念を表明する仕組み](https://www.w3.org/2019/04/WHATWG-W3C-MOU.html#conflict)があり、最終的に WHATWG HTML 標準の最新の内容を W3C 勧告として承認しない可能性がある

リンク先まとめ。

- [WHATWG の Working Mode について](https://whatwg.org/working-mode)
  - WHATWG で仕様を決めるまでの規範とかが記載されている
- [W3C が懸念を表明する仕組み](https://www.w3.org/2019/04/WHATWG-W3C-MOU.html#conflict)
  - 未解決の意義がある場合に、HTML WG は WHATWG リポジトリにて issue を再度 open する事を依頼する
  - 特定のレビュードラフトを推奨することを W3C は拒否できる
  - 最後の W3C と WHATWG との直接交渉で合意できないと、この覚書(WHATWG と)を修了する

#### [W3C と WHATWG の覚書](https://www.w3.org/2019/04/WHATWG-W3C-MOU.html)

- W3C と WHATWG の覚書
- この覚書では HTML および DOM 仕様の開発のためのコラボレーション プロセスについて説明
- HTML と DOM の仕様は WHATWG Living Standard 仕様プロセスに従って、WHATWG で開発される
- W3C は CR、PR、REC に至る W3C プロセスを通じて、WHATWG レビュードラフトを承認して W3C 勧告にする予定
- W3C 勧告と WHATWG レビュードラフトが同じドキュメントであること
- W3C はメンバーに対して HTML WG を設立。HTML WG 憲章は、W3C コミュニティが問題を提起し、HTML および DOM 仕様に関する解決策を提案するのを支援し、レビュー ドラフトを推奨に持ち込むことを目的
- W3C HTML WG のディスカッションは WHATWG の GitHub リポジトリに反映される必要がある
- レビュードラフトが WHATWG で公開された場合に W3C がやること
  - HTML WG は CR になるよう務める
  - 異議がなければ W3C 勧告になる
  - W3C 勧告されるとレビュードラフトに W3C のロゴを含める
  - 最終的に WHATWG と W3C が合意にいたらなければ、相違点を含む勧告を発表し、WHATWG はこの覚書を終了する

#### [W3C とは](https://www.w3.org/Consortium/)

- Web 標準を開発する国際コミュニティ
- Web の発明者の Tim Berners-Lee さん、CEO の Ralph Swick さん、および取締役会が率いてる
- Web の可能性を最大限に引き出すことが使命

[ミッション](https://www.w3.org/Consortium/mission)

- Web の社会的価値はすべての人々が、すべてを共有する機会を可能にすること
- すべてのデバイスで Web にアクセスできる

#### [W3C の勧告プロセス](https://www.w3.org/2021/Process-20211102/)

- W3C 勧告(W3C Recommendations)は working group によって開発されている

[Dicisions](https://www.w3.org/2021/Process-20211102/#decisions)

- コンセンサス: かなりの数の個人が決定を支持し、誰も正式な意義を登録しない

---

[The W3C Recommendation Track](https://www.w3.org/2021/Process-20211102/#Reports)

W3C の勧告トラックは次の順番で構成されている。

1. First Public Working Draft
2. Working Drafts
3. Candidate Recommendations
4. Proposed Recommendation
5. W3C Recommendation

- Working Draft(WD)
  - W3C がテクニカルレポートページで公開したドキュメント
  - テクニカルレポートの最初は First Public Working Draft と呼ばれる
- Candidate Recommendations(CR)
  - Working Draft が技術的要件と依存関係を満たし、幅広いレビューを受けているドキュメント
  - 最終レビューが行われることを広くコミュニティに知らせる
  - 実装経験を集める
- Proposed Recommendation(PR)
  - W3C 勧告になるのため十分な品質であることを W3C が承認した文書
  - この段階で委員会による正式なレビューが開始
  - 委員会は W3C 勧告として公開するか、更なる作業のためにワーキングループへ戻すかをソウル
- W3C Recommendation(REC)
  - W3C 勧告は広範囲なコンセンサス構築の後、W3C メンバーの承認を受けた、仕様・ガイドライン。
  - この勧告を Web の標準として広く展開することを推奨

---

実装経験とは？

- ステータスを勧める対象の仕様が明確である場合に、各機能が実装されているか、相互運用可能な実装かを Director が考慮する

### [HTML Working Group 憲章](https://www.w3.org/2022/06/html-wg-charter.html)

- HTML: W3C 勧告は年単位で行う
- DOM: W3C 勧告は年単位で行う
  - 2021/06/08 に W3C 勧告された
  - https://dom.spec.whatwg.org/review-drafts/2020-06/

### https://wiki.whatwg.org/

WHATWG の wiki。

- 最近の多くのアクティビティは GitHub になっているので、この wiki ページは歴史的なものになるかも

### [WHATWG のプロセス](https://whatwg.org/faq#process)

https://whatwg.org/faq#patent-policy

- In non-normative layperson's terms: roughly every six months, a Living Standard is “snapshotted” to create a Review Draft suitable for patent review.
- 6 ヶ月ごとに Living Standard を特許に適した Review Draft として作成するためにスナップショットされる
- review draft のディレクトリがある
  - https://github.com/whatwg/html/tree/main/review-drafts
- HTML の Review Drafts
  - https://html.spec.whatwg.org/review-drafts/
- DOM の Review Drafts
  - https://dom.spec.whatwg.org/review-drafts/

### https://www.tohoho-web.com/html/memo/htmlls.htm

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

### https://blog.jxck.io/entries/2020-11-19/how-to-track-web-standards.html

- 仕様は IETF, WHATWG, TC39 を主とし、実装は Chrome, Firefox と Safari Edge の 4 ブラウザを主とした。

### https://blog.jxck.io/entries/2018-07-18/how-to-logging-monthly-web.html

- WHATWG/W3C 動向
- W3C のドラフトにはステージがある
- WHATWG では Living Standard なので、ステージを持たない
  - 多くは GitHub で管理されているので、スペックのリンクのリポジトリを追えばいい
- Mailing List には標準化に関する提案、議論、Meeting の議事録などが上がる

### W3C との関係

- working group のリスト
  - https://www.w3.org/groups/wg/
- HTML Working Group のリスト
  - https://www.w3.org/groups/wg/htmlwg
- CSS の Working Group のリスト
  - https://www.w3.org/groups/wg/css

W3C の役割の説明記事。

- https://parashuto.com/rriver/development/which-html-spec
- https://twitter.com/momdo_/status/1287601041727344640
- CR から次の段階に進むため、実装レポートが作成される、W3C の各ワーキンググループにレビューを依頼できる。W3C 勧告されると特許ポリシーのもと、技術を実装できる

### W3C と WHATWG の HTML の関係性

- https://www.mitsue.co.jp/knowledge/blog/frontend/201905/29_1711.html
  - HTML と DOM の仕様策定は共同で行い、管理自体は WHATWG のリポジトリで行われる？
  - これは 2019 年の記事、2021 年には W3C が HTML Review Draft を W3C 勧告として承認することに
    - https://www.w3.org/blog/news/archives/8909
    - HTML Review Draft は Living Standard のスナップショット
  - [詳細の記事](https://www.w3.org/blog/2021/01/whatwg-review-drafts-of-html-and-dom-endorsed-as-w3c-recommendations/)
    - The W3C HTML Working Group was rechartered to assist the W3C community in raising issues and proposing solutions for those specifications.
    - W3C の HTML ワーキンググループは W3C コミュニティが問題を提起し、仕様に関する解決策を提案するのをアシストするために設立
    - The HTML Working Group expects to bring WHATWG Review Drafts of HTML and DOM to Recommendation once a year.
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

- [HTML の history](https://www.w3.org/standards/history/html)
  - 2021-01-28 に Recommendation
  -

## CSS

https://www.w3.org/Style/CSS/

## JavaScript
