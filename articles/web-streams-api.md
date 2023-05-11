---
title: "Streaming with SuspenseのStreamingを理解する"
emoji: "♨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["webapi"]
published: true
publication_name: "cybozu_frontend"
---

## 3 行まとめ

- Next.js の App Router では `<Suspense>` を使ったストリーミングがサポートされている
- React には、Node.js Streams と Web Streams に対応する Server API がある
- [Streams API](https://developer.mozilla.org/ja/docs/Web/API/Streams_API)の `ReadableStream` を使ってサーバーサイドから値を段階的に受け取る(ストリーミングする)方法を紹介

## Streaming with Suspense

https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming#streaming-with-suspense

Next.js の App Router では `<Suspense>` を使ったストリーミングがサポートされています。

`<Suspense>` を使ったストリーミングではページの HTML を小さな塊に分解し、その塊をクライアントに順次送信できます。この `<Suspense>` を使うことで、ページの一部をより早く表示できます。

Next.js のドキュメントにある[Example](https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming#example)のコード(一部内容を変更)を例に見てみましょう。

```tsx
import { Suspense } from "react";
import { PostFeed, Weather } from "./Components";

export default function Posts() {
  return (
    <section>
      <Suspense fallback={<p>Loading feed...</p>}>
        <PostFeed />
      </Suspense>
      <Suspense fallback={<p>Loading weather...</p>}>
        <Weather />
      </Suspense>
      <Nus3 /> {/* 非同期の処理がないコンポーネント */}
    </section>
  );
}
```

この例の場合、非同期処理がない `<Nus3 />` コンポーネントはすぐに表示されます。`<Suspense>` で囲われた `<PostFeed />` と `<Weather />` は、まず `fallback` に渡した内容(`<p>Loading feed...</p>` など)が表示されます。その後、`<PostFeed />` と `<Weather />` は、それぞれのコンポーネント内部で実行する非同期処理が完了すると、ストリーミングを使って表示する内容が段階的にクライアントサイドに送信されます。

## React の Streaming ってどうやってるの？

React には、Node.js Streams と Web Streams を使った Server API が用意されています。

https://react.dev/reference/react-dom/server

例えば Web Streams の場合、[`renderToReadableStream`](https://react.dev/reference/react-dom/server/renderToReadableStream)を使うことで、React tree を [`ReadableStream`](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream)にレンダリングできます。

## 実際に Streaming してみる

`ReadableStream` を使ってサーバーサイドから値をストリーミング(段階的に受け取る)してみましょう。

具体的には次のような流れになります。

1. クライアントサイドで　`/api/streams` にリクエストを送信
2. サーバーサイドでは `ReadableStream` をレスポンスで返す
3. レスポンスで返す `ReadableStream` では、すぐにテキストを送信(`<Suspense>` の `fallback` を想定)
4. その後 2 秒待つ(コンポーネントの非同期処理中を想定)
5. 別のテキストを送信(非同期処理が完了したコンポーネントを想定)
6. クライアントサイド側では `ReadableStream` の内容を取得し、表示する

次のリンクからデモを確認できます。

https://nus3-fresh-labs.deno.dev/streams-api

また、[nus3/fresh-labs](https://github.com/nus3/fresh-labs)でコードも確認できます。

### `ReadableStream` を返すエンドポイントを作る

今回は Deno + Fresh を使ってデモを実装したので、Streaming 部分には Deno でも[サポート](https://deno.land/api@v1.33.2?s=ReadableStream)されている `ReadableStream` を使います。

前述しましたが、React の[`renderToReadableStream`](https://react.dev/reference/react-dom/server/renderToReadableStream)を使うことで React tree を `ReadableStream` にレンダリングできます。

実際に `renderToReadableStream` の実装を見てみると `ReadableStream` が使われていることが確認できます。

https://github.com/facebook/react/blob/df12d7eac40c87bd5fdde0aa5a739bce9e7dce27/packages/react-dom/src/server/ReactDOMFizzServerBrowser.js#L58-L70

Streaming の挙動を理解するために今回は、 `renderToReadableStream` の React tree をレンダリングしてくれる部分をただのテキストを返す擬似的な `pseudoRenderToReadableStream` 関数を作りましょう。

```ts
const pseudoRenderToReadableStream = () => {
  const stream = new ReadableStream({
    async pull(controller) {
      const firstMessage = new TextEncoder().encode("Loading feed...");
      // ストリームのキューに"Loading feed..."を入れる
      controller.enqueue(firstMessage);

      // 2秒待つ(コンポーネントの非同期処理中を想定)
      await sleep(2000);

      const resolvedMessage = new TextEncoder().encode("PostFeed Content");
      // ストリームのキューに"PostFeed Content"を入れる
      controller.enqueue(resolvedMessage);

      // ストリームを閉じる
      controller.close();
    },
  });
};
```

`ReadableStream` はインスタンス生成時にコンストラクタとして[ `start()`、`pull()`、`cancel()`](https://developer.mozilla.org/ja/docs/Web/API/Streams_API/Using_readable_streams#readablestream_%E3%82%B3%E3%83%B3%E3%82%B9%E3%83%88%E3%83%A9%E3%82%AF%E3%82%BF%E3%83%BC) などを受け取ります。

今回は実際の `renderToReadableStream` でも使われていた `pull()` を使って実装しています。この `pull()` はキューがいっぱいになるまで繰り返し呼び出されるメソッドです。

今回の実装の場合、まず最初にエンコードされた `Loading feed...` が `controller.enqueue()` を使ってストリームのキューに入れられ、その後 2 秒待ってから、エンコードされた `"PostFeed Content"` がキューに入れられます。その後、`controller.close()` を実行することでストリームを閉じています。

あとは、`/api/stream` にリクエストが来た時に `pseudoRenderToReadableStream` で生成した `ReadableStream` をレスポンスで返すようにしましょう。

```ts:routes/api/stream.ts
export const handler = (_req: Request, _ctx: HandlerContext): Response => {
  const stream = pseudoRenderToReadableStream();
  return new Response(stream);
};
```

### クライアントサイドで `ReadableStream` を読み取る

クライアントサイド側で `/api/stream` のレスポンスとして `ReadableStream` を読み込み、UI 上に表示しましょう。

[`ReadableStream.getReader()`](https://developer.mozilla.org/ja/docs/Web/API/ReadableStream/getReader) を使うことでストリームのデータを読み取れます。

```ts
const pump = async (reader: ReadableStreamDefaultReader<Uint8Array>) => {
  const content = await reader.read();
  // done === trueの場合、すべてのキューの読み取りが完了
  if (content.done) return;
  // content.valueにはエンコードされた値が格納されているのでデコードする
  const value = new TextDecoder().decode(content.value);

  // ...取得した値を表示する

  // キューがなくなるまで再帰的に実行
  pump(reader);
};

// ReadableStreamを受け取る
const res = await fetch("/api/stream");
const reader = res.body?.getReader();
if (reader) {
  pump(reader);
}
```

[`ReadableStream.getReader()`](https://developer.mozilla.org/ja/docs/Web/API/ReadableStream/getReader) を実行すると `ReadableStreamDefaultReader` が取得できます。このインスタンスのメソッドである[`ReadableStreamDefaultReader.read()`](https://developer.mozilla.org/ja/docs/Web/API/ReadableStreamDefaultReader/read)から、キューに入れた値とストリームが閉じられたかどうかを確認できます。

あとは読み取った値を UI に表示すると、`ReadableStream` から段階的に値を受け取っていることが確認できます。実際にデモ画面では最初 `Loading feed...` が表示され、次に `PostFeed Content` 表示される様子が確認できます。

![デモ画面で実際にStreamingしてみる](/images/web-streams-api/stream.gif =500x)

## 最後に

`ReadableStream` を使った Streaming の挙動の確認は意外と簡単に実装できるので、気になる方はぜひ触ってみてください！
