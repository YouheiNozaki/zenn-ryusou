---
title: "VuejsとTypeScriptで快適にコーティングをするTips"
emoji: "✌️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Vuejs","TypeScript"]
published: true
---

こんにちは。
現役の教員エンジニアを名乗っています。りゅーそうと申します。

普段わたしは、React.jsを使ってフロントエンドのコードを書くことが多いのですが、今回Vue.jsに初めて入門してみました。その際に心がけたことやTipsを紹介したいと思います。
何か間違っていることを書いていたらぜひ教えていただけると嬉しいです。

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

```js
<script lang="ts">

import { defineComponent, PropType } from "vue";
import type { User } from "@/types/user";

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
またTypeScriptでコンポーネントを作成するには`defineComponent`で書いていきます。

配列に型をつける際には上記のコードのようにvue.jsの`PropType`を使用します。
TypeScriptのType Assertionを使用して配列のデータに型をつけていきます。
これで、型の情報がtemplatesにも渡るようになり、参照しながらコードを書くことができます。

型の`required`は値がundefinedかをチェックすることができます。

ちなみにObjectの場合は
`type: Object as PropType<User>,`
とすることで型をつけることができます。

## 配列(reactiveの場合)の型

Vue.jsではコンポーネントに状態を保つために、reactiveという関数があります。
ReactでいうところのuseState,Vue2.0でいうところの`data:`に与えていたように初期値を設定し、状態管理を行うことができます。
reactiveにおいても配列で状態をもって置く場面が多々ありますが、その際には以下のように書きます。

```js
    const state = reactive({
      users: [] as User[],
    });
```

配列の型をType Assertionで与えてあげるようにすると関数内やtemplatesで状態を参照する際に型の補完を得ながらコーティングできるようになります。

# Data Fetching

これはVueに限った話ではないですが、外部APIなどとの通信を行う際には型をつけましょう。
有名なDate Fetchingライブラリaxiosの例をあげてみます。

VueのcompositionAPIにおいては以下のような関数はsetup()の中に記述します。
https://v3.ja.vuejs.org/guide/composition-api-introduction.html#%E3%81%AA%E3%81%9B%E3%82%99%E3%82%B3%E3%83%B3%E3%83%9B%E3%82%9A%E3%82%B7%E3%82%99%E3%82%B7%E3%83%A7%E3%83%B3-api-%E3%81%AA%E3%81%AE%E3%81%8B

```js
    const FetchingUser = async () => {
      const response = await axios.get<Users>(
        `${baseUrl}`
      );

      return (state.users = response.data.items);
    };
```

上記のようにaxiosに型を与えてあげましょう。reactiveで参照した方と一致するようにコードを書くと良いと思います。
axiosのresponse.dataはany型なので自分で型の参照を与えてあげると良いでしょう。
標準のFetchメソッドを使用する場合は自身で型を拡張したFetch関数を実装する必要があるかと思います。

まだ試せてはいないのですが、このほかに気になっている型安全にData Fetchingをおこなうための関連ライブラリをまとめておきます。
https://github.com/aspida/aspida
https://apollo.vuejs.org/
https://github.com/Kong/swrv


## 最後に
Vueの2は触れたことがないのですが、Vue3とcompositionAPIによって比較的Reactのような書き味でコードをかけるなあという印象を受けました。
Vue.jsは`required`を与えてあげることでundefinedの値をチェックしたり、実装すると地味に手間がかかる部分を気軽に処理を行えることがメリットのように感じました。
一方、そのあたりの処理を明示的に、厳密にチェックしていきたいというケースはReactの方がすぐれているようにも感じました。JSXはやはり便利ですね。
また、現状templatesの型の参照はVSCodeのプラグインに依存しているので、その辺りで好き嫌いが分かれるのかもしれません。
VueにもJSXで書けるオプションがあるみたいなので、いつか試してみたいです。

そして、まとめた後にほぼほぼドキュメントの参照に終わってしまっていることに気がつきました。
ぜひ、ドキュメントも合わせてチェックしてみてください。
https://v3.ja.vuejs.org/

またTipsが溜まっていったら追記していこうと思います。



