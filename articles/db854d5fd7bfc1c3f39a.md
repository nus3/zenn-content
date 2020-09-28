---
title: "Nuxt + TypeScript + Vuetify モジュールに Storybook をいれる"
emoji: "🤠"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nuxtjs", "typescript", "vuetify"]
published: false
---

## 概要

Nuxt + Vuetify に Storybook いれる記事はいくつかあったが、Nuxt + Vuetify モジュールに Storybook 入れる方法があまりなかったので残しとく
zenn での初投稿のテストも兼ねて

## 環境

```json
"nuxt": "2.12.2",
"ts-node": "8.10.1",
"vee-validate": "3.3.0",
"@nuxtjs/vuetify": "1.11.2",
"@storybook/vue": "5.3.19",
"babel-preset-vue": "2.0.2",
```

## Storybook のインストール

Manual setup にした
https://storybook.js.org/docs/guides/guide-vue/

```bash
npm i -D @storybook/vue babel-preset-vue
```

babel-preset-vue がいるようだ

## Storybook の config

vue ファイル内の TypeScript、scss、alias 解消のために webpack の config を書き換える

`.storybook/webpack.config.js`

```javascript
const path = require("path");

module.exports = ({ config }) => {
  config.module.rules.push({
    test: /\.ts$/,
    exclude: /node_modules/,
    use: [
      {
        loader: "ts-loader",
        options: {
          appendTsSuffixTo: [/\.vue$/],
          transpileOnly: true,
        },
      },
    ],
  });
  config.module.rules.push({
    test: /\.s(a|c)ss$/,
    use: ["style-loader", "css-loader", "sass-loader"],
  });
  config.resolve.alias = {
    vue: "vue/dist/vue.esm.js",
    "@": path.resolve(__dirname, "../src"),
    "~": path.resolve(__dirname, "../src"),
  };

  return config;
};
```

全ページ共通の scss やアイコンとかの読み込み
これは`preview-head.html`のがよかったかも

`.storybook/Decorator.vue`

```js
<template>
  <div class="decorator">
    <slot name="story"></slot>
  </div>
</template>

<script>
export default {
  name: "Decorator",
};
</script>

<style lang="scss">
@import "@/assets/scss/app.scss";
@import "https://fonts.googleapis.com/css?family=Lato:400,700|Noto+Sans+JP:400,700&display=swap";
@import "https://fonts.googleapis.com/css?family=Material+Icons";
</style>
```

Vuetify や vee-validate が Storybook 上でも描画できるように読み込ませる
`.storybook/config.js`

```js
import { configure, addDecorator } from "@storybook/vue";
import Decorator from "./Decorator.vue";

import Vue from "vue";
import Vuetify from "vuetify";
import "vuetify/dist/vuetify.css";
import {
  extend,
  localize,
  ValidationObserver,
  ValidationProvider,
} from "vee-validate";
import {
  confirmed,
  digits,
  email,
  image,
  length,
  max,
  numeric,
  regex,
  required,
  size,
} from "vee-validate/dist/rules";
import ja from "vee-validate/dist/locale/ja.json";

Vue.use(Vuetify);

// vee-validate用のコンポーネント読み込み
Vue.component("ValidationObserver", ValidationObserver);
Vue.component("ValidationProvider", ValidationProvider);

// validationのルールも必要であれば読み込み
extend("confirmed", confirmed);
extend("digits", digits);
extend("email", email);
extend("image", image);
extend("length", length);
extend("max", max);
extend("numeric", numeric);
extend("regex", regex);
extend("required", required);
extend("size", size);
localize("ja", ja);

addDecorator((story) => ({
  // 共通のscssやフォントを読み込んだコンポーネント
  components: { Decorator },
  vuetify: new Vuetify({
    // nuxt.config.tsでテーマとか設定してたらここに書く
    preset: {
      icons: {
        iconfont: "md",
      },
      theme: {
        dark: false,
        themes: {
          light: {
            primary: "#00A381",
            sidebar: "#F7FAF9",
            accountSidebar: "#333333",
            copyright: "#798181",
            menuBg: "#FFFFFF",
            loginBg: "#F7FAF9",
            error: "#F75F4F",
            success: "#00A381",
            disable: "#CCCCCC",
            lineBtn: "#00C300",
          },
        },
      },
    },
  }),
  template: `
    <decorator>
      <v-app slot="story">
        <v-content>
          <story/>
        </v-content>
      </v-app>
    </decorator>
  `,
}));

configure(require.context("../src/", true, /\.stories\.ts$/), module);
```

Storybook の実装

```typescript
import { storiesOf } from "@storybook/vue";

import RequireBadge from "./RequireBadge.vue";

storiesOf("atoms/badge/RequireBadge", module).add("default", () => ({
  components: { RequireBadge },
  template: `
      <require-badge />
    `,
}));
```

## まとめ

nuxt の vuetify モジュールではなく、普通に vuetify インストールして plugin で設定しておけば storybook の config と nuxt.config.ts とで二重に theme とかメンテしなくてよかったのかとか思ったり
