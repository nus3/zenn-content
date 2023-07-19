---
title: "スクロールに合わせてアニメーションが進められるScroll-driven Animationsについて"
emoji: "🎰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["css"]
published: true
publication_name: "cybozu_frontend"
---

## 3 行まとめ

- Chrome115 のリリースで、Scroll-driven Animations が実装された
- Scroll-driven Animations ではスクロールに合わせてアニメーションを進行できる
- シンプルな記述でアニメーションとスクロールを関連づけられる

## Scroll-driven Animations とは

https://developer.chrome.com/blog/scroll-animation-performance-case-study/

Scroll-driven Animations は Chrome115 で実装された、スクロール量に合わせてアニメーションが進められる API です。この新しい API では `scroll` イベントを使わずに CSS と JavaScript でアニメーションとスクロールを関連づけることができるようになりました。

上記の記事で紹介されていますが、`scroll` イベントを使ったアニメーションの実装と比べて、メインスレッドで重い処理を実行していたとしても、スクロールに合わせたアニメーションがスムーズに実行できます。

## Scroll-driven Animations を使ってみよう

:::message
2023/07/19 現在、[Scroll-driven Animations](https://www.w3.org/TR/scroll-animations-1) は、Working Draft の API です。
:::

実際に Scroll-driven Animations を使って、次の Gif のようにスクロールに合わせて進捗バーが進むアニメーションを実装してみましょう。

![Scroll-driven Animations を使ったデモ](/images/scroll-driven-animations/scroll-driven.gif =400x)

今回実装したデモはバージョンが 115 以上の Chrome で次のリンクから確認できます。

https://nus3.github.io/ui-labs/scroll-driven/

また、[nus3/ui-labs](https://github.com/nus3/ui-labs/tree/main/src/scroll-driven)でコードも確認できます。

Scroll-driven Animations は JavaScript を使っても実装できますが、今回はシンプルに CSS で実装してみましょう。

### `animation-timeline` プロパティと `scroll()` 記法

Scroll-driven Animations を使って CSS で、スクロールに合わせたアニメーションを実装するには、`animation-timeline` プロパティに `scroll()` 記法を指定することで実現できます。

```css
@keyframes grow-progress {
  0% {
    width: 0;
  }
  100% {
    width: 100%;
  }
}

.progress-bar {
  animation: grow-progress auto linear forwards;
  animation-timeline: scroll(root y);
}
```

`scroll()` 記法の引数には、どの要素からみたスクロール量なのかを指定する `scroller` と、X 軸・Y 軸どちらのスクロールなのかを指定する `axis` があります。現時点の仕様を見る限り[指定の順番はどちらでも良さそう](https://www.w3.org/TR/scroll-animations-1/#scroll-notation)です。

上記のサンプルコード(`scroll(root y)`)の場合、root 要素での Y 軸のスクロール量に合わせてアニメーションが進行します。

`scroller` で指定できる要素には `root` の他に、`self` と `nearest` があり、それぞれ、`self` では自身の要素、`nearest` では親要素が指定できます。

また、`axis` では `x` と `y` の他に、コンテンツが縦書きなのか横書きなのかで X 軸と Y 軸の解釈が変わる `block` と `inline` があります。

## 参考リンク

- [Scroll-driven Animations の仕様](https://www.w3.org/TR/scroll-animations-1)
- [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-timeline/scroll)

## あとがき

今回紹介したサンプルコードを見てもらうと分かる通り、Scroll-driven Animations を使うと、とてもシンプルな記述でスクロールとアニメーションを関連づけることができるようになって良いですね。
