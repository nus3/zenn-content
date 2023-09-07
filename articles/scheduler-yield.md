---
title: "長いタスクを分割するscheduler.yieldという提案"
emoji: "🕰️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["webapi"]
published: true
---

## 3 行まとめ

- `scheduler.yield`が Chrome115 から origin trial で試せるように
- `scheduler.yield`を使うと長いタスクを分割できる
- `scheduler.yield`ではユーザーのインタラクション以外のタスクが割り込まない

## Long task の問題とタスクの分割

ブラウザのメインスレッドを占有するような実行時間が長いタスク(Long task)は、そのタスクが実行されている間に何かしらのインタラクションがあっても、タスクが終わるまでブラウザはインタラクションに対応できません。

![クリックがあっても長いタスクが終わるまではクリックのタスクは実行できない](/images/scheduler-yield/long-task.png)
_クリックしても長いタスクが終わるまではクリックのタスクは実行されない_

こういった長いタスクを分割することで、インタラクションなどの優先度の高いタスクに対応することができます。これは Core Web Vitals の指標である[Interaction to Next Paint(INP)](https://web.dev/inp/)の向上も期待できます。

![タスクを分割するとクリックのタスクを間に差し込める](/images/scheduler-yield/small-task.png)
_タスク分割をすることでクリックのタスクが次のタスクとして実行される_

### `async`/`await` と `setTimeout`を使ったタスクの分割

長いタスクを分割する方法として、`async`/`await` と `setTimeout`を使った方法があります。

実際にフォーム画面で submit した際の一連の長い処理を例にタスクの分割をみてみましょう。

この例では submit された際に、以下の処理を行います。

1. バリデーション
2. ローディングの表示
3. 入力内容を保存する API へ送信
4. submit 後の画面の更新
5. アナリティクスの送信

```js
function validateForm() {
  // バリデーション
}
function showSpinner() {
  // submit後のローディングの表示
}
function saveToDatabase() {
  // 入力内容を保存するAPIへ送信
}
function updateUI() {
  // submit後の画面の更新
}
function sendAnalytics() {
  // アナリティクスの送信
}

function handleSubmit() {
  validateForm();
  showSpinner();
  saveToDatabase();
  updateUI();
  sendAnalytics();
}
```

タスクの実行状況は Chrome DevTools の Performance タブ上で確認することもできます。

![Chrome DevTools上に表示されるLong task](/images/scheduler-yield/long-task-dev-tools.png)

この長いタスクを `async`/`await` と `setTimeout`を使って分割してみましょう。

```js
function yieldToMain() {
  return new Promise((resolve) => {
    setTimeout(resolve, 0);
  });
}

async function handleSubmit() {
  const tasks = [
    validateForm,
    showSpinner,
    saveToDatabase,
    updateUI,
    sendAnalytics,
  ];

  for (const task of tasks) {
    task();
    await yieldToMain();
  }
}
```

それぞれの処理(関数)を配列に格納し、`for`文で順番に実行しています。`for`文の中では、関数が実行された後に`yieldToMain`を`await`します。`yieldToMain`では、`setTimeout`を使って 0ms 後に`resolve`する`Promise`を返しています。

`async`/`await` と `setTimeout`を使うことで、タスクが Chrome 上でどのように実行されているのか確認してみましょう。

![Chrome DevTools上に表示される分割されたタスク](/images/scheduler-yield/small-task-dev-tools.png)

Performance タブを見ると関数ごとにタスクが分割されていることがわかります。

さらに分割されたタスクを見てみると、最初の関数である`validateForm`は Click 時に実行され、それ以降の関数は`setTimeout`によるタイマー起動時(Timer Fired)に Microtask として実行されています。

### 長いタスクが分割される仕組み

`async`/`await` と `setTimeout`を使うと、なぜタスクが分割されるのか順番に見ていきましょう。

:::message
以降の流れは、自分の理解のために作りました。内容が間違っている場合は教えていただけると大変喜びます。
:::

まず、フォームが submit された際に`handleSubmit`が実行されます。`handleSubmit`の中では最初の関数である`validateForm`が実行されます。

![step1](/images/scheduler-yield/step-1.png =500x)

次に、`handleSubmit`は`yieldToMain`が実行され、`setTimeout`によって 0ms 後に`resolve`する`Promise`が返されます。この時、`setTimeout`のコールバックの処理は Task Queue に入れられます。

![step2](/images/scheduler-yield/step-2.png =500x)

`yieldToMain`は`await`しているので、返ってくる`Promise`が完了するまで`handleSubmit`の処理は中断されます。

![step3](/images/scheduler-yield/step-3.png =500x)

次のタスクとして、Task Queue にある`setTimeout`のコールバックの処理が実行されます。

![step4](/images/scheduler-yield/step-4.png =500x)

`setTimeout`のコールバックの処理が終わると、`yieldToMain`の`Promise`が`resolve`されるので、`handleSubmit`の後続の処理が再開されます。`for`文の中では次の関数である`showSpinner`と`yieldToMain`が Microtask Queue に入れられます。

![step5](/images/scheduler-yield/step-5.png =500x)

Microtask Queue に入れられた`showSpinner`と`yieldToMain`が実行されます。

![step6](/images/scheduler-yield/step-6.png =500x)

`yieldToMain`が実行され、`setTimeout`のコールバックの処理は Task Queue に入れられます。(以降、関数分、この流れが繰り返される)

![step7](/images/scheduler-yield/step-7.png =500x)

### `async`/`await`と`setTimeout`を使った際の問題点

`async`/`await`と`setTimeout`を使って長いタスクを分割した場合、ユーザーのインタラクション以外のタスクが、分割したタスクの間に割り込んでしまうことがあります。

例えば、サードパーティのスクリプトが`setInterval`を使い、一定の間隔で何かしらの処理をしていた場合などが挙げられます。実際に`setInterval`を実行中に`async`/`await`と`setTimeout`を使ったタスク分割を行うと`setInterval`のタスクが間に割り込んでしまうのが Performance タブで確認できます。

![長いタスクを分割した際にサードパーティのスクリプト等のタスクが間に割り込んでしまう](/images/scheduler-yield/set-interval.png)

## `scheduler.yield`とは

前置きが長くなりましたが、今回紹介する`scheduler.yield`を使うと、長いタスクを分割する際にユーザーのインタラクション以外のタスクが、分割したタスクの間に割り込まないようにできます。

https://developer.chrome.com/blog/introducing-scheduler-yield-origin-trial/

`scheduler.yield`は Chrome115 以降でフラグか origin trial で試せます。

使い方も簡単で、前述したサンプルコードの`yieldToMain`を`scheduler.yield`に置き換えるだけです。

```diff js
  for (const task of tasks) {
    task();
-   await yieldToMain();
+   await scheduler.yield();
  }
```

実際に`setInterval`を実行中に`scheduler.yield`を使ったタスク分割を行うと、`setInterval`のタスクに割り込まれません。

![scheduler.yieldを使ったタスク分割では`setInterval`のタスクが割り込まない](/images/scheduler-yield/scheduler-yield.png)

## DEMO

Chrome115 以上で origin trial のトークンが有効な間は、以下のリンクで`scheduler.yield`の動作を確認できます。

https://nus3.github.io/ui-labs/scheduler-yield/

Start ボタンを押すと、`setInterval`のタスクが一定間隔で実行します。その後、yielding setTimeout のボタンを押すと`async`/`await`と`setTimeout`を使ったタスク分割が実行され、`setInterval`のタスクが間に割り込んでくるのを確認できます。

![async、awaitとsetTimeoutによるタスク分割のデモ](/images/scheduler-yield/demo-settimeout.png =400x)

また、Start ボタンを押してから yielding scheduler.yield のボタンを押すと、`scheduler.yield`を使ったタスク分割が実行され、`setInterval`のタスクが割り込まないことを確認できます。

![scheduler.yieldによるタスク分割のデモ](/images/scheduler-yield/demo-scheduler-yield.png =400x)

ソースコードは以下のリポジトリで公開しています。

https://github.com/nus3/ui-labs/tree/main/src/scheduler-yield

## あとがき

`scheduler.yield`は、オリジントライアルかフラグ(`chrome://flags/`)を設定することで Chrome 上で気軽に使えます。また、[フィードバック](https://github.com/wicg/scheduling-apis/issues)も募集しているようなので、興味がある人はぜひ試してみてください。
