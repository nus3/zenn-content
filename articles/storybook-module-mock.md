---
title: "Storybook8.1からSubpath importsを使ったモジュールのモックができるように"
emoji: "📓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["storybook"]
published: false
publication_name: "cybozu_frontend"
---

Storybook の 8.1 のリリースで、Storybook 上でコンポーネントを表示するときに特定のモジュールをモックできるようになった。

## Module mock

https://storybook.js.org/blog/type-safe-module-mocking/

https://storybook.js.org/docs/writing-stories/mocking-modules

Node.js の Subpath imports を使い、package.json の`imports`に Storybook で表示する際にモックするファイルを指定できる。モジュールを指定する際は外部のモジュールと区別できるように常に`#`を先頭につける必要がある。

```json: package.json
  "imports": {
    "#src/api/example": {
      "storybook": "./src/api/example.mock.ts",
      "default": "./src/api/example.ts"
    },
    "#*": [
      "./*",
      "./*.ts",
      "./*.tsx"
    ]
  },
```

対象のモジュールをインポートする際にも`#`をつける。

```diff tsx: GetExamplesButton.tsx
- import { type GetExamplesResponse, getExamples } from "../../api/example";
+ import { type GetExamplesResponse, getExamples } from "#src/api/example";
```

Node.js が提供する Subpath imports は Vite と Webpack、また [TypeScript5.4](https://devblogs.microsoft.com/typescript/announcing-typescript-5-4/?ref=storybookblog.ghost.io#auto-import-support-for-subpath-imports) でサポートされている。実際に次の環境で試すと、型の補完も効くし、モジュールがモックされた。tsconfig も Vite の config もいじってないのに。

- Storybook: 8.1.6
- Vite: 5.2.8
- TypeScript: 5.4.5

https://github.com/nus3/react-test-examples/pull/16

`@storybook/test`の`fn`と Story の`beforeEach`を使うことで、Jest や Vite のように Story ごとにモックが返す値も変えられる。

```ts: src/api/example.mock.ts
import type { GetExamplesRequest, GetExamplesResponse } from "#src/api/example";
import { fn } from "@storybook/test";

export const getExamples = fn(async (_params?: GetExamplesRequest) => {
  const response: GetExamplesResponse = {
    examples: [
      { id: "1", name: "nus1" },
      { id: "2", name: "nus2" },
      { id: "3", name: "nus3" },
    ],
  };
  return response;
}).mockName("getExamples");
```

```tsx: GetExamplesButton.stories.tsx
import { GetExamplesButton } from "./GetExamplesButton";
import { getExamples } from "#src/api/example.mock";

import type { Meta, StoryObj } from "@storybook/react";

const meta: Meta = {
  component: GetExamplesButton,
} satisfies Meta<typeof GetExamplesButton>;
export default meta;

type Story = StoryObj<typeof meta>;

export const ChangeMockData: Story = {
  args: {},
  beforeEach: async () => {
    getExamples.mockReturnValue(
      new Promise((resolve, _) => {
        resolve({
          examples: [
            { id: "1", name: "change-mock-data-nus1" },
            { id: "2", name: "change-mock-data-nus2" },
            { id: "3", name: "change-mock-data-nus3" },
          ],
        });
      })
    );
  },
};
```

## Storybook 上から新しい Story を追加できるように

https://storybook.js.org/blog/interactive-story-generation/

Storybook のサイドメニューや Controls パネルから新しい Story を追加できるようになった。

サイドメニューに表示されるプラスボタンを押すと、検索ダイアログが表示され、Story を追加したいコンポーネントを選択できる。
![サイドメニューから新しいStoryを追加](/images/storybook-module-mock/interactive-story1.png)

また、Controls パネルの値を変更するとフッターに変更した値で Story を更新するか、新たな Story を追加するかの選択が表示される。
![Controlsでの値の変更から新しいStoryを追加](/images/storybook-module-mock/interactive-story2.png)
