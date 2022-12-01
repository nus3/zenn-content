---
title: "Deno + Fresh + Storybookの環境を作る"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["deno", "fresh", "storybook"]
published: false
publication_name: "cybozu_frontend"
---

Deno と Fresh に Storybook を導入してみた。

## 作業手順

- Fresh の環境構築
- npx sb init で preact のプロジェクトを作成
- node_modules や package.json を削除
- 余分な設定も削除
- `deno run npm:@storybook/preact start-storybook -p 6006` を実行してみた
  - storybook で必要な依存が見つからないぞってエラー
- `deno run -A npm:sb init` npm scripts を実行してみた
  - deno.lock に storybook の依存が追加された
  - `deno run npm:@storybook/preact start-storybook -p 6006` を実行してみた
  - エラー
  - `deno run -A npm:@storybook/cli start-storybook -p 6006`
  - エラー
  - `deno run npm:@storybook/preact/start-storybook -p 6006`
- `deno run -A npm:@storybook/preact/start-storybook -p 6006` でいけそう
  - esm で扱うために main.cjs 変更したらできそうかも
- なんか npm v7 を選んでエラーになったかも？

## メモ

- preact の場合、`@storybook/preact` が該当のパッケージ。

storybook の依存のエラー。

```bash
thread 'main' panicked at 'could not find id @storybook/core-common@6.5.13_react@16.14.0_react-dom@16.14.0__react@16.14.0 in the tree', cli/npm/resolution/graph.rs:247:7
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

storybook を起動しようとした時のエラー。

```bash
deno run npm:@storybook/preact start-storybook -p 6006
error: package '@storybook/preact' did not have a bin entry for '@storybook/preact' in its package.json

Possibilities:
 * npm:@storybook/preact/build-storybook
 * npm:@storybook/preact/start-storybook
 * npm:@storybook/preact/storybook-server
```

deno で sb init すると出てくるメッセージ。

```

╭───────────────────────────────────────────────────────────────────────────────────────────────────╮
│                                                                                                   │
│   We've detected you are running npm 8.19.2                                                       │
│    which has peer dependency semantics which Storybook is incompatible with.                      │
│                                                                                                   │
│   In order to work with Storybook's package structure, you'll need to run `npm` with the          │
│   `--legacy-peer-deps=true` flag. We can generate an `.npmrc` which will do that automatically.   │
│                                                                                                   │
│   More info: https://github.com/storybookjs/storybook/issues/18298                                │
│                                                                                                   │
╰───────────────────────────────────────────────────────────────────────────────────────────────────╯
✔ Do you want to run the 'npm7' migration on your project? … no
```
