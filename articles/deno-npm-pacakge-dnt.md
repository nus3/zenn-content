---
title: "Deno + dntで簡単にnpmパッケージを作ろう"
emoji: "🦖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["deno", "npm"]
published: true
publication_name: "cybozu_frontend"
---

## 3 行まとめ

- CJS、ESM に対応した npm パッケージが Deno + `dnt`で簡単に作成できる
- Deno で開発できるので、Lint、Format、Test、TypeCheck が設定なしですぐに使える
- `dnt`で作成した CJS、ESM のファイルに対して、それぞれ Node.js でもテストを実行してくれる

## Deno のモジュールを npm パッケージに変換する`dnt`

https://deno.com/blog/publish-esm-cjs-module-dnt

Deno の[`dnt`](https://github.com/denoland/dnt)モジュールを使うと、Deno で実装したモジュールを CommonJS(CJS) と ESM、TypeScript に対応した npm パッケージとして公開できます。

実際、今回 Deno と`dnt`を試してみましたが、Deno には Linter、Formatter、Test などが組み込まれているので、すぐに npm パッケージの作成に取り掛かれるのが良かったです。また、CICD の自動化(GitHub Actions)もとてもシンプルに実装できた印象がありました。

また、`dnt`ではビルド時に生成された CJS、ESM のファイルに対して、それぞれ Node.js でテストを実行してくれたりもします。

## 実際にモジュールを npm に公開する

実際に Deno と`dnt`を使って、npm パッケージを作成してみましょう。

次のリポジトリに今回実装したコードがあります。

https://github.com/nus3/deno-npm-package

### Deno でモジュールを実装

まず、[`cowsay`](https://www.npmjs.com/package/cowsay)を使ったシンプルなモジュールを作ります。

```ts: mod/index.ts
import cowsay from "npm:cowsay@1.5.0";

export function cowsayNus3() {
  return cowsay.say({ text: "Hello nus3!" });
}
```

このモジュールでは、[`npm:`specifiers](https://deno.land/manual@v1.36.0/node/npm_specifiers)を使って`cowsay`パッケージをインポートしています。`npm:`specifiers や[Skypack](https://www.skypack.dev/)、[esm.sh](https://esm.sh/)経由で使用されるパッケージは、`dnt`によってビルド後の`pacakge.json`の依存関係に追加されます。

```json:npm/package.json
{
  "dependencies": {
    "cowsay": "1.5.0"
  }
}
```

### Deno でテストを実装

Deno で作成したモジュールに対するテストを追加します。

```ts: mod/index_test.ts
import { assertStringIncludes } from "https://deno.land/std@0.192.0/testing/asserts.ts";
import { cowsayNus3 } from "./index.ts";

Deno.test("Cow should say Hello nus3!", () => {
  assertStringIncludes(cowsayNus3(), "Hello nus3!");
});
```

### ビルドして npm パッケージを作成

次に、`dnt`を使って Deno で実装したモジュールを npm パッケージとしてビルドします。今回はルート直下に`build_npm.ts`というファイルを作成し、そこに`dnt`の設定を追加します。

```ts: build_npm.ts
import { build, emptyDir } from "https://deno.land/x/dnt@0.37.0/mod.ts";

await emptyDir("./npm");

await build({
  entryPoints: ["./mod/index.ts"],
  outDir: "./npm",
  shims: {
    deno: true,
  },
  package: {
    name: "@nus3/cowsay-nus3",
    version: Deno.args[0],
    description: "String function that returns where the cow says Hello nus3",
    license: "MIT",
    repository: {
      type: "git",
      url: "git+https://github.com/nus3/deno-npm-package.git",
    },
    bugs: {
      url: "https://github.com/nus3/deno-npm-package/issues",
    },
  },
  postBuild() {
    Deno.copyFileSync("LICENSE", "npm/LICENSE");
    Deno.copyFileSync("README.md", "npm/README.md");
  },
});
```

作成したファイルを`deno run -A build_npm.ts {version}`のように Deno で実行することで、対象のモジュールが npm パッケージとしてビルドされ、今回の場合、`npm`ディレクトリに出力されます。

```sh
❯ deno run -A build_npm.ts 0.0.1
[dnt] Transforming...
[dnt] Running npm install...

added 47 packages, and audited 48 packages in 5s

3 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
[dnt] Building project...
[dnt] Type checking ESM...
[dnt] Emitting ESM package...
[dnt] Emitting script package...
[dnt] Running post build action...
[dnt] Running tests...

> @nus3/cowsay-nus3@0.0.1 test
> node test_runner.js

Running tests in ./script/index_test.js...

test Cow should say Hello nus3! ... ok

Running tests in ./esm/index_test.js...

test Cow should say Hello nus3! ... ok
[dnt] Complete!
```

実行時のログを見るとわかりますが、`dnt`ではビルドで生成した ESM、CJS 両方の形式のファイルに対して、Deno で書いたテストを Node.js で実行してくれます。

サンプルリポジトリでは、[`example`配下](https://github.com/nus3/deno-npm-package/tree/main/example)で公開したパッケージを`require`、`import` で呼び出せるか確認できるようにしてあります。

### GitHub Action で CICD を整える

[前述した記事](https://deno.com/blog/publish-esm-cjs-module-dnt#automate-with-github-actions)に GitHub 上で Release を作成することで npm に公開する GitHub Actions が紹介されています。

ただ、せっかくなので今回は合わせて、Lint、Format、Test、TypeCheck が GitHub Actions 上で実行されるようにしましょう。

```yaml
name: CI
on:
  pull_request:

jobs:
  ci:
    name: Check lint and test
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
      - name: Setup Deno
        uses: denoland/setup-deno@v1
        with:
          deno-version: v1.x
      - name: Lint
        run: deno lint
      - name: Test
        run: deno test -A
      - name: Check format
        run: deno fmt --check
      - name: Type check
        run: deno check mod/**.ts
      - name: Build
        run: deno run -A build_npm.ts
```

Deno を実行できる環境を用意するだけで Lint、Format、Test、TypeCheck が使えるので、とてもシンプルに実装することができて良きですね。

## あとがき

最後の GitHub Actions が特に顕著ですが、Deno をインストールすることで開発に必要なエコシステムが用意された状態で、tsconfig などの設定も要らず、すぐに開発に着手できるのがとても良い点だと感じました。

さらに、今回紹介した`dnt`を使うことで、npm パッケージを作成する上で考慮しなければいけない点を`dnt`が担ってくれます。

もちろん、2023/08/09 現在、`dnt`のバージョンは 0.38.0 でまだメジャーバージョンではない点や、使用したい npm パッケージがまだ Deno ではサポートされていない可能性もあります。しかし、Deno はバージョンを上げるごとに Node.js との互換性がどんどん上がっているので、npm パッケージを作成する際に Deno + `dnt`を選択肢の 1 つとして試してみるのはいかがでしょうか。
