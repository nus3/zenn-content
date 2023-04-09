---
title: "オリジントライアルになったDocument Picture-in-Picture APIを試してみよう"
emoji: "📺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["webapi"]
published: false
publication_name: "cybozu_frontend"
---

:::message
Document Picture-in-Picture API は策定中のものです。本記事で紹介した内容が変わる可能性があります。
:::

Chrome111 から Document Picture-in-Picture API がオリジントライアルで試せるようになりました。

https://developer.chrome.com/docs/web-platform/document-picture-in-picture/

## Picture-in-Picture とは？

次の画像のように、画面の一部に別でウインドウを表示することを Picture-in-Picture といいます。

![Picture-in-Picture を使って、右下にウインドウを表示する様子](/images/picture-in-picture-api/pip.png =500x)
_[Picture-in-Picture のサンプルページ](https://googlechrome.github.io/samples/picture-in-picture/)_

動画を右下で流しつつ、別の WEB サイトを確認するなど、1 つの画面で 2 つのウインドウが表示できます。

## Document Picture-in-Picture API とは？

今回新たにオリジントライアルに追加された Document Picture-in-Picture API とは別に [Picture-in-Picture API](https://w3c.github.io/picture-in-picture/) の仕様が [Media Working Group](https://www.w3.org/media-wg/) によって策定されています。

この Picture-in-Picture API では使用できる要素が `<video>` のみでしたが、 Document Picture-in-Picture API では `<video>` 以外の要素も使用できるようになります。

## Document Picture-in-Picture API をどのように使うか

実際に Document Picture-in-Picture API を使って Picture-in-Picture のウインドウの中にストップウォッチを表示してみます。

オリジントライアルのトークンが有効な間は、次のリンクから実装したものを確認できます。

https://nus3.github.io/ui-labs/pip/

![Document Picture-in-Picture API を使ってストップウォッチを表示](/images/picture-in-picture-api/sample-pip.png =500x)
_Picture-in-Picture のウインドウにストップウォッチを表示_

実装したコードは[nus3/ui-labs](https://github.com/nus3/ui-labs/tree/main/src/pip)にあります。

### Picture-in-Picture ウインドウを表示する

Document Picture-in-Picture API では `documentPictureInPicture` を使って Picture-in-Picture の操作をします。

`documentPictureInPicture.requestWindow()` を実行することで Picture-in-Picture のウインドウが表示されます。

`requestWindow()` はウインドウが表示した際に resolve する Promise を返します。この Promise は、クリックなどのユーザー操作時以外の場合には reject します。

```ts
// ボタンをクリックした時にPicture-in-Pictureのウインドウを表示する
button.addEventListener("click", async () => {
  await documentPictureInPicture.requestWindow();
});

// ユーザー操作時以外の場合は以下のようなエラーを返します
// Uncaught (in promise) DOMException: Failed to execute 'requestWindow' on 'DocumentPictureInPicture': Document PiP requires user activation
documentPictureInPicture.requestWindow();
```

### 元のウインドウで定義した CSS を Picture-in-Picture ウインドウにもコピーする

`requestWindow()` には引数にオプションを指定できます。オプションの 1 つに `copyStyleSheets` があり、これを true にすると元のウインドウの CSS がコピーされます。

```ts
button.addEventListener("click", async () => {
  await documentPictureInPicture.requestWindow({
    copyStyleSheets: true,
  });
});
```

この他にオプションでは Picture-in-Picture のアスペクト比と幅、高さを指定できます。

### Picture-in-Picture ウインドウに表示する要素を追加する

`requestWindow()` か `documentPictureInPicture.window` を使うことで、Picture-in-Picture ウインドウの `Window` インスタンスを取得できます。

取得した `Window` インスタンスに対して要素を追加することで、Picture-in-Picture ウインドウに追加した要素を表示できます。

```tsx
<div id="timerWrapper">
  <span id="time">00:00:00.000</span>
  <!-- ... -->
</div>

button.addEventListener("click", async () => {
  // Picture-in-Picture ウインドウに表示したい要素を取得
  const player = document.querySelector("#timerWrapper");
  // Picture-in-Picture ウインドウのインスタンスをrequestWindow()の返り値として取得
  const pipWindow = await documentPictureInPicture.requestWindow();
  // Picture-in-Picture ウインドウに表示したい要素を追加
  pipWindow.document.body.append(player);
});

// Picture-in-Picture ウインドウが表示されている場合は、下記でも取得できる
const pipWindow = documentPictureInPicture.window
```

### Picture-in-Picture ウインドウが閉じる際の処理を追加する

Picture-in-Picture ウインドウは閉じるアイコンをクリックするか、`close()` メソッドを実行することでウインドウを閉じることができます。

![Picture-in-Picture ウインドウの閉じるアイコン](/images/picture-in-picture-api/close.png =400x)

```ts
const pipWindow = await documentPictureInPicture.requestWindow();
pipWindow.close();
```

また、Picture-in-Picture ウインドウの `Window` インスタンスに対して `unload` イベントを登録することでウインドウが閉じた際の処理を追加できます。

```ts
button.addEventListener("click", async () => {
  // Picture-in-Picture ウインドウのインスタンスをrequestWindow()の返り値として取得
  const pipWindow = await documentPictureInPicture.requestWindow();

  pipWindow.addEventListener("unload", (event) => {
    // Picture-in-Picture ウインドウが閉じた際の処理をここに書ける
  });
});
```

### ストップウォッチを Picture-in-Picture ウインドウに表示する

今回は、`requestWindow()` で取得した Picture-in-Picture ウインドウのインスタンスに対して、ストップウォッチの要素を追加したり、イベントハンドラの追加、削除をしています。

```ts
button.addEventListener("click", async () => {
  // ストップウォッチの要素を取得
  const player = document.querySelector("#timerWrapper");
  // Picture-in-Picture ウインドウを表示し、元のCSSをコピー
  const pipWindow = await documentPictureInPicture.requestWindow({
    copyStyleSheets: true,
  });
  // Picture-in-Picture ウインドウにストップウォッチ要素を表示
  pipWindow.document.body.append(player);

  // Picture-in-Picture ウインドウ内のタイマー部分の要素を取得
  const time = pipWindow.document.getElementById("time");
  // startボタンを押された時のイベントハンドラを定義
  const handleStart = () => {
    start(time);
  };

  // Picture-in-Picture ウインドウ内のスタートボタンに対してクリックイベントを登録
  const startBtn = pipWindow.document.getElementById("start");
  startBtn?.addEventListener("click", handleStart);

  pipWindow.addEventListener("unload", (event) => {
    // Picture-in-Picture ウインドウが閉じられる時にスタートボタンのクリックイベントを削除
    startBtn?.removeEventListener("click", handleStart);
  });
});
```

## Document Picture-in-Picture API 関連リンク

- [Picture-in-Picture API](https://w3c.github.io/picture-in-picture/)
  - ステータスは Editors' Draft
  - [Media Working Group](https://www.w3.org/media-wg/)が進めている
  - https://developer.chrome.com/blog/watch-video-using-picture-in-picture/
- [Document Picture-in-Picture API](https://wicg.github.io/document-picture-in-picture/)
  - ステータスは Unofficial Draft
  - [Web Incubator CG](https://wicg.io/)が進めている
    - https://github.com/WICG/document-picture-in-picture

## Special Thanks

この記事を書くにあたり、ストップウォッチ部分の実装は ChatGPT さんにお願いしました。

![ChatGPT さんにストップウォッチの実装を教えてもらった](/images/picture-in-picture-api/chat-gpt.png =400x)

## 最後に

まだ仕様の策定段階ですが、Document Picture-in-Picture API は動画だけでなく、今回実装したストップウォッチのように色々な場面で利用できそうでおもしろいですね。

Chrome111 であれば[フラグを有効にする](https://developer.chrome.com/docs/web-platform/document-picture-in-picture/#local-testing)だけで簡単にローカルで試せるので、興味のある方はぜひ試してみてください。
