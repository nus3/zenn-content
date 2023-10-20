---
title: "<select>を自由にカスタマイズできる<selectlist>について"
emoji: "📜"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["html"]
published: true
publication_name: "cybozu_frontend"
---

:::message
2023/10/19 時点では`<selectlist>`は実験的な要素で、この記事で紹介した仕様が変わる可能性があります。
:::

## 3 行まとめ

- Chrome で`<selectlist>`という新しい要素のプロトタイプが実装されている
- `<selectlist>`は自由にカスタマイズできる`<select>`
- 基本的なキーボード操作も組み込まれている

## Chrome でプロトタイプが実装されている`<selectlist>`

次のポスト(旧ツイート)では、Chrome Canary で`<selectlist>`という新しい要素のプロトタイプをテストしていることが紹介されています。

https://x.com/Una/status/1706777335762997283?s=20

この`<selectlist>`は[chrome://flags/](chrome://flags/)から Experimental Web Platform features を Enabled にすることで実際に手元の Chrome から確認できます。(Chrome Canary だけでなく、Chrome118 でも動作することを確認)

![Experimental Web Platform featuresフラグ](/images/selectlist-element/flag.png =400x)

## `<selectlist>`とは

`<selectlist>`は現在 [Open UI](https://open-ui.org/) によって提案され、Chrome によって[プロトタイプの実装](https://chromestatus.com/feature/5737365999976448)が進められています。

https://open-ui.org/components/selectlist/

### デモ

次のリンクに`<selectlist>`のデモを用意したので、Chrome のフラグを有効にして実際に触ってみてください。

https://nus3.github.io/ui-labs/selectlist/

![selectlist要素のデモ](/images/selectlist-element/demo.png)

`<selectlist>`を使うことで見た目を自由にカスタマイズした上で、`<select>`のようにボタンをクリックすることで複数の`<option>`を表示し、その`<option>`を選択できます。

## `<selectlist>`を使ってみよう

### マークアップ

`<selectlist>`では次の要素を使うことで、自分たちで自由にカスタマイズができます。

- `<selectlist>`: リストを表示するためのボタンと listbox を含むルートの要素
- `<button type="selectlist">`: リストを表示するためのボタン
- `<listbox>`: `<option>`を含むラッパー要素
- `<selectedoption>`: 現在選択されている `<option>` のテキストを表示する要素。

これらの要素を使ってデモでは次のようにマークアップしました。

```html
<selectlist>
  <button type="selectlist">
    Current Value:
    <selectedoption></selectedoption>
  </button>
  <listbox>
    <!-- デフォルト値 -->
    <option value="" hidden>undefined</option>
    <div>
      <h3>nus</h3>
      <div class="options">
        <option>nus1</option>
        <option>nus2</option>
        <option>nus3</option>
        <option>nus4</option>
      </div>
      <!-- ... -->
    </div>
  </listbox>
</selectlist>
```

`<button type="selectlist">`を指定することで、クリックされた際にブラウザにリストボックスを開く必要があることを伝えます。

### スタイル

`<selectlist>`に内包される`<button>`や`<listbox>`、`<selectedoption>`にはそれぞれ`::button`、`::listbox`、`::selectedoption`といった疑似要素が用意されています。

また、`<selectlist>`に対して`:open`、`:closed`といった疑似クラスを使うと`<listbox>`が表示、非表示の時のスタイルを指定できます。

デモでは、`<listbox>`が表示されている時に`<option>`が grid に並ぶように`:open`疑似クラスを使っています。

```css
.selectlist {
  &:open {
    .listbox {
      .options {
        display: grid;
        grid-template-columns: 1fr 1fr;
        gap: 4px;
      }
    }
  }
}
```

### キーボード操作

2023/10/19 時点の Chrome では、`<selectlist>`にはデフォルトで次のようなキーボード操作も組み込まれています。

- `<selectlist>`内の`<button>`にフォーカスがある状態で Space、Enter キーを押すと`<listbox>`が表示される
- `<listbox>`が表示されている状態で Esc キーを押すと`<listbox>`が非表示になる
- `<listbox>`が表示されている状態で上下矢印キーを押すと`<option>`のフォーカスが移動し、フォーカスされた値が選択される

### フォーム

`<selectlist>`は`<select>`と同様にフォームに含めることができます。

`selectlist.value`で選択されている`<option>`の `value` を返したり、`disabled`や、`name`、`required`などの属性が使えます。

ただ、`<select>` とは異なり、1 つの `<option>` しか選択できません。

## あとがき

これまで独自の UI を提供しようと思うと実装が大変だったものが、`<selectlist>`を使うことでキーボード操作などアクセシビリティを担保した上で簡単にカスタマイズできるようになるのはとても良いですね。

主要なブラウザで使えるようになるのはもう少し先かもしれませんが、`<selectlist>`の仕様策定とブラウザへの実装が楽しみです。
