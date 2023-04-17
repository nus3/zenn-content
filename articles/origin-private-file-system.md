---
title: "The origin private file system"
emoji: "🌟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["webapi"]
published: false
---

## メモ

### https://developer.chrome.com/articles/origin-private-file-system/#browser-support を読んでみて

- オリジンプライベートファイルシステム(OPFS)でページのオリジンに非公開でユーザーには見えないストレージエンドポイント導入できる？
- テキストファイルを Web アプリケーションで開く従来の方法
  - <input type="file">でファイルを開く
  - <a download="ToDo.txt">でダウンロードする
  - folder を開くには <input type="file" webkitdirectory>を使う
- モダンな方法は File System API を使う
  - ToDo.txt を`showOpenFilePicker()`で開き、FileSystemFileHandle オブジェクトを取得
  - FileSystemFileHandle オブジェクトから getFile()で File 取得する
  - ファイルを変更し、requestPermission({mode: 'readwrite'})でハンドルする
  - ユーザーがアクセプトしたら、変更内容を元のファイルに保存
  - showSaveFilePicker()でも、ファイルを更新することができる
- これらの方法はユーザーから見えるファイルシステムで行う
- ウェブから保存されたファイルはセキュリティ的に危ないから OS が追加の警告をできるようになっている
- File System Access API での操作は遅くなるかも
- ファイルはデータを記録するために優れてる
  - SQLite は DB を単一ファイルに保存する
- 従来の Web ベースのファイル処理をパフォーマンスのコストなしに使う方法としてオリジンプライベートファイルシステム
- オリジンプライベートファイルシステムはユーザーには表示されない
- プライベートなファイルとフォルダでオリジンに対してプライベート
- 同一のオリジンであればオリジンプライベートファイルシステムを確認できる
- OPFS は navigator.storage.getDirectory()でアクセスできる
  - 最初は空
- ルートディレクトリ以外は同じ扱いができる
- OPFS はユーザーに表示されないため、許可プロンプトや安全なブラウジングチェックはない
- OPFS は全てのブラウジングデータやサイトデータを削除すると、削除される(localStorage, IndexDB と同様)
- ストレージの使用量も確認できる
- 仕様は WHATWG の File System Living Standard で策定されてる
- Firefox111 でリリースされたことで、主要なブラウザで利用できる
- OPFS のルートディレクトリにアクセスする

```js
const opfsRoot = await navigator.storage.getDirectory();
```

- OPFS はメインスレッドと Web Worker から使用できる
- Web Worker だと OPFS は同期的に扱える？
- ファイル操作を最速で行う場合は Web Worker 上で OPFS を扱う方がいいっぽい
- メインスレッドでファイル操作する
  - getFileHandle('file name', {create: true})や getDirectoryHandle('directory name', {create: true})でファイルやフォルダを作成できる
  - すでに同じファイル名やフォルダ名がある場合は、そのファイルやフォルダにアクセスする
  - createWritable()を使うことでデータをファイルにストリームでき、内容を書き込むことができる
    - ストリームは最後に close()する必要がある
  - ファイル、フォルダの削除
    - remove()は現在 Chrome のみ実装

```js
await fileHandle.remove();
await directoryHandle.remove({ recursive: true });
```

- chrome で実装された move()メソッドを使うことでファイルやファルだのリネームや移動ができる
- resolve()でファイルやフォルダのパスを確認できる

TODO: https://developer.chrome.com/articles/origin-private-file-system/#using-the-origin-private-file-system-in-a-web-worker から読む
