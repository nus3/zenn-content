---
title: "React + Testing Library + Jestの覚書"
emoji: "🚨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react", "jest", "test"]
published: false
---

最近、Zenn に全然(?)記事書けてないなぁっていうのと、フロントエンドのテスト大事やなぁと感じることが多かったので、React + Testing Library + Jest の覚書を雑に書くことにした

(特定の用途で覚書まとめたら、この内容だったら Zenn にも出せるやんか、とかそんなことがあったわけでは断じてない)

## JavaScript のテストフレームワークである Jest について

- Create React App(以下:CRA)ではテストランナーに Jest を採用している
  - https://create-react-app.dev/docs/running-tests
- CRA での Jest のコンフィグのベースは下記の実装を確認する
  - https://github.com/facebook/create-react-app/blob/main/packages/react-scripts/scripts/utils/createJestConfig.js
- CRA の場合はテストを実行する前のセットアップの設定を追加したい場合は`src/setupTests.js`に追加する
  - 素の Jest を使う場合は`jest.setup.js`みたいなセットアップファイルを作成し、`jest.config.js` などの config ファイルの`setupFilesAfterEnv` にセットアップファイルを指定する

### Jest での transformer の話

- Jest は実装されたコードを JavaScript として実行するが、Node.js でサポートされていない JSX, TypeScript, Vue テンプレートなどの構文を使う場合はプレーンな JavaScript にトランスパイルする必要がある
  - https://jestjs.io/ja/docs/next/code-transformation
- この Node.js でサポートされていない構文に対応するために transform 機能がある
  - CRA ではここら辺が該当の実装
  - https://github.com/facebook/create-react-app/tree/main/packages/react-scripts/config/jest
  - babel-jest + babel-preset-react-app を採用してる

この transform を利用して実装されている JSX や TypeScript をトランスパイルしている

Jest の transformer には ts-jest や babel-jest、@swc/jest、esbuild-jest などがある

### test.todo, test.each, test.only

Jest の `test` には便利なメソッドがある

- [test.todo()](https://jestjs.io/ja/docs/api#testtodoname)
  - このメソッドを使うことで、実装予定のテストを TODO として残せる
- [test.each()](https://jestjs.io/ja/docs/api#testeachtablename-fn-timeout)
  - 同じテストケースを異なるテストデータを使ってテストしたい場合に使える
  ```tsx
  test.each([
    { title: "nus3", value: "3" },
    { title: "nus4", value: "4" },
    { title: "nus5", value: "5" },
  ])(
    "$title の場合", // ${name}でtest.eachで渡したオブジェクトのプロパティの値をここでも使える
    ({ test, value }) => {
      // test.eachオブジェクトの配列分ののテストが実行できる
    }
  );
  ```
- [test.only()](https://jestjs.io/ja/docs/api#testonlyname-fn-timeout)
  - 一つのテストケースのみを実行したい場合に使える

## Testing Library について

Testing Library は実際に対象のコンポーンエントをユーザーが操作してるようなテストを書くためのテストユーティリティ

### コンポーネントのテスト方法

大まかに次の流れでコンポーネントのテストは

1. 対象のコンポーネントを render する
2. render したコンポーネントに対して、対象の要素を testing-library のクエリで取得する
3. 対象の要素に対して何かしらのユーザー操作を行う(表示確認だけだったらこのステップは省略)
4. 対象の要素が期待した状態になっているか確認する

実際に Testing Library のドキュメントに載ってる[サンプルコード](https://testing-library.com/docs/queries/about)を少し編集して上記の流れを説明すると

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { Login } from "."; // テストしたい対象のコンポーネント

test("should show login form", () => {
  render(<Login />); // 1. 対象のコンポーネントをrender

  const input = screen.getByLabelText("Username"); // 2. 対象の要素をクエリで取得
  userEvent.type(input, "nus3"); // 3. 対象の要素にユーザー操作を行う

  expect(input).toHaveValue("nus3"); // 4. 対象の要素が期待した状態になっているか確認
});
```

### クエリの種類について

詳細は次のリンクの Summary Table を見ると何が違うのかわかる

[https://testing-library.com/docs/queries/about/#types-of-queries](https://testing-library.com/docs/queries/about/#types-of-queries)

ざっくりまとめると

- get 〇〇、query 〇〇、find 〇〇では、該当する要素がある時とない時で Error を返すかどうかが違う
- 〇〇 by、〇〇 AllBy では該当の要素が複数あった場合に Error を返すかどうかが違う

### クエリの優先順位について

Testing Library の基本原則では、テストコードはなるべくユーザーが操作するような実装を推奨しており、その基本原則に則ったクエリの優先度がある

[https://testing-library.com/docs/queries/about/](https://testing-library.com/docs/queries/about/)

1. 対象の要素に誰も(マウス、支援技術、視覚的)がアクセスできるクエリ
   1. getByRole
   2. getByLabelText
   3. getByPlaceholderText
   4. getByText
   5. getByDisplayValue
2. セマンティッククエリ
   1. getByAltText
   2. getByTitle
3. テスト ID
   1. getByTestig

### @testing-library/jest-dom について

@testing-library/jest-dom を使用すると、テスト時のコンポーネント状態を確認するための便利なメソッド(カスタムマッチャー)が使えるようになる

どのようなカスタムマッチャーがあるかは次の README を読むとイメージできる

[https://github.com/testing-library/jest-dom](https://github.com/testing-library/jest-dom)

たとえば

- 対象の要素が document ないにあるかどうかを確認する `toBeInTheDocument()`
- 対象の要素が visible かどうかを確認する`toBeVisible()`
- 対象の要素に該当の属性があるかどうかを確認する`toHaveAttribute(attr: string, value?: any)`
- 対象の要素がフォーカスされているかどうかを確認する`toHaveFocus()`

などがある。

対象の要素の状態をテストしたい時に、一度この jest-dom にどんなカスタムマッチャーがあるのか確認しても良いかもしれない

### waitFor について

テスト対象のコンポーネントの表示やユーザー操作の中で非同期の実装がある場合に、await + waitFor を使うことで非同期処理を待ってくれる(waitFor に渡した callback 関数がエラーを投げなくなるまで待つようになる)

実際に waitFor の実装例を Testing Library のドキュメントのサンプルコードを参考に作ってみると次のようになる

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { Login } from ".";
import * as loginApi from "./api";

test("should submit login form", async () => {
  const mockLoginApi = jest.fn();
  jest.spyOn(loginApi, "login").mockImplementation(mockLoginApi); // submitボタンを押された際に実行されるloginのモック

  render(<Login />);

  const submitButton = screen.getByRole("button");
  userEvent.click(submitButton);

  // loginApiのloginが呼ばれるまで待つ
  await waitFor(() => expect(mockLoginAPI).toHaveBeenCalledTimes(1));

  // loginのAPIが呼ばれた後のコンポーネントの状態をテストできる
});
```

### logDOM について

logDOM という関数が@testing-library/react にはあり、テスト時に実際にクエリで取得した要素を確認することができる

```tsx
import { logDOM, render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { Login } from ".";

test("should show login form", () => {
  render(<Login />);

  const input = screen.getByLabelText("Username");
  // test実行時にscreen.getByLabelText('Username')で取得したDOMが出力される
  logDOM(input);
});
```

### userEvent について

- testing library でユーザー操作を実行する際に使用できるものに fireEvent と@testing-library/user-event がある
- [fireEvent のドキュメント](https://testing-library.com/docs/dom-testing-library/api-events/#fireevent)をみるとほとんどの場合では user-event を使うことを推奨している
- fireEvent は単純にイベントを dispatch するのに対して、user-event では実際にユーザーが操作したようなイベントを dispatch する
  - [https://ph-fritsche.github.io/blog/post/why-userevent](https://ph-fritsche.github.io/blog/post/why-userevent)
- 特段、考慮することがなければ user-event を使った方が良さそう
