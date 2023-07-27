---
title: "Astro 2.9でView Transitions APIがお手軽に"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["astro"]
published: true
publication_name: "cybozu_frontend"
---

## 3 行まとめ

- Astro 2.9 で View Transitions が experimental で実装された
- SPA モードであれば、View Transitions API を使った画面遷移時のアニメーションが簡単に使える
- View Transitions API が未サポートのブラウザのためのフォールバックも用意されている

## Astro 2.9 で実装された View Transitions

https://astro.build/blog/astro-290/

https://docs.astro.build/en/guides/view-transitions/

Astro 2.9 がリリースされ、その中には `<ViewTransitions />` コンポーネントが experimental で実装されています。

SPA モードの際に、この `<ViewTransitions />` コンポーネントを `<head>` やレイアウトのような画面共通のコンポーネントに追加することで、[View Transitions API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transitions_API)を使って画面遷移時にアニメーションが実行されるようになります。

画面遷移時のアニメーションは[いくつかビルドインのもの](https://docs.astro.build/en/guides/view-transitions/#built-in-animation-directives)が用意されていますし、[カスタマイズ](https://docs.astro.build/en/guides/view-transitions/#customizing-animations)も可能です。

また、[View Transitions API がサポートされていないブラウザ](https://developer.mozilla.org/en-US/docs/Web/API/View_Transitions_API#browser_compatibility)のための[フォールバックのオプション](https://docs.astro.build/en/guides/view-transitions/#fallback-control)も用意されています。View Transitions API がサポートされていないブラウザの対応を Astro がいい感じにコントロールしてくれるのは良きですね。

## `<ViewTransitions />` コンポーネントを使ってみよう

実際に `<ViewTransitions />` コンポーネントを使ってみましょう。

![View Transitions を使ったデモ](/images/astro-view-transitions/demo.gif =400x)

今回実装したデモは次のリンクから確認できます。

https://nus3.github.io/astro-labs/

また、[nus3/astro-labs](https://github.com/nus3/astro-labs)でコードも確認できます。

### View Transitions を有効にする

`astro.config.mjs` で View Transitions 用のフラグを有効にすることで、`<ViewTransitions />` コンポーネントが使えるようになります。

```javascript: astro.config.mjs
import { defineConfig } from 'astro/config';

export default defineConfig({
  experimental: {
   viewTransitions: true
  }
});
```

### `<ViewTransitions />` コンポーネントを共通コンポーネントで呼び出す

今回は、`<main>` の中身以外の部分を `Layout.astro` コンポーネントとして定義し、その `<head>` の中で `<ViewTransitions />` コンポーネントを呼び出します。

```html: Layout.astro
---
import { ViewTransitions } from "astro:transitions";
---

<html lang="en">
  <head>
    <!-- ... -->
    <ViewTransitions />
  </head>
  <body>
    <header>
      <!-- ... -->
    </header>
    <main>
      <slot />
    </main>
  </body>
</html>
```

基本的には、この `<ViewTransitions />` を追加するだけで、Astro 側で遷移前と遷移後の画面の両方で見つかった要素に一意の `view-transition-name` が割り当てられ、アニメーションが実行されます。遷移前と遷移後で要素が同一かどうかの判定は、[要素の種類と DOM 構造から推測されるよう](https://docs.astro.build/en/guides/view-transitions/#naming-a-transition)です。

### ビルトインのアニメーションを利用する

画面遷移時のアニメーションとして、Astro ではビルトインで `morph`、`slide`、`fade` が用意されています。

デフォルトでは `morph` が指定されており、DOM 構造など遷移前後でのページの差分に応じて、Astro が用意したアニメーションが実行されるようになっています。

`transition:animate` ディレクティブで指定することで、デフォルト以外のアニメーションを指定できます。

今回は、`<main>` に対して、画面遷移時には `slide` のアニメーションが実行されるようにしましょう。

```html: Layout.astro
<!-- ... -->

<html lang="en">
  <!-- ... -->
  <main transition:animate="slide">
    <slot />
  </main>
</html>
```

### 対象の要素を指定する

前述しましたが、 `<ViewTransitions />` を追加すると、Astro が遷移前後で要素が同一かどうか判定してくれます。この判定を独自で行いたい場合は `transition:name` ディレクティブを使います。

今回はそれぞれの画面で 4 つの要素の中で、1 つだけ枠をつけた要素に対して `transition:name` ディレクティブを追加しています。

![デモ画面](/images/astro-view-transitions/top-page.png =400x)

```html: index.astro
---
import Layout from "../components/Layout.astro";
---

<Layout>
  <div class="wrapper">
    <div class="section target" transition:name="target">nus1 Content</div>
    <div class="section">nus1 Content</div>
    <div class="section">nus1 Content</div>
    <div class="section">nus1 Content</div>
  </div>
</Layout>
```

```html: nus2.astro
---
import Layout from "../components/Layout.astro";
---

<Layout>
  <div class="wrapper">
    <div class="section">nus2 Content</div>
    <div class="section">nus2 Content</div>
    <div class="section">nus2 Content</div>
    <div class="section target" transition:name="target">nus2 Content</div>
  </div>
</Layout>
```

```html: nus3.astro
---
import Layout from "../components/Layout.astro";
---

<Layout>
  <div class="wrapper">
    <div class="section">nus3 Content</div>
    <div class="section target" transition:name="target">nus3 Content</div>
    <div class="section">nus3 Content</div>
    <div class="section">nus3 Content</div>
  </div>
</Layout>
```

Astro では遷移前後の画面で `transition:name="target"` が指定された要素を同一の要素としてアニメーションが実行されます。

![transition:name を使った要素を同一の要素として扱っている様子](/images/astro-view-transitions/demo.gif =400x)

## View Transitions のフォールバックはどのようにして行われているのか

`<ViewTransitions />` を使った場合のフォールバックの挙動は `fallback` 属性で指定できます。指定できる値には `animate`、`swap`、`none` があります。

デフォルトで指定されているのが `animate` になっており、この値が指定されている場合は、View Transitions API がサポートされていないブラウザの場合でも、画面遷移時にアニメーションが実行されます。

今回作ったデモでは `animate` を指定しているので View Transitions API がサポートされていない Firefox でも、画面遷移時にアニメーションが実行されます。実際に Firefox で[デモ](https://nus3.github.io/astro-labs/)を開いてもらえると確認できるのでよければ試してみてください。

View Transitions API がサポートされている Chrome とサポートされていない Firefox で適用される CSS をみてみましょう。

```css
/* Chrome */
[data-astro-transition-scope="astro-4STNBBPF-2"] {
  view-transition-name: astro-4STNBBPF-2;
}

/* Firefox */
[data-astro-transition-fallback="new"]
  [data-astro-transition-scope="astro-4STNBBPF-2"] {
  animation-duration: 210ms, 300ms;
  animation-timing-function: cubic-bezier(0, 0, 0.2, 1), cubic-bezier(0.4, 0, 0.2, 1);
  animation-delay: 90ms;
  animation-fill-mode: both, both;
  animation-name: astroFadeIn, astroSlideFromRight;
}
```

これらのセレクタは実装を見ていると、`<style>` の中にどちらも定義されていそうです。

https://github.com/withastro/astro/blob/d5f526b3397cf24aa06353de2de91b2ba08cd4eb/packages/astro/src/runtime/server/transition.ts#L24-L96

![style 要素にセレクタが定義されている](/images/astro-view-transitions/style-element.png)
_デモ環境の `<style>`_

また、`<ViewTransitions>` の実装を見ると、View Transitions API がサポートされておらず、`fallback` の値が `animate` の場合に `data-astro-transition-fallback` などの data 属性を付与してアニメーションを実行しています。

https://github.com/withastro/astro/blob/d5f526b3397cf24aa06353de2de91b2ba08cd4eb/packages/astro/components/ViewTransitions.astro#L65-L121

実際に画面遷移後では Firefox と Chrome で追加されている data 属性が異なっています。

```html
<!-- Firefox -->
<html
  class="astro-DMQSI53G"
  data-astro-transition="forward"
  data-astro-transition-fallback="new"
></html>

<!-- Chrome -->
<html class="astro-DMQSI53G"></html>
```

[Astro のドキュメントにも記載](https://docs.astro.build/en/guides/view-transitions/#fallback-control)されていますが、`fallback` 時のアニメーションは data 属性を使ってハンドリングしているようです。ただ、今回紹介した `transition:name` やビルトインアニメーションの `morph` のような View Transitions API 特有の機能を使うものは `fallback` 時に再現できないので注意が必要です。

## あとがき

View Transitions API はまだ[Working Draft な仕様](https://drafts.csswg.org/css-view-transitions/#dom-document-startviewtransition)で、ブラウザへの実装状況も揃っているわけではありませんが、そういった部分を Astro がサポートしてくれることで、[Progressive Enhancement](https://developer.mozilla.org/ja/docs/Glossary/Progressive_Enhancement) に View Transitions API が採用できそうですね。
