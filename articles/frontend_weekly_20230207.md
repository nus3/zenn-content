---
title: "Webアプリの主流がCSRからSSRに？など : Cybozu Frontend Weekly (2023-02-07号)"
emoji: "🎸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CybozuFrontendWeekly", "frontend"]
published: false
publication_name: "cybozu_frontend"
---

こんにちは！サイボウズ株式会社フロントエンドエキスパートチームの [nus3](https://twitter.com/nus3_) です。

# はじめに

フロントエンドエキスパートチームでは毎週火曜日に Frontend Weekly という「1 週間の間にあったフロントエンドニュースを共有する会」を社内で開催しています。

今回は、2023/02/07 の FrontendWeekly で取り上げた記事や話題を紹介します。

# 取り上げた記事・話題

## Typewind

https://typewind.vercel.app/

Tailwind CSS のユーティリティクラスを type-safe に扱えるライブラリです。Zero Bundle Size や `tailwind.config.js` をベースに型を生成する点、エディタの拡張が必要ない点、また、Next.js や SolidStart、Vite などのフレームワークをサポートしてる点が挙げられています。

Tailwind CSS のユーティリティクラスに対して静的解析したい場合に、導入を検討できそうです。

## Unlock New Possibilities with Hybrid Rendering

https://astro.build/blog/hybrid-rendering/

Astro v2 で新しく追加された Hybrid Rendering の記事です。

この Hybrid Rendering を利用することで画面ごとに SSG か SSR かを選択できるようになりました。この記事では、画面ごとのレンダリング方法の判別に、ビルド時の静的解析を用いたり、 `export const prerender = true` のように明示的な指定をできることが紹介されています。また、Hybrid Rendering のユースケースもいくつか紹介されているので、どの様な場合に利用できるか参考になりそうです。

## If you use React, you should be using a React framework

https://twitter.com/acdlite/status/1617611126514266112?s=46&t=mjRBYqR2Pn6e94alL2aiRQ

React コアチームの[Andrew Clark](https://twitter.com/acdlite)が React を利用する場合はフレームワークを採用した方が良いとツイートした話です。

Frontend Weekly の中では、React では特に v18 で追加された `useSyncExternalStore` や `useTransition` のようにフレームワークで利用するような API 設計に変わってきており、また、ルーティングやチャンク分割などパフォーマンスのことを考慮するとフレームワークを経由して React を使うのは自然な流れになりそうと話していました。

合わせて、[Create React App の今後について](https://github.com/reactjs/reactjs.org/pull/5487#issuecomment-1409720741)も話題に上がりました。

## The Future (and the Past) of the Web is Server Side Rendering

https://deno.com/blog/the-future-and-past-is-server-side-rendering

今後、Web アプリケーションは Client Side Rendering(CSR)から Server Side Rendering(SSR)が主流になるのでは、という内容の記事が Deno から公開されました。

Deno では、SSR でのハイドレーションには Islands Architecture を採用することが気に入ってるそうです。(実際に Deno の Web フレームワークである Fresh では Islands Architecture が採用されている)。

また、この記事に関連して次のような話題で盛り上がっていました。

- 昔と比べて Edge や Vercel などのホスティングサービスの登場によって SSR がやりやすくなった
- フレームワークとホスティングの関係性も変わってきている
- [Cloudflare の記事](https://blog.cloudflare.com/ja-jp/welcome-to-the-supercloud-and-developer-week-2022-ja-jp/)で Supercloud って言葉が紹介されていた

## Next.js に追加予定の機能

### `<Link>` を利用する際に Next.js のファイルルーティングに合わせた型が利用できるように

https://twitter.com/shuding_/status/1620137501192253440

### SEO で利用する様な metadata を app ディレクトリの layout でオブジェクトとして定義できるように

https://twitter.com/leeerob/status/1619743437577912321

## Netlify Acquires Gatsby Inc. to Accelerate Adoption of Composable Web Architectures

https://www.netlify.com/press/netlify-acquires-gatsby-inc-to-accelerate-adoption-of-composable-web-architectures/

Netfily が Gatsby を買収しました。記事の中では具体的な今後の取組等は特に記載されていませんでしたが、Gatsby と Netlify の動向が気になりますね。

## Interop 2023: continuing to improve the web for developers

https://web.dev/interop-2023/

ブラウザの互換性を向上するための取組が Interop2023 として継続して行われることが紹介されています。

Interop2023 では、26 個以上の機能にフォーカスしています。具体的には次の機能のように、Web アプリケーションを開発する上で利用できる便利な機能のブラウザの互換性を向上を目標にしている様です。

- Container Queries in CSS
- Custom Properties in CSS
- Media Query4
- Flexbox
- Grid
- `:has()`
- Web Components

## Turbopack が webpack loader をサポート

https://twitter.com/jaredpalmer/status/1619071988181651456

Turbopack で webpack loader が読み込める様になる機能が canary に実装され、次のリリースに入りそうです。

このツイートからも、Turbopack へ移行する際に、問題になりそうな webpack loader を重要視して開発チームが取り組んでいることが伺えます。また、[Turbopack(Rust)と webpack loader(Node.js)のやりとりは IPC ペース](https://twitter.com/jaredpalmer/status/1619074649630842880)で行うそうです。

## tc39 のミーティングが開催され、プロポーザルの stage が変わりました

https://twitter.com/robpalmer2/status/1621234674327605248

次のプロポーザルが stage4 になりました。

- Change Array by Copy
- Intl.NumberFormat V3
- Symbols as WeakMap keys

配列に対してソートなどの操作を immutable に行える Change Array by Copy は便利で良さそうです。

## Learn Images

https://web.dev/learn-images/

web.dev の Learn シリーズに画像のコースが新しく追加されました。

画像フォーマットの特徴(GIF, PNG, JPEG など)や画像のパフォーマンス、`<picture>` を利用したレスポンシブな画像の実装方法などが紹介されています。

## Component Story Format 3 is here

https://storybook.js.org/blog/storybook-csf3-is-here/

Storybook の Component Story Format 3(CSF) が完全にリリースされました。以前から CSF3 は利用できましたが、`useEvent` が内部でバージョンアップされたり、Story のタイトルがディレクトリ構造に合わせて自動で生成されるようになっています。

## Resumable React: How To Use React Inside Qwik

https://www.builder.io/blog/resumable-react-how-to-use-react-inside-qwik

Qwik アプリケーションの中で、React のランタイムなしに React コンポーネントを読み込む方法を紹介しています。

`host:onClick$={() => {}}` のように [`host:` ディレクティブ](https://qwik.builder.io/integrations/integration/react/#listen-to-dom-events-without-hydration) を利用することでハイドレーションせずに DOM のイベントを listen することで実現しています。

なお、記事内で紹介されている機能はまだ beta 版とのことです。

## PSOne.css

https://micah5.github.io/PSone.css/

この CSS フレームワークを使って FF7 のメニュー画面を作りましょう。

# あとがき

いかがだったでしょうか。個人的には、本記事で紹介した Deno や Qwik の記事、モダンなフレームワークが提供する機能からも CSR から SSR にトレンドが変化し始めてようで、ワクワクしています。

---

フロントエンドエキスパートチームでは毎月、最終火曜日の 17 時から Frontend Monthly というイベントを Youtube Live で開催しています。その月のフロントエンド注目ニュースやゲストを呼んでの対談などフロントエンドに関する発信していますので是非どうぞ！

https://www.youtube.com/playlist?list=PLPTndynQK4dxLZFEZgOZjt_zKG-0JWoWy

またフロントエンドエキスパートチームでは、一緒にサイボウズのフロントエンドを最高にする仲間を募集しています。興味ある方はこちら ↓ から！

https://cybozu.github.io/frontend-expert/
