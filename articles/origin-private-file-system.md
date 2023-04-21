---
title: "Origin Private File Systemを使ってブラウザ上でファイルを高速に操作しよう"
emoji: "📁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["webapi"]
published: true
---

## 3 行まとめ

- Origin Private File System という名の通り、Origin に紐づくプライベートなファイルシステムが扱える API
- Firefox 111 で実装されたことで、主要ブラウザでほとんどの機能が使える[^1]
- Origin Private File System は FileSystem Access API よりもファイル操作のパフォーマンスが良い

[^1]: [FileSystemWritableFileStream](https://developer.mozilla.org/ja/docs/Web/API/FileSystemWritableFileStream)が Safari で使えないなど、一部、実装されていない機能がある。

## Origin Private File System とは

https://developer.chrome.com/articles/origin-private-file-system/

Origin と紐づき、ユーザには非公開なブラウザ上で扱えるファイルシステムです。

## なぜ Origin Private File System を使うのか

Origin Private File System を使わずとも `Blob` と `URL.createObjectURL()` を組み合わせて `a` 要素をクリックすることでファイルをダウンロードしたり、File System Access API の `window.showOpenFilePicker()` などを使うことでファイル操作はできます。

しかし、これらの方法は手元にダウンロードしたファイルが残ったり、ユーザーから見えるフォルダ、ファイルが対象なので安全性の観点からファイル操作のパフォーマンスが最適化されないなどの問題があるようです。

> When you write data to a file using the File System Access API, writes are not in-place, but use a temporary file. The file itself is not modified unless it passes all these security checks
> 意訳: File System Access API を使ってファイルにデータを書き込む場合、セキュリティチェックをパスしない限り変更されない。

参考: [Google Safe Browsing](https://safebrowsing.google.com/)

Origin Private File System を使うことで、ユーザーには見えないフォルダ、ファイルとして扱えます。また、Origin Private File System を使うことで、パフォーマンスが最適化された方法でファイルにアクセスできるようです。

実際に「SQLite3 WASM/JS」では、Origin Private File System が採用されています。

参考: [SQLite Wasm in the browser backed by the Origin Private File System](https://developer.chrome.com/blog/sqlite-wasm-in-the-browser-backed-by-the-origin-private-file-system)

## Origin Private File System を使ってテキストファイルを操作してみよう

Origin Private File System 上でテキストファイルの作成、読込、書込を実装してみましょう。

次のリンクからデモを確認できます。

https://nus3.github.io/ui-labs/opfs/

また、[nus3/ui-labs](https://github.com/nus3/ui-labs/tree/main/src/opfs)でコードも確認できます。

### Origin Private File System のルートを取得する

`navigator.storage.getDirectory()` を実行すると `resolve` 時に Origin Private File System のルートディレクトリの操作ができる [`FileSystemDirectoryHandle`](https://developer.mozilla.org/ja/docs/Web/API/FileSystemDirectoryHandle) インスタンスを返す `Promise` が取得できます。

このインスタンスを元に Origin Private File System のファイルやフォルダを操作します。

### ファイル・フォルダの作成

`FileSystemDirectoryHandle` のメソッドである `getDirectoryHandle()` と `getFileHandle()` をオプションを指定して実行することでファイルとフォルダが作成できます。

```js
// ルートを取得
const opfsRoot = await navigator.storage.getDirectory();
// create: trueを指定することでファイル・ディレクトリがない場合にルート直下に新たに作成する
// ファイルの場合
await opfsRoot.getFileHandle("nus3File", {
  create: true,
});
// フォルダの場合
await opfsRoot.getDirectoryHandle("nus3Dir", {
  create: true,
});
```

`getDirectoryHandle()` はフォルダがあった場合は、そのフォルダの `FileSystemDirectoryHandle` インスタンスを返してくれます。このインスタンスを使って特定のフォルダ配下のファイルにアクセスできます。

```js
// ルートを取得
const opfsRoot = await navigator.storage.getDirectory();
// ルート直下のnus3Dirディレクトリを取得
const dirHandle = await opfsRoot.getDirectoryHandle("nus3Dir");
// nus3Dir直下にnus3-1Fileを作成する
await dirHandle.getFileHandle("nus3-1File", {
  create: true,
});
```

デモ画面では File name に作成したいファイル名を入力して Create ボタンを押すとファイルが作成できます。

![デモ画面でファイルを作成する](/images/origin-private-file-system/create-file.gif =400x)

同様に Directory name に作成したいディレクトリ名を入力して Create Directory ボタンを押すとディレクトリが作成できます。

### ファイルの書込

`getFileHandle()` で取得する `FileSystemFileHandle` のメソッドである[`createWritable()`](https://developer.mozilla.org/ja/docs/Web/API/FileSystemFileHandle/createWritable)を実行することでファイルに書き込みができる [`FileSystemWritableFileStream`](https://developer.mozilla.org/ja/docs/Web/API/FileSystemWritableFileStream) インスタンスを取得できます。

この `FileSystemWritableFileStream` インスタンスの `write()` メソッドを実行するとファイルにデータを書き込めます。

```js
const opfsRoot = await navigator.storage.getDirectory();
// nus3FileのFileSystemFileHandleを取得
const fileHandle = await opfsRoot.getFileHandle("nus3File");
// nus3Fileに書き込みができるFileSystemWritableFileStreamを取得
const writable = await fileHandle.createWritable();
// nus3FileにHello nus3を書き込む
await writable.write("Hello nus3");
// ディスクに書き込む
await writable.close();
```

デモ画面では、File name に書き込みたいファイル名を入力し、Text でファイルに書き込みたい内容を入力してから Write ボタンを押すとファイルに書き込みができます。

![デモ画面でファイルに書き込む](/images/origin-private-file-system/write-file.gif =400x)

### ファイルの読込

`FileSystemFileHandle` のメソッドである [`getFile()`](https://developer.mozilla.org/ja/docs/Web/API/FileSystemFileHandle/getFile) を使うことで、ファイルの内容を確認できる `File` オブジェクトを取得できます。

```js
const opfsRoot = await navigator.storage.getDirectory();
// nus3FileのFileSystemFileHandleを取得
const fileHandle = await opfsRoot.getFileHandle("nus3File");
// nus3FileのFileオブジェクトを取得
const file = await fileHandle.getFile();
// ファイルに書き込まれた内容が出力される
console.info(await file.text());
```

### Web Worker 上でファイルの書込

ここまでのファイル操作は基本的に非同期で実行していましたが、Web Worker 上であれば `FileSystemFileHandle` のメソッドである `createSyncAccessHandle()` を使うことで同期的にファイルの読込や書込ができます。

今回はメインスレッドから `postMessage()` 経由で指定されたファイル名とテキストを元に、該当のファイルにテキストを書き込みます。

まずは Web Worker 上で実行する処理を `worker.js` に定義します。

```js:worker.js
onmessage = async (e) => {
  // メインスレッドで実行されたpostMessage()からファイル名とテキストを取得
  const { fileName, text } = e.data
  // Origin Private File Systemにアクセスし、該当のファイルを取得
  const opfsRoot = await navigator.storage.getDirectory()
  const fileHandle = await opfsRoot.getFileHandle(fileName, { create: true })

  // ファイルの読込や書込を同期的に扱えるFileSystemSyncAccessHandleを取得
  const accessHandle = await fileHandle.createSyncAccessHandle()

  // テキストのエンコードとデコードをするためにインスタンスを取得
  const textEncoder = new TextEncoder()
  const textDecoder = new TextDecoder()

  // メインスレッドから受け取ったテキストをエンコード
  const content = textEncoder.encode(`${text}`)
  // ファイルの先頭から内容を同期的に書き込む
  accessHandle.write(content, { at: 0 })
  // 変更をディスクに書き込み、FileSystemSyncAccessHandleを閉じる
  accessHandle.flush()
  accessHandle.close()
}
```

次にメインスレッド側からファイル名とテキストを `postMessage()` で渡します。

今回のデモでは[Vite の Query Suffixes](https://vitejs.dev/guide/features.html#web-workers)を使って `worker.js` をインポートしています。

```js
import MyWorker from "./worker?worker";
const worker = new MyWorker();
// 定義したworkerにファイル名とテキストを渡す
worker.postMessage({ fileName: "nus3File", text: "Hello nus4" });
```

### Origin Private File System 上のフォルダ、ファイルを確認する

各ブラウザの DevTools が対応するまでは[OPFS Explorer](https://chrome.google.com/webstore/detail/opfs-explorer/acndjpgkpaclldomagafnognkcgjignd)を使うことで、Origin Private File System 上のファイルとフォルダを確認できます。

![デバッグに使えるOPFS Explorer拡張](/images/origin-private-file-system/opfs-explorer.png)
_OPFS Explorer でフォルダとファイル_

## 最後に

Origin Private File System の API がサポートされたことで、ファイルシステムの仕組みを簡単に利用できるようになって良いですね。クライアントサイドで完結するメモアプリなどを作ってみるのも楽しそうと感じました。
