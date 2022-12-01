---
title: "Cloudflare PagesでNext.jsのSSR(Edge Runtime)を利用する"
emoji: "😽"
type: "tech"
topics: ["cloudflare", "nextjs"]
published: false
---

## Cloudflare Pages で Next.js の SSR の話

https://blog.cloudflare.com/next-on-pages/

## Next.js v13 の機能を試す話

app ディレクトリは beta なので config の設定が必要。

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    appDir: true,
  },
};

module.exports = nextConfig;
```

### App Directory Roadmap

https://beta.nextjs.org/docs/app-directory-roadmap

`app` ディレクトリのロードマップ。

現在実装されているものと、今後実装予定のもの。また実装しないものが記載されてる。

`next export` は実装予定。

### Rendering Fundamentals

https://beta.nextjs.org/docs/rendering/fundamentals

サーバーでレンダリングするには Node.js ランタイムと Web API に準拠した Edge ランタイム。

### Server and Client Components

https://beta.nextjs.org/docs/rendering/server-and-client-components

app ディレクトリはデフォルトでは React Server Components である。

Client Component を使う場合は app ディレクトリ内部にファイルを作成し、`"use client"` ディレクティブを追加する。

```tsx
"use client";

import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

useState や useEffect を使う場合にのみ `"use clinet"` ディレクティブ使ったほうが良い。そうすることでクライアントサイドの JS サイズを減らせることができる。

サードパーティパッケージにはまだこのディレクティブが追加されていないので、ラップして、ディレクティブの宣言をしている Client Component を自前に作る方がよい。

context は Server Components では使えない。
context を更新する再レンダリングするので、Client Component として扱う。

Server Components 間のデータの共有は、React の状態管理の話ではなく、サーバー側での話なので、普通にモジュールを共通して使うような設計で良し。

[Server Components の同じ内容のフェッチは自動的に重複が排除される。](https://beta.nextjs.org/docs/rendering/server-and-client-components#sharing-fetch-requests-between-server-components)

Client Components はなるべく、レイアウトの末端コンポーネントで使うようにする。

Client Components の中で、Server Components は使えない。

Server Components を Client Components の children、または、props として受け取ることは可能。

### Data Fetching Fundamentals

https://beta.nextjs.org/docs/data-fetching/fundamentals

app ディレクトリでは `getServerSideProps`, `getStaticProps`, `getInitialProps` はサポートされない。

app ディレクトリでは、必要なデータのフェッチは Server Components で行うことを推奨。

Server Components 内で同じリクエストを送った場合には、キャッシュされているので、重複したリクエストは削除されるので、同じデータを取得するのに複数回リクエストが飛ぶことはない。

現在、Client Component では fetch はサポートしていない。SWR などを使えば可能だが、パフォーマンスの問題がある。なのでできるだけ ServerComponent で `use` や `fetch` を使って取得したデータを Client Component に渡すほうが良い。

### Data Fetching

https://beta.nextjs.org/docs/data-fetching/fetching

Server Components である layout・page で fetch と async/await が使える。

Client Components では fetch が使えないが、新しい `use` を使い fetch をラップすることで呼び出すこともできる。

静的データ(`fetch` をキャッシュの設定を指定せずに使う)に対して、async/await を使うと `getStaticProps` と同等になる。

`fetch` はデフォルトで `force-cache`。

`fetch` 時に `cache: 'no-store'` を指定する場合は `getServerSideProps` と同等になる。

### Mutating

https://beta.nextjs.org/docs/data-fetching/mutating

現時点では `router.refresh()` を使って、Server Component 内のレンダリングされた内容が更新される。

```tsx
"use client";

import { useRouter } from "next/navigation";

async function update(id: number, completed: boolean, refresh: () => void) {
  await fetch(`https://api.example.com/todo/${id}`, {
    method: "PUT",
    body: JSON.stringify({ completed }),
  });

  // Refresh the current route and fetch new data from the server
  refresh();
}

export default function Todo(todo: Todo) {
  const router = useRouter();

  return (
    <li>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={(e) => update(todo.id, !todo.completed, router.refresh)}
      />
      {todo.title}
    </li>
  );
}
```

### Caching

https://beta.nextjs.org/docs/data-fetching/caching

同じリクエストはキャッシュされる前提を利用した preload パターンの実装が紹介されてる。

事前に必要そうなリクエストをコンポーネントのレンダリングするよりも前に呼び出しておくことで、実際にリクエストが必要な際にはキャッシュした値がすぐ返るような実装。

### Streaming and Suspense

https://beta.nextjs.org/docs/data-fetching/streaming-and-suspense

app ディレクトリでは、Node.js と Edge Runtime の両方で server-rendering 中に streaming と Suspense をサポート。

### Generating Static Params

https://beta.nextjs.org/docs/data-fetching/generating-static-params

動的なルーティングがある場合は、`getStaticPaths` の代わりに `generateStaticParams` を使う。

```tsx: app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await getPosts();

  return posts.map((post) => ({
    slug: post.slug,
  }));
}
```

### 参考ドキュメント

プレイグラウンドのリポジトリ
https://github.com/vercel/app-playground

プレイグラウンド
https://codesandbox.io/p/sandbox/nextjs-turborepo-example-1hhuf4?file=%2FREADME.md

Next.js の v13 に対応したドキュメント
https://beta.nextjs.org/docs/getting-started

React Server Component
https://postd.cc/how-react-server-components-work/

React での Promise の RFC
https://github.com/acdlite/rfcs/blob/first-class-promises/text/0000-first-class-support-for-promises.md

## TODO:

- Streaming と Suspense を試す
  - https://beta.nextjs.org/docs/data-fetching/streaming-and-suspense
