---
title: "Origin Private File Systemを使ってブラウザ上でファイルを高速に操作しよう"
emoji: "📁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["webapi"]
published: false
---

## 3 行まとめ

- Origin Private File System という名の通り、Origin に紐づくプライベートなファイルシステムが扱える API
- Firefox 111 で実装されたことで、主要ブラウザで使える
- Origin Private File System は FileSystem Access API よりもファイル操作のパフォーマンスが良い

## Origin Private File System とは

https://developer.chrome.com/articles/origin-private-file-system/

Origin と紐づき、ユーザには非公開なブラウザ上で扱えるファイルシステムです。

## なぜ Origin Private File System を使うのか

Origin Private File System を使わずとも `Blob` と `URL.createObjectURL()` を組み合わせて `a` 要素をクリックすることでファイルをダウンロードする方法や、File System Access API の `window.showOpenFilePicker()` など使うことでファイル操作はできます。

しかし、これらの方法は手元にダウンロードしたファイルが残ったり、ユーザーから見えるフォルダ、ファイルが対象なので安全性の観点からファイル操作のパフォーマンスが最適化されないなどの問題があります。

参考: [Google Safe Browsing](https://safebrowsing.google.com/)

Origin Private File System を使うことで、ユーザーには見えないフォルダ、ファイルとして扱えます。また、Origin Private File System 上でのファイル操作は Safe Browsing の対象にはならないので、その分、パフォーマンスのコストを考えずにファイルの操作ができます。

実際に「SQLite3 WASM/JS」では、Origin Private File System が採用されています。

参考: [SQLite Wasm in the browser backed by the Origin Private File System](https://developer.chrome.com/blog/sqlite-wasm-in-the-browser-backed-by-the-origin-private-file-system)

## Origin Private File System を使ってファイルを操作してみよう

実際に Origin Private File System 上でファイルの作成、読込、更新、削除を実装してみましょう。

次のリンクから実装したものを確認できます。

https://nus3.github.io/ui-labs/opfs/

また、[nus3/ui-labs](https://github.com/nus3/ui-labs/tree/main/src/opfs)でコードも確認できます。

### ファイルの作成

### ファイルの更新

### ファイルの読込

### ファイルの削除

### ファイルの移動

### Web Worker 上でファイルを更新

### Debug

各ブラウザの DevTools が対応するまでは[OPFS Explorer](https://chrome.google.com/webstore/detail/opfs-explorer/acndjpgkpaclldomagafnognkcgjignd)を使うことで、Origin Private File System 上のファイルとフォルダを確認できます。

![デバッグに使えるOPFS Explorer拡張](/images/origin-private-file-system/opfs-explorer.png)
_OPFS Explorer でフォルダとファイル_

## 最後に

Origin Private File System の API が提供されることで、ファイルシステムの仕組みが簡単に実装できるようになって良いですね。クライアントサイドで完結するメモアプリなどを作ってみても楽しそうと筆者は感じました。
