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

この記事では、マイクロフロントエンドを採用するまでではないですが、1 プロダクトに複数の開発チームがある場合のフロントエンドでのモジュールの共有パターンを洗い出し、それぞれ比較してみます。

## フロントエンドのモジュールを共有する方法を考えてみる

今回は大まかに次の 3 つのパターンを考えてみました。他にも、こんな方法があるよってものがあれば、ぜひ教えてください。

- npm パッケージとして npm registry に公開する
- フロントエンドを 1 つの Monorepo にまとめる
- 共通のロジックやコンポーネントを直接扱う

それぞれのパターンについて詳細を見ていきます。

### npm パッケージとして npm registry に公開する

[npm の公式レジストリ](https://www.npmjs.com/)や[GitHub Packages](https://docs.github.com/ja/packages/learn-github-packages/introduction-to-github-packages)などの npm registry に npm パッケージを public・private に公開することで、各チームがそれをインストールして利用できます。

この方式の良い点としては、npm パッケージのバージョニングによってモジュール開発側は利用側の影響を考慮せずに、独立して開発できる点が挙げられます。実際に幾つかのモジュールではこの方式を採用し private な npm パッケージとして公開し、各チームで利用されています。

一方で、この方式で運用していく中で、次のような点に注意が必要だと感じています。

- npm パッケージのオーナーを明確にする
- 任意のバージョンを全チームに反映するためには、利用チーム側の協力が必要
- 破壊的変更があった場合の利用チームへのサポート
- npm パッケージを公開するまでの仕組みの自動化
- npm パッケージ利用チームがコントリビューションしやすい仕組み
- 1 つのプロダクト内で別パージョンが使われてることを許容できるかどうか

特に継続して npm パッケージを運用していくためには、兼務メンバーや当番制などのオーナーが明確じゃない状態にはしないことが重要です。オーナーが曖昧だと、メンテナンスが滞り、継続して運用していくことが難しくなります（ました）。

### フロントエンドを 1 つの Monorepo にまとめる

[npm](https://docs.npmjs.com/cli/v8/using-npm/workspaces)や [yarn](https://yarnpkg.com/features/workspaces)、[pnpm](https://pnpm.io/workspaces) などのパッケージマネージャーや[Deno](https://docs.deno.com/runtime/fundamentals/workspaces/)のようなランタイムが提供するワークスペース機能を利用して、1 つの Monorepo 構成を作る方式です。

冒頭に紹介した図のように frontend ディレクトリ配下を一つの Monorepo にまとめると次の図のような構成が作れます。

![frontendディレクトリ配下が1つのmonorepo構成になっている](/images/module-share-pattern/monorepo.png =300x)

この例では`applications`ディレクトリに各チームごとのディレクトリを作り、`packages`ディレクトリに共通で利用するモジュールを配置するイメージです。

このような構成は、例えば pnpm であれば、次のように`pnpm-workspace.yaml`で Monorepo 内で扱うパッケージ（ディレクトリ）を指定し、

```yaml: pnpm-workspace.yaml
packages:
  - applications/*
  - packages/*
```

各チームのディレクトリの`package.json`で`workspace:*`を指定することで、Monorepo 内では例えば`packages/ui-components`ディレクトリを npm パッケージとして直接、参照することができます。

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

次の点がこの方式の良い点だと考えています。

- Monorepo 内では、ローカル npm パッケージの作成・共有が簡単
  - ESLint や tsconfig などの設定ファイルも共有しやすい
- ローカル npm パッケージではバージョニング管理が不要

一方で気になる点としては、npm パッケージの依存管理に使われる lock ファイルは一つに集約されるので、各チームで頻繁に依存パッケージの追加、更新をする際にコンフリクトがどの程度発生するのか、また、このコンフリクトにより各チームで独立したフロントエンド開発が阻害されないかという点です。

この点に対して、例えば pnpm の場合、lock ファイルのコンフリクトは手動で解消せずに、[`pnpm install`を実行することで自動で解消したり](https://pnpm.io/git#merge-conflicts)、[Git Branch Lockfiles](https://pnpm.io/ja/git_branch_lockfiles)といった機能を使うことでブランチごとに lock ファイルを分けることで、開発時にはコンフリクト回避し、別のタイミングで分けた lock ファイルをマージすることが可能なようです。

このような仕組みを使うことで、1 つの Monorepo 構成であっても、各チームが独立してフロントエンド開発を進められるかもしれません。

### 共通のロジックやコンポーネントを直接扱う

特定のディレクトリにあるモジュールを各チームが直接参照する方式です。

例えば、次のようなディレクトリ構成で shared ディレクトリに各チームが参照する共通モジュールを配置する場合、

```
├── shared/
│   ├── package.json
│   ├── tsconfig.json
│   └── src/
│       ├── components/
│       │   └── Button.tsx
│       └── functions/
└── team-a/
    ├── package.json
    ├── vite.config.ts
    └── tsconfig.json
```

Vite の[`resolve.alias`](https://ja.vite.dev/config/shared-options.html#resolve-alias)オプションを利用してエイリアスを設定し、

```ts: team-a/vite.config.ts
import { defineConfig } from 'vite';
import { fileURLToPath, URL } from 'node:url';

export default defineConfig({
  resolve: {
    alias: {
      // sharedディレクトリのaliasを設定
      '@shared': fileURLToPath(new URL('../shared-modules', import.meta.url)),
    },
  },
});
```

各チームは shared ディレクトリで定義されたモジュールを直接インポートします。

```ts: team-a/src/index.ts
import { formatDate } from '@shared/functions';
```

この方式では、利用チーム側では共通モジュールを npm パッケージとして扱わない点が、Monorepo 構成と異なる点です。

Monorepo 構成でなくても、この方式を利用できるのは良い点かもしれませんが、運用していく中で npm パッケージとして公開したくなった場合に、Monorepo 構成と比べて、npm パッケージ化するためのコストが高くなる点は注意が必要です。

また、個人的な感想にはなりますが、各チームが共有で利用するモジュールなのであれば、npm の仕組みに則り、npm パッケージの形で提供される方が中長期の運用を見据えるのであれば良いように感じています。

## 最後に

今回のように 1 プロダクトの中で複数チームがフロントエンド開発を行う場合に、各チーム・領域を跨いだモジュールの共有をどうするかはまだ自分の中でもはっきりとした答えは出ていません。うちではこんな事例があるよ、というのがあればぜひ教えてください。

また、方針が決まり、無事に導入・運用ができたら、その過程を [Cybozu Inside Out](https://blog.cybozu.io/) にでも書きたいと思います（宣言駆動）。
