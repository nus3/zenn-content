---
title: "VSCodeを使ってJavaScriptをデバッグする色々な方法"
emoji: "📘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vscode", "javascript"]
published: false
---

## 書きたいこと

- Vitest 見たら debugger の項目があって、実際にデバッガを起動してみるとめっちゃコードを読みやすくて良かった話
- Node.js でのデバッグ方法
- TypeScript のデバッグ方法
- Deno のデバッグ方法
- Bun のデバッグ方法

## VSCode のデバッグのドキュメントを読む

https://code.visualstudio.com/docs/editor/debugging

- Ndde.js ランタイムのデバッグサポートが組み込まれてる

### デバッグアクション

デバッグツールバーは左から

- 次のブレークポイントまでのプログラムを実行する
- 次のメソッドを実行する
- 1 行ずつ追跡
- 現在のメソッドの残りの行を実行
- 再起動
- 停止

### Logpoints

- ブレークポイントの一種
- 実際に実行を止めずに、ログをコンソールに出力できる

### launch.json

- .vscode/launch.json にデバッグの構成が定義できる
- `type`: デバッガーのタイプ
- `request`: launch か attach。デバッガーの起動 notaipu

### ブレークポイントのトピック

- 式が true になった時にブレークポイントの設定ができる
- トリガーされたブレークポイント: 別のブレークポイントに到達すると自動的に有効になるブレークポイント
- 関数名で指定してブレークポイントを設定できる

### Debug Console REPL

デバッグ中に、その状態で式などを実行し、コンソールに出力できる

## Deno

https://docs.deno.com/runtime/manual/references/vscode_deno/#using-the-debugger

VSCode の Deno 拡張をインストールし、`launch.json`を以下のようにし、program 部分で実行したいファイルを指定することでデバッグができるように。

```json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "request": "launch",
      "name": "Launch Program",
      "type": "node",
      "program": "${workspaceFolder}/main.ts",
      "cwd": "${workspaceFolder}",
      "runtimeExecutable": "/opt/homebrew/bin/deno",
      "runtimeArgs": ["run", "--inspect-wait", "--allow-all"],
      "attachSimplePort": 9229
    }
  ]
}
```

## Bun

デバッグできそう
ちょっとやり方違いそうだけど
https://bun.sh/guides/runtime/vscode-debugger

> VSCode extension support is currently buggy.

Bug が多いとのこと。
ただ VScode 拡張をインストールすることで、`Bun: Debug File`コマンドが使えるようになり、デバッグができるようになる。

## TypeScript

https://code.visualstudio.com/docs/typescript/typescript-debugging

- ビルトインの Node.js デバッガーが TypeScript をサポートしてる
- JS のソースマップがサポートされている
- tsc でコンパイルされた js を実行し、デバッグを行いたい場合は tsconfig で sourcemap を true にする必要がある
- TS でソースマップを生成するには`--sourcemap`オプションをつけるか tsconfig.json で`compilerOptions`の`sourcemap`を`true`にする
- tsx で実行した場合、普通に tsconfig の soucemap を true にしなくてもデバッグできる
- tsx は内部でソースマップを生成してそう
  - https://github.com/privatenumber/tsx/blob/develop/src/source-map.ts
  - https://github.com/privatenumber/tsx/blob/develop/src/cjs/index.ts
- vitest はなんで TS 上でデバッグできるの？
- Vitest で TS のデバッグができてるのはビルド時にソースマップの対応をしてるからなのか？
  - ビルドされたファイルには以下のように sourceMappingURL が記載されており、base64 エンコードされた JSON が含まれている
  - この JSON の中に元の TS ファイルがいる

```js
//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJmaWxlIjoiYnJvd3Nlci5qcyIsInNvdXJjZXMiOltdLCJzb3VyY2VzQ29udGVudCI6W10sIm5hbWVzIjpbXSwibWFwcGluZ3MiOiI7Ozs7OzsifQ==
```

- VSCode のデバッガも sourceMappingURL に対応してる？

## React

https://code.visualstudio.com/docs/nodejs/reactjs-tutorial#_debugging-react

launch.json を定義

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "msedge",
      "request": "launch",
      "name": "Launch Edge against localhost",
      "url": "http://localhost:3000",
      "webRoot": "${workspaceFolder}"
    }
  ]
}
```

- 開発サーバーを起動する `npm run dev`などで
- url のポートを開発サーバーのポートと合わせる
- Run and Debug を実行するとデバッグできる

## リンク

https://code.visualstudio.com/docs/nodejs/nodejs-debugging
https://github.com/microsoft/vscode-js-debug/blob/main/README.md
