---
title: "2022年にWebブラウザに実装された気になる機能を実装例とともに紹介"
emoji: "🌊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["html", "css", "javascript"]
published: true
publication_name: "cybozu_frontend"
---

気付けば、もう年末ということで、2022 年に主要な Web ブラウザで実装された機能から個人的に気になったものをいくつか紹介します。

本記事は、11 月 26 日に行われた JSConf.jp のサイボウズのセッションの中で紹介した内容の詳細版になります。このセッションでは、フロントエンドエキスパートチームのメンバーが 2022 年の印象に残ったフロントエンドトピックについて話していますので、興味のある方は是非ご覧ください！

https://www.youtube.com/live/ccysINf4qTE

(4:51:50 あたりから始まります)

## 主要ブラウザ全てで実装された機能

### ダイアログ要素 `<dialog>`

| Chrome | Edge | Firefox | Safari |
| ------ | ---- | ------- | ------ |
| 37     | 79   | 98      | 15.4   |

3 月にリリースされた[Firefox98](https://developer.mozilla.org/en-US/docs/Mozilla/Firefox/Releases/98)と [Safari15.4](https://developer.apple.com/documentation/safari-release-notes/safari-15_4-release-notes)で実装されたことで、主要ブラウザ(Chrome, Edge, Firefox, Safari)で `<dialog>` が利用できるようになりました。

`<dialog>` が利用できるようになり、これまでよりも簡単にモーダルなダイアログが実装できるようになりました。[`HTMLDialogElement.showModal()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/showModal)でダイアログ以外の要素にフォーカスが当たらないモーダルなダイアログを呼び出せます。

```javascript
const dialog = document.getElementById("dialog");
const showBtn = document.getElementById("show");

showBtn.addEventListener("click", () => {
  dialog.showModal();
});
```

また、[`:modal`](https://developer.mozilla.org/en-US/docs/Web/CSS/:modal)や[`::backdrop`](https://developer.mozilla.org/en-US/docs/Web/CSS/::backdrop)を使うことで、スタイルも簡単に実装できます。

```css
/* オーバレイ部分のスタイル */
.dialog::backdrop {
  background: rgba(0, 0, 0, 0.25);
}
/* モーダルなダイアログ部分のスタイル */
.dialog:modal {
  background-color: #e7f6f2;
}
```

@[codepen](https://codepen.io/yota-hada/pen/vYroKre)

### マウスとキーボードでのフォーカスのスタイルを分けて定義できる `:focus-visible`

| Chrome | Edge | Firefox | Safari |
| ------ | ---- | ------- | ------ |
| 86     | 86   | 85      | 15.4   |

3 月にリリースされた[Safari15.4](https://developer.apple.com/documentation/safari-release-notes/safari-15_4-release-notes)で実装されたことで、主要ブラウザで利用できるようになりました。

下記の CodePen に `:focus` と `:focus-visible` を指定したボタンを用意したので、それぞれマウス・キーボードで操作した場合のフォーカスのスタイルの違いを見てみてください。

@[codepen](https://codepen.io/yota-hada/pen/BaVXzeG)

それぞれのボタンのラベルに指定したスタイルを記載しています。

`only :focus` ボタンではマウス操作(クリック)とキーボード操作の両方でアウトラインが表示されるのに対し、`only :focus-visible` ボタンでは、キーボード操作でのみアウトラインが表示されます。

`:focus` と `:focus-visible` の両方を指定した `:focus and :focus-visible` ボタンではマウス操作時とキーボード操作時でアウトラインの色が変わっていることが確認できます。

主要なブラウザ全てに実装されたのは今年ですが、[State of CSS 2022](https://2022.stateofcss.com/en-US/features/accessibility/)の回答者の割合を見ても、すでに利用している人も多そうです。

![State of CSS 2022で公開された:focus-visibleの利用率](/images/2022-pickup-new-features/focus-visible.png)

### 重なり合った要素の背後の領域に、ぼかしが入れられる `backdrop-filter`

| Chrome | Edge | Firefox | Safari |
| ------ | ---- | ------- | ------ |
| 76     | 17   | 103     | 9      |

7 月にリリースされた [Firefox103](https://developer.mozilla.org/en-US/docs/Mozilla/Firefox/Releases/103)で実装されたことで、主要ブラウザで利用できるようになりました。

`backdrop-filter` プロパティを指定することで背後の領域に対してスタイルを変化できます。どのような変化があるかの一例を、下記の CodePen にまとめました。

@[codepen](https://codepen.io/yota-hada/pen/jOKgrRj)

特に `blur()` による背後の領域のぼかしは簡単に UI の幅を広げられそうですね。

## もう少しで主要ブラウザ全てで実装されそうな機能

### 特定の要素のサイズに合わせてレスポンシブなスタイルが指定できる Container Queries

| Chrome | Edge | Firefox | Safari |
| ------ | ---- | ------- | ------ |
| 105    | 105  | Nightly | 16     |

8 月にリリースされた [Chrome105](https://developer.chrome.com/blog/new-in-chrome-105/)と 9 月にリリースされた[Safari16](https://developer.apple.com/documentation/safari-release-notes/safari-16-release-notes)で Container Queries が実装されました。Firefox でも実装が進められており、現在、[Nightly で利用が可能](https://twitter.com/FirefoxDevTools/status/1595076200199749633?ref_src=twsrc%5Etfw%7Ctwcamp%5Etweetembed&ref_url=notion%3A%2F%2Fwww.notion.so%2F2022-11-15-12-12-2d6f965cc74745b9b74a4adeacbb6d7b)です。

`@container` ルールを利用することで、指定した要素の状態(幅や高さなど)に応じたスタイルを定義できるようになります。

```css
.container {
  container-type: inline-size;
}

/* .containerが指定された要素の幅に合わせてスタイルを変更する */
@container (max-width: 800px) {
  .card {
    display: block;
  }
}
```

CodePen のサンプルでは、`.container` を指定した要素の幅に応じて子要素の `display` を `flex` から `block` に変更しています。

![親要素の幅に合わせて、レスポンシブなスタイルを定義できる](/images/2022-pickup-new-features/container-queries.gif)

Container Queries は要素に対してレスポンシブにスタイルを指定できるので、コンポーネントベースの開発とも相性が良さそうです。Firefox の実装も進んでいるので、来年が楽しみです。

@[codepen](https://codepen.io/yota-hada/pen/MWXNePg)

### セレクターで指定した条件に当てはまった場合にスタイルを指定できる `:has()`

| Chrome | Edge | Firefox | Safari |
| ------ | ---- | ------- | ------ |
| 105    | 105  | 🙅      | 15.4   |

3 月にリリースされた[Safari15.4](https://developer.apple.com/documentation/safari-release-notes/safari-15_4-release-notes) と 8 月にリリースされた [Chrome105](https://developer.chrome.com/blog/new-in-chrome-105/)で実装されました。Firefox では 103 からフラグ付きで試すことができます。

`:has()` の引数にセレクターを渡すことで、その条件に当てはまった場合のスタイルを指定できます。

```html
<style>
  /* .card-hasを指定した要素の子要素にpicture要素がある場合のh2要素のスタイル */
  .card-has:has(> picture) h2 {
    font-size: 1rem;
  }
</style>

<div class="card-has">
  <!-- 子要素に<picture>がある -->
  <picture>
    <img />
  </picture>
  <h2>nus3 blog</h2>
</div>
<div class="card-has">
  <h2>nus3 blog</h2>
</div>
```

@[codepen](https://codepen.io/yota-hada/pen/oNyKLQW)

`:has()` に関しても[Firefox のロードマップ](https://bugzilla.mozilla.org/show_bug.cgi?id=418039)にはありそうなので、実装されるのが待ち遠しいですね！
