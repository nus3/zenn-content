---
title: "CSSだけでtooltipの位置を指定できるCSS Anchor Positioningについて"
emoji: "🪧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["css"]
published: true
publication_name: "cybozu_frontend"
---

## 3 行まとめ

- 特定の要素(アンカー)を基準にして、その要素から位置を指定できる CSS Anchor Positioning
- JavaScript を使わなくても tooltip などの実装ができるように
- Chrome Canary で試せる

## CSS Anchor Positioning とは

https://developer.chrome.com/blog/tether-elements-to-each-other-with-css-anchor-positioning/

CSS Anchor Positioning は特定の要素(アンカー)を基準にして、その要素から別の要素の位置を指定できる API です。

[Chrome114 で実装された Popover API](https://developer.chrome.com/blog/new-in-chrome-114)と CSS Anchor Positioning を使うと、[Floating UI](https://floating-ui.com/)のような tooltip が JavaScript を使用せずに実装できます。

## CSS Anchor Positioning を使って tooltip を実装してみよう

:::message
2023/05/31 現在、[CSS Anchor Positioning](https://drafts.csswg.org/css-anchor-position-1/) は、Editor's Draft の実験的な機能です。
:::

実際に Popover API と CSS Anchor Positioning を使って tooltip を実装してみましょう。

[Chrome Canary](https://www.google.com/intl/ja/chrome/canary/) で `Experimental web platform feature` フラグを enable にすることで、次のリンクから実際にデモを確認できます。

https://nus3.github.io/ui-labs/anchor-position/

また、[nus3/ui-labs](https://github.com/nus3/ui-labs/tree/main/src/anchor-position)でコードも確認できます。

### 基準となる要素(アンカー)を定義する

基準となる要素を定義するためには、CSS で `anchor-name` プロパティを使うか、HTML で `anchor` 属性を使うかの 2 種類があります。

CSS の `anchor-name` プロパティを使ってアンカー要素を定義する場合は次のようになります。

```html
<button class="anchor">Anchor</button>
<style>
  .anchor {
    anchor-name: --anchor;
  }
</style>
```

`anchor-name` プロパティの値には CSS カスタムプロパティで使うような [`<dashed-ident>`](https://developer.mozilla.org/en-US/docs/Web/CSS/dashed-ident) を指定します。

HTML の `anchor` 属性を使う場合は次のようになります。

```html
<button id="anchor">Anchor</button>
<div anchor="anchor">Tooltip</div>
```

アンカー要素に `id` 属性を追加し、その値を tooltip に該当する要素の `anchor` 属性に渡します。

今回は Chrome Canary で動作確認ができる次の 2 つの条件を前提に実装します。

- tooltip に Popover API を使う
- アンカー要素の定義に HTML の `anchor` 属性を使う

### アンカー要素を基準にした位置を指定する

CSS の `anchor` 関数を使って、アンカー要素を基準にした位置を指定します。

```html
<button id="anchor" popovertarget="tooltip">Anchor</button>
<div id="tooltip" class="tooltip" popover anchor="anchor">Tooltip</div>

<style>
  .tooltip {
    top: anchor(bottom);
    left: anchor(center);
    translate: -50% 0;
  }
</style>
```

HTML の `anchor` 属性を使ってアンカー要素を定義していた場合、CSS の `anchor` 関数の引数にはアンカー要素から見て、どこに要素(tooltip)を配置したいのかを指定できます。

今回の場合、`top: anchor(bottom);` では tooltip の `top` はアンカー要素の `bottom` に位置するように指示しています。`left: anchor(center);` では、tooltip の `left` はアンカー要素の `center` にくるよう指示しています。

このままだと、次のスクショのように tooltip の `left` がちょうどアンカー要素の中央の位置に配置されてしまいます。

![tooltipがアンカー要素の中央から右に配置される](/images/css-anchor-position/tooltip2.png =400x)

`translate: -50% 0;` を指定して、アンカー要素の中央に tooltip を配置します。

![tooltipがアンカー要素の下に配置される](/images/css-anchor-position/tooltip1.png =400x)

### フォールバックを定義して、tooltip が Viewport 内に留まるようにする

`@position-fallback` と `position-fallback` を使うことで、`anchor` 関数によって指定された位置が Viewport 外だった場合に別の位置を指定できます。

```css
@position-fallback --bottom-to-top {
  /* アンカー要素の下にtooltipを表示する */
  @try {
    top: anchor(bottom);
    left: anchor(center);
  }
  /*
  アンカー要素の下にtooltipを表示する領域がViewport上にない場合は、
  アンカー要素の上にtooltipを表示する
  */
  @try {
    bottom: anchor(top);
    left: anchor(center);
  }
}

.tooltip {
  position-fallback: --bottom-to-top;
  translate: -50% 0;
}
```

`@position-fallback` に `<dashed-ident>` を指定し、その中の `@try` でアンカー要素から見て、tooltip をどの位置に配置したいのかを定義します。そして、 `position-fallback` プロパティに指定した `<dashed-ident>` を渡すことで、`@try` で指定した位置の中で、Viewport 内に表示される配置が適用されます。

次の gif ではそれぞれ Viewport の幅と高さによって tooltip の表示位置が変わることがわかります。

![Viewportの高さによってtooltipの表示位置が変わる](/images/css-anchor-position/positioning1.gif =400x)
_Viewport の高さによって tooltip の表示位置が変わる_

![Viewportの幅によってtooltipの表示位置が変わる](/images/css-anchor-position/positioning2.gif =400x)
_Viewport の幅によって tooltip の表示位置が変わる_

## 最後に

`<dialog>` もそうですが、今回紹介した CSS Anchor Positioning と Popover API のように、よく使うコンポーネントが JavaScript を使わず、容易に実装できるようになるのはとても良いですね。

まだ実験的な段階ですが、今後、各ブラウザで実装されるのが楽しみな機能です。

### 関連リンク

https://drafts.csswg.org/css-anchor-position-1/
https://developer.chrome.com/blog/whats-new-css-ui-2023/#anchor-positioning
https://kizu.dev/anchor-positioning-experiments/
