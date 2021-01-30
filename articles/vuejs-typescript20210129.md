---
title: "VuejsとTypeScriptで快適にコーティングをするTips"
emoji: "✌️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Vuejs","TypeScript"]
published: false
---

こんにちは。
現役の教員エンジニアを名乗っています。りゅーそうと申します。

普段わたしは、React.jsを使ってフロントエンドのコードを書くことが多いのですが、今回Vue.jsに初めて入門してみました。その際に心がけたことやTipsを紹介したいと思います。
何か間違っていることを言っていたらぜひ教えていただけると嬉しいです。

# やりたいこと
- Vue.jsではスタンダード(とされているような気がする)SFCを使用して書きたい。
- TypeScriptを使用してなるべく型安全なプログラミングを導入したい。
- templatesにも型の情報を元に補完を聞かせたい。

Vue3を対象に解説していきます。
使用エディタはVSCodeです。


# Veturを使用してtemplatesでも型チェックをする
早速ですが、VueのSFCではReactのJSX記法とは異なり、templates部分では型の情報を与えることができません。
Vueでの型の情報をtemplatesに与えたい場合はVSCodeのプラグインであるVeturを使用しましょう。
https://github.com/vuejs/vetur

以下の設定をVSCodeの設定ファイルに加えます

```
  "vetur.useWorkspaceDependencies": true,
  //型の情報を与えるには以下のexperimentalの設定が必要
  "vetur.experimental.templateInterpolationService": true,
```

これでReactのPropsと近いような感覚でコード補完がされるようになってとても快適にコードが書けます。

# 配列の型
言わずもがなですが、要素の追加など配列の操作でアプリケーションの機能を作成する場面は多々あると思います。
Vue.jsにおいても配列にいれる値に型をつけてコーティングをしたいですね。
そんな時にはvue.jsに内包されている`PropTypes`を使用します。

例としてUserを配列に追加するケースをあげてみます。

```
<script lang="ts">

import { defineComponent, PropType } from "vue";
import type { User } from "@/types/book";

export default defineComponent({
  props: {
    user: {
      type: Array as PropType<User[]>,
      required: true,
    },
  },
});
</script>

```
VueのSFCでTypeScriptを使用する場合はscriptタグに`lang=ts`を追加します。
配列は上記のように
