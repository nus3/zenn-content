---
title: "フロントエンドのモジュールを共有する手法を考える"
emoji: "📑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["frontend"]
published: false
publication_name: "cybozu_frontend"
---

この記事は、[CYBOZU SUMMER BLOG FES '25](https://cybozu.github.io/summer-blog-fes-2025/) の記事です。

## はじめに

サイボウズの[kintone](https://kintone.cybozu.co.jp/)というプロダクトでは、2021 年からフロントエンド刷新を進めています。

https://blog.cybozu.io/entry/2022/02/04/171154

このフロントエンド刷新では「フロントエンドに Autonomy をもたらす」ことを目指して、次のような取組も行なっています。

- モノリスな構成からの脱却
- アーキテクチャとチーム（組織）体制をセットで考える
- チーム単位の Monorepo によりオーナーシップを明確に

フロントエンド刷新によって kintone の領域ごとにフロントエンドは分割され、現状だと次の図のように、`frontend`ディレクトリ配下には各チームが独立した Monorepo 構成のディレクトリを持っています。

![frontendディレクトリ配下に各チーム単位でmonorepo構成になっている](/images/module-share-pattern/directory.png =300x)

上記のような構成で運用を続けていくことで、各チームがオーナーシップを持ってフロントエンドの開発を進められるようになりました。一方で、各チーム・領域を跨いだモジュール（UI コンポーネントや関数など）の共有のしづらさが課題になってきました。

この記事では、1 プロダクトに複数の開発チームがある場合のフロントエンドでのモジュールの共有パターンを洗い出し、それぞれ比較してみます。

## フロントエンドのモジュールを共有する方法を考えてみる

今回は大まかに次の 4 つのパターンを考えてみました。他にも、こんな方法があるよってものがあれば、ぜひ教えてください。

- npm パッケージとして npm registry に公開する
- フロントエンドを 1 つの Monorepo にまとめる
- 共通のロジックやコンポーネントを直接扱う
- マイクロフロントエンドを採用する
  - Module Federation
  - qiankun

それぞれのパターンについて詳細を見ていきます。

### npm パッケージとして npm registry に公開する

[npm の公式レジストリ](https://www.npmjs.com/)や[GitHub Packages](https://docs.github.com/ja/packages/learn-github-packages/introduction-to-github-packages)などの npm registry に npm パッケージを public・private に公開することで、各チームがそれをインストールして利用できます。

この方式の良い点としては、npm パッケージのバージョニングによってモジュール開発側は利用側の影響を考慮せずに、独立して開発できる点が挙げられます。

一方で、実際にこの方式で運用していく中で、次のような点に注意が必要です。

- npm パッケージのオーナーを明確にする
- 任意のバージョンを全チームに反映するためには、利用チーム側の協力が必要
- 破壊的変更があった場合の利用チームへのサポート
- npm パッケージを公開するまでの仕組みの自動化
- npm パッケージ利用チームがコントリビューションしやすい仕組み
- 1 つのプロダクト内で別パージョンが使われてることを許容できるかどうか

特に継続して npm パッケージを運用していくためには、兼務メンバーや当番制などでのメンテナンスはせずに、専任のメンバーやチームを設けることが重要だと感じています。

### フロントエンドを 1 つの Monorepo にまとめる

[npm](https://docs.npmjs.com/cli/v8/using-npm/workspaces)や [yarn](https://yarnpkg.com/features/workspaces)、[pnpm](https://pnpm.io/workspaces) などのパッケージマネージャーや[Deno](https://docs.deno.com/runtime/fundamentals/workspaces/)のようなランタイムが提供するワークスペース機能を利用して、1 つの Monorepo 構成を作る方式です。

冒頭に紹介したように複数のチームが 1 つの Monorepo 構成の中に存在する例として次の図のような構成が作れます。

![frontendディレクトリ配下が1つのmonorepo構成になっている](/images/module-share-pattern/monorepo.png =300x)

`applications`ディレクトリに各チームごとのディレクトリを作り、`packages`ディレクトリに共通で利用するモジュールを配置するイメージです。

このような構成は、例えば pnpm であれば、次のように`pnpm-workspace.yaml`で Monorepo 内で扱うパッケージ（ディレクトリ）を指定し

```yaml: pnpm-workspace.yaml
packages:
  - applications/*
  - packages/*
```

各チームのディレクトリの`package.json`で`workspace:*`を指定することで、Monorepo 内のローカル npm パッケージを参照することができます。

```json: applications/team-a/package.json
{
  "name": "team-a",
  "dependencies": {
    "shared-functions": "workspace:*"
  }
}
```

```json: packages/shared-functions/package.json
{
  "name": "shared-functions",
}
```

この方式の良い点としては次のようなものが考えられます。

- Monorepo 内では、ローカル npm パッケージの作成・共有が簡単
  - ESLint や tsconfig などの設定ファイルも共有しやすい
- ローカル npm パッケージではバージョニング管理が不要

一方で気になるポイントとしては、npm パッケージの依存管理に使われる lock ファイルは一つに集約されるので、各チームが依存する npm パッケージを追加、更新した場合にコンフリクトが発生します。チームで頻繁に依存パッケージの追加、更新がする際にコンフリクトがどの程度発生するのか、また、このコンフリクトにより各チームで独立したフロントエンド開発を阻害されないかという点です。

この点に対して、例えば pnpm の場合、lock ファイルのコンフリクトは手動で解消せずに、[`pnpm install`を実行することで自動で解消したり](https://pnpm.io/git#merge-conflicts)、[Git Branch Lockfiles](https://pnpm.io/ja/git_branch_lockfiles)といった機能を使うことでブランチごとに lock ファイルを分けることで、開発時にはコンフリクト回避し、別のタイミングで分けた lock ファイルをマージすることが可能なようです。

このような仕組みを使うことで、1 つの Monorepo 構成であっても、各チームが独立してフロントエンド開発を進められるかもしれません。

### 共通のロジックやコンポーネントを直接扱う

### マイクロフロントエンドを採用する

## まとめ

<!-- TODO: 文章化する -->

- pnpm を使った 1 つの Monorepo 構成は導入を検討してみても良さそう
