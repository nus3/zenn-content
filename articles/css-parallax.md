---
title: "CSSカスタムプロパティを使ってパララックスを実装する"
emoji: "📸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["css"]
published: true
---

## Motivate

次の記事で紹介されているパララックスの実装が面白かったので日本語でまとめることにしました
https://css-tricks.com/parallax-powered-by-css-custom-properties/

余談ですが、初めて海外の人にTwitterでDMを送りました
この記事を書いてる[Jhey](https://twitter.com/jh3yy)さんに向けて次のようなやり取りをしたんですが
(意訳)

> ワイ: パララックスの記事おもろかったから日本語で記事書いてもええ？
> Jhey: わざわざ聞いてくれてあんがとー! 全然ええで!!
> Jhey: なんか手伝えることあったら言ってやー

Jheyさんめっちゃいい人だった

## Parallax

カーソルを移動させると錨の周りにいる魚たちがそれぞれ違う動きをします

![今回実装したparallaxのgif](/images/css-parallax/parallax.gif)

[実際に触ってみたい方はこちら](https://nus3.github.io/p-storybook/?path=/story/parallax-index--default)
[ソースコード](https://github.com/nus3/p-storybook/tree/main/src/components/Parallax)

## How to Parallax

パララックスを実装する[^1]手順をざっくりとまとめると次になります

[^1]: 今回作るものはHTML・CSS・素のJSのみで実装できますが、好みでReactで実装しています

1. [`pointermove`](https://developer.mozilla.org/ja/docs/Web/API/Document/pointermove_event)イベントからカーソルの座標(x, y)を取得する
2. 対象elementの中心座標を取得する
3. 対象elementの中心からみて、X軸とY軸のカーソルの変化量を計算する
4. 計算した値を[CSSカスタムプロパティ](https://developer.mozilla.org/ja/docs/Web/CSS/--*)に格納する
5. 各要素(魚)がどれくらい動くか(縦、横、回転)を係数として設定する
6. カーソルの変化量と各要素の係数を使って`transform`や`rotate`する

1つずつ詳細を見てきましょう

### [`pointermove`](https://developer.mozilla.org/ja/docs/Web/API/Document/pointermove_event)イベントからカーソルの座標を取得する

カーソルが動いた際のx軸とy軸の値を取得できます

```typescript
window.addEventListener('pointermove', ({x, y}: PointerEvent) => {
  // 座標(x, y)を使った処理をする
})
```

### 対象elementの中心座標を取得する

elementのサイズと位置を返してくれる[`getBoundingClientRect()`](https://developer.mozilla.org/ja/docs/Web/API/Element/getBoundingClientRect)を使い、対象elementの中心を計算します

```typescript
const elementBounds = element.getBoundingClientRect()
const centerX = elementBounds.left + elementBounds.width / 2
const centerY = elementBounds.top + elementBounds.height / 2
```

elementの中心座標を出したい時ってちょくちょくあるので、これは便利でヨキですね。

### 対象elementの中心からみて、X軸とY軸のカーソルの変化量を計算する

[元記事](https://css-tricks.com/parallax-powered-by-css-custom-properties/)では、[GSAP](https://github.com/greensock/GSAP)というライブラリを使ってアニメーションに必要な計算をしています

今回、対象のelementの中心からみて、X軸とY軸でどれくらい変化があったのかを計算するために[`gsap.utils.mapRange()`](https://greensock.com/docs/v3/GSAP/UtilityMethods/mapRange())を使っています


:::details mapRangeを使って中心からの変化量を計算する仕組み

mapRangeについて

- `gsap.utils.mapRange()`ではinputとoutputの範囲をそれぞれ決め、最後の引数で渡した値をその範囲の比率に応じてoutputの範囲の値で返してくれます
- 例えば`gsap.utils.mapRange(0, 100, 0, 1000, 50)`の返り値は500になります
- inputの範囲は`0 < x < 100`、outputの範囲は`0 < x < 1000`なので最後の引数で渡した値は10倍されて返ってきます


次のコードでは、ビューポートの中心からみたxの変化量をcallback関数に渡しています

```typescript
const bounds = 100

// gsap.utils.mapRangeはinputとoutputの最大・最小の比率を出し、valueをその比率で計算する
// mapRange(inMin: number, inMax: number, outMin: number, outMax: number, value: number)
const boundX = gsap.utils.mapRange(
  0,                   // inputの最小値
  window.innerWidth,   // inputの最大値
  -bounds,             // outputの最小値
  bounds,              // outputの最大値
  x,                   // 実際の値(通常はinMinとinMaxの間)
)

callback(boundX / 100)
```

ポイントはinputの範囲が`0 < input < ウインドウ幅(window.innerWidth)`に対して、outputの範囲は`-100 < output < 100`になっている部分
pointermoveイベントで取得できるx座標が中心に来たときに`boundX = 0`になり、カーソルが中心より左いる場合は`x < 0`、右にいる場合は`x > 0`になる
:::


また、[`gsap.utils.clamp()`](https://greensock.com/docs/v3/GSAP/UtilityMethods/clamp())を使って、x,yの変化量が一定の範囲を超えないようにしています
次のコードではxが-60よりも小さければ-60に、60よりも大きければ60になります

```ts
`${Math.floor(gsap.utils.clamp(-60, 60, x))}`
```

### 計算した値をCSSカスタムプロパティ格納する

[CSSカスタムプロパティ](https://developer.mozilla.org/ja/docs/Web/CSS/--*)を使うと独自のプロパティを定義することができます
独自に定義したプロパティの値は[var関数](https://developer.mozilla.org/ja/docs/Web/CSS/var())使って呼び出せます

```css
:root {
  --range-x: 300;
}
.foo {
  left: var(--range-x, 0); /* var関数の第二引数はfallbackの値 */
}
```

次のコードではcallbackで受け取ったx,yの変化量をCSSのカスタムプロパティの`--range-x`, `--range-y`に渡しています

```tsx
const containerRef = useRef<HTMLDivElement>(null)
const callback = (x: number, y: number) => {
  if (!containerRef.current) return

  containerRef.current.style.setProperty(
    '--range-x',
    `${Math.floor(gsap.utils.clamp(-60, 60, x * 100))}`,
  )
  containerRef.current.style.setProperty(
    '--range-y',
    `${Math.floor(gsap.utils.clamp(-60, 60, y * 100))}`,
  )
}
```

### 各要素(魚)がどれくらい動くか(縦、横、回転)を係数として設定する

各要素の係数を次のような型として定義しました
一部`Optional Properties`にしているものは`undefined`の場合は変化しないようにします

```ts
type Item = {
  key: string // ユニークな識別子(同じ画像を使いたい時に区別がつくように)
  name: string // 画像のファイル名
  config: {
    positionX: number    // 魚の初期位置
    positionY: number
    positionZ?: number
    height?: number
    width?: number
    moveX?: number      // x軸の変化量の係数
    moveY?: number      // y軸の変化量の係数
    rotate?: number     // 回転量の係数
  }
}
```

:::details 実際にItem型を使った各要素の定義
```ts
export const ITEMS: Item[] = [
  {
    key: 'anchor',
    name: 'anchor',
    config: {
      positionX: 50,
      positionY: 50,
      positionZ: -1,
      width: 68,
      rotate: 0.06,
    },
  },
  {
    key: 'seahorse',
    name: 'seahorse',
    config: {
      positionX: 74,
      positionY: 15,
      moveX: 1.5,
      moveY: -0.85,
      width: 10,
    },
  },
  // ....必要な要素を実際に動かしつつ、係数を調整するんじゃ！
]
```
https://github.com/nus3/p-storybook/blob/main/src/components/ParallaxItem/item.ts
:::


各要素のコンポーネント側では設定された値をCSSカスタムプロパティに格納する

```tsx
export const ParallaxItem: VFC<ParallaxItemProps> = ({ config, children }) => {
  // ...configから使う値を分割代入する
  return (
    <div
      className={styles.wrap}
      style={
        {
          '--x': positionX,
          '--y': positionY,
          '--z': positionZ,
          '--r': rotate,
          '--rx': rotateX,
          '--ry': rotateY,
          '--mx': moveX,
          '--my': moveY,
          '--height': height,
          '--width': width,
        } as ItemCSS
      }
    >
      {children}
    </div>
  )
}
```

:::details React+TSの場合はカスタムプロパティ用にReact.CSSPropertiesを拡張した型を定義する

```tsx
interface ItemCSS extends React.CSSProperties {
  '--x': number
  '--y': number
  '--z': number
  '--r': number
  '--rx': number
  '--ry': number
  '--mx': number
  '--my': number
  '--height': number
  '--width': number
  '--motion-rate': number
}
```

:::



### カーソルの変化量と各要素の係数を使って`transform`や`rotate`する

各要素に当てるスタイルはCSSカスタムプロパティに格納した初期位置や係数からvar関数を使って定義します

次のCSSではrotateの値が`回転の係数(--r) * カーソルのx軸の変化量(--range-x) * 1deg`で算出される

```css
.wrap {
  transform:
    rotate(
      calc(((var(--r, 0) * var(--range-x, 0)) * var(--motion-rate)) * 1deg)
    );
}
```


:::details 各要素(魚)のスタイル全容

魚の縦・横・z方向の動きをtranslate3dで、回転をrotateで定義している

```css
.wrap {
  position: absolute;
  left: calc(var(--x, 50) * 1%);
  top: calc(var(--y, 50) * 1%);
  height: calc(var(--height, auto) * 1%);
  width: calc(var(--width, auto) * 1%);
  transform: translate(-50%, -50%)
    translate3d(
      calc(((var(--mx, 0) * var(--range-x, 0)) * var(--motion-rate)) * 1%),
      calc(((var(--my, 0) * var(--range-y, 0)) * var(--motion-rate)) * 1%),
      calc(var(--z, 0) * 1vmin)
    )
    rotateX(
      calc(((var(--rx, 0) * var(--range-y, 0)) * var(--motion-rate)) * 1deg)
    )
    rotateY(
      calc(((var(--ry, 0) * var(--range-x, 0)) * var(--motion-rate)) * 1deg)
    )
    rotate(
      calc(((var(--r, 0) * var(--range-x, 0)) * var(--motion-rate)) * 1deg)
    );
  transform-style: preserve-3d; /* 子要素を3D空間に配置できるように */
}
```
:::

### この他にも

元記事では良さげなテクニックが色々書かれていました

- [CSSでのトレース方法](https://css-tricks.com/advice-for-complex-css-illustrations/#tracing-is-perfectly-acceptable)
- 多くの画像を読み込まずに`background-position`や`background-size`を使って、一枚の画像から必要なパーツを切り出す
- [`prefers-reduced-motion`](https://developer.mozilla.org/ja/docs/Web/CSS/@media/prefers-reduced-motion)がreduce時に対応するために`--motion-rate`いれたよ

CSSでトレースするの面白そう

## Finally

実は、この記事はサイボウズアドベントカレンダーの5日目[^2]のものでした
25日間、サイボウズの人たちがいろんなこと書いているので気になった方はゼヒ見てみてください〜

https://adventar.org/calendars/6823

[^2]: アドベントカレンダーやるぞーと思って、今回のパララックスの実装ができたのが昨日。そこから記事を書きはじめたからギリギリチョップだった・・

## Special thanks

Jhey 👏
https://twitter.com/jh3yy

Icon Pack: Ocean | Lineal color 🐠
https://www.flaticon.com/packs/ocean-23