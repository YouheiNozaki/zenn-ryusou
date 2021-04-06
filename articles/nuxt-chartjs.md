---
title: "nuxt.jsでChartグラフを作るチュートリアル"
emoji: "📊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vuejs", "chartjs"]
published: true
---

こんにちは。[りゅーそう](https://twitter.com/ryusou_mtkh)です。

開発に関わらさせていただいているPARKLoTさんで、Chartグラフを作成する機会がありました。

https://id.park-lot.com/

今回はVue.js(nuxt.js)を使ってChartグラフを作る方法を紹介します。
この記事では以下のような(ちょっぴりカッコよさげな)グラフを作っていきます。


![](https://storage.googleapis.com/zenn-user-upload/z55he2kgl21l52z7shyt3c628aez)

# この記事を通して学べること
- axios-mock-serverを使用して、axiosでmockAPIを叩く方法
- nuxt.jsのpageコンポーネントでparamsを取得して使用する方法
- Chart.js(vue-chartjs)を使用してChartを作成する方法
- Chart.jsで作成したグラフをカスタマイズする方法

nuxt.jsの開発環境の構築やVue3/compositionAPIなどは扱いません。
(chart.jsがVue3に対応していないため)

# 使用ライブラリなど
関連するライブラリ等
```
  "nuxt": "^2.15.4",
  "vue": "^2.6.12",
  "vue-chartjs": "^3.5.1",
  "chart.js": "^3.0.1",
  "@nuxtjs/axios": "^5.3.6",
  "dayjs": "^1.10.4",
```

# axios-mock-serverでAPIのmockを作成する
nuxt.jsの開発環境が構築されている前提で説明します。
今回グラフに表示させるデータですが、バックエンドのAPIを叩いて表示させても良いのですが、フロントエンドを先に実装したい。技術的な調査を進めたい場合には仮のデータであるmockを作成して、フロントエンド部分のみで開発を進めるのがおすすめです。
nuxt.jsでmockを作成する際はaxios-mock-serverを使用すると簡単にできるのでおすすめです(axiosを利用することが前提にはなってしまいますが)。

https://www.npmjs.com/package/axios-mock-server

設定等は以下のページを参考に進めると良いと思います。
https://qiita.com/m_mitsuhide/items/b8e073cba0dae5af2359

まずはインストールします。
```
npm install --save-dev axios-mock-server
```

nuxt.jsのpluginsにmockを登録(これからmockディレクトリは作成します)
Nuxtpluginを設定すると各コンポーネントで使用できるようになります。
```javascript:plugins/mock.js
import mock from '~/mocks/$mock.js' // ビルド時に自動生成されるファイル

export default ({ $axios }) => mock($axios)
```
nuxt.config.jsに登録します。
```js:nuxt.config.js
  plugins: [
    { src: '~/plugins/mock.js' }
  ],
```

package.jsonにスクリプトを追加します。
開発中は基本的にdevコマンドを叩いて開発することになると思います。
```
  "mock:dev": "axios-mock-server -b && nuxt",
  "mock:build": "axios-mock-server -b && nuxt build",
  "mock:start": "axios-mock-server -b && nuxt start",
  "mock:generate": "axios-mock-server -b && nuxt generate"
```

mockディレクトリにAPIのmockを作成していきます。
axios-mock-serverはnuxt.jsのpages(ファイルシステムルーティング)と同じように機能します。
https://ja.nuxtjs.org/docs/2.x/features/file-system-routing
例えば、
`mocks/promoters/_promoterId.js`というディレクトリ構成でファイルを作成すると
`/promoters/:id`というAPIが作成されるといった具合です。
(なぜpromotersなのかというとプロジェクトにpromoterというAPIがあったということに他ならないので、皆さんはよしなに作成すると良いです。)
今回は以下のようなAPIを作成しました。

```js:mocks/promoters/_promoterId.js
const data = [
  {
    promoter_id: 1,
    list: [
      { date: "2021-01-01", friends_count: 100, followers_count: 200 },
      { date: "2021-01-02", friends_count: 120, followers_count: 210 },
      { date: "2021-01-03", friends_count: 140, followers_count: 300 },
      { date: "2021-01-04", friends_count: 140, followers_count: 320 },
      { date: "2021-01-05", friends_count: 160, followers_count: 330 },
      { date: "2021-01-06", friends_count: 167, followers_count: 350 },
      { date: "2021-01-07", friends_count: 173, followers_count: 380 },
      { date: "2021-01-08", friends_count: 177, followers_count: 370 },
      { date: "2021-01-09", friends_count: 189, followers_count: 410 },
    ]
  },
  {
    promoter_id: 2,
    list: [
      { date: "2021-01-01", friends_count: 157, followers_count: 200 },
      { date: "2021-01-02", friends_count: 172, followers_count: 222 },
      { date: "2021-01-03", friends_count: 200, followers_count: 234 },
      { date: "2021-01-04", friends_count: 223, followers_count: 277 },
      { date: "2021-01-05", friends_count: 227, followers_count: 312 },
      { date: "2021-01-06", friends_count: 231, followers_count: 380 },
      { date: "2021-01-07", friends_count: 233, followers_count: 392 },
      { date: "2021-01-08", friends_count: 240, followers_count: 402 },
      { date: "2021-01-09", friends_count: 242, followers_count: 407 }
    ]
  }
]

export default {
  get({ values }) {
    return [200, data.find(data => data.promoter_id === values.promoterId)]
  }
}
```
findメソッドでdataの配列から取得したpromoter_idとAPI呼び出し時に取得するidが一致する要素を取得します。このように簡単かつ柔軟にmockを書くことができるのでおすすめです。
以下は全てのpromoterを取得するgetメソッドです。
呼び出す時は`/promoters`で呼び出せます。
```js:mocks/promoters.js
const data = [
  {
    promoter_id: 1,
    list: [
      { date: "2021-01-01", friends_count: 100, followers_count: 200 },
      { date: "2021-01-02", friends_count: 100, followers_count: 210 }
      // 省略
    ]
  },
  {
    promoter_id: 2,
    list: [
      { date: "2021-01-01", friends_count: 150, followers_count: 200 },
      { date: "2021-01-02", friends_count: 200, followers_count: 300 },
      { date: "2021-01-03", friends_count: 300, followers_count: 320 }
      // 省略
    ]
  }
]

export default {
  get: () => [200, data]
}
```
試しにこのmockを叩いてみましょう。promoter_idを取得してみます。
先ほど出てきたようにnuxt.jsではpages配下に作成したファイルはルーティングとして作用します。
例えば,
`pages/chart/index.vue`は`http://localhost..../chart`というようになります。
というわけでこのページを作成してmockを叩いてみます。

```js:pages/chart/index.vue
<template>
  <ul>
    <li v-for="promoter in promoters" :key="promoter.promoter_id">
      <nuxt-link :to="'/chart/' + promoter.promoter_id">
        {{ promoter.promoter_id }}
      </nuxt-link>
    </li>
  </ul>
</template>
<script>

export default {
  async asyncData({ $axios }) {
    const promoters = await $axios.$get("/promoters")

    return { promoters }
  }
}
</script>
```

nuxt.jsの`asyncData`はpages内部のみで使用することが出来ます。
https://ja.nuxtjs.org/docs/2.x/features/data-fetching/#async-data
asyncDataでは非同期でデータを取得し、そのデータをコンポーネントに反映させることが出来ます。

asyncDataの処理の中で`$axios`があります。これは`@nuxtjs/axios`です。以下のページを参考にセットアップをすると、各コンポーネントで参照することが出来ます。
https://axios.nuxtjs.org/setup

```
npm install --save @nuxtjs/axios
```

```js:nuxt.config.js
export default {
  modules: ['@nuxtjs/axios']
}
```
create-nuxt-appではaxiosを設定することができるので、create-nuxt-appで設定するのも手取り早いのでおすすめです。
asyncDataでは以下のように$axiosを利用して先ほどのAPIを叩くことが出来ます。取得したデータをreturnしてtemplatesに渡します。
```js
  async asyncData({ $axios }) {
    const promoters = await $axios.$get("/promoters")

    return { promoters }
  }
```
ちなみにasyncDataが使用できないコンポーネントでは、`this.$axios.$get`というように書くことでaxiosにアクセスすることが出来ます。
次にtemplates部分です。v-forでmockから取得した配列データを展開します。
nuxt.jsでページ内リンクを作成するには`NuxtLink`を利用します。
リンク先に以下のように`promoter_id`を渡すことによって`http.../charts/1`のようにページ遷移をすることが出来ます。
```
<template>
  <ul>
    <li v-for="promoter in promoters" :key="promoter.promoter_id">
      <nuxt-link :to="'/chart/' + promoter.promoter_id">
        {{ promoter.promoter_id }}
      </nuxt-link>
    </li>
  </ul>
</template>
```

# nuxt.jsでparamsを取得する方法
先ほどmock APIから取得したIDをもとに遷移先のページを作成します。
nuxt.jsで動的ルーティングを作成する際には`promoters/_id.vue`のように`_`をつけることによってページを実装します。
※デザインにはvuetifyを一部で用いています。
https://vuetifyjs.com/ja/

```js:pages/chart/_id.vue
<template>
  <v-container>
    <page-title-label :title="id" />
  </v-container>
</template>

<script>
import PageTitleLabel from "@/components/atoms/labels/PageTitleLabel"

export default {
  components: {
    PageTitleLabel
  },
  asyncData({ params }) {
    const id = params.id
    return { id }
  }
}
</script>
```

上記の実装では、PageTitleLabelというコンポーネントを読み込んでいます。
このコンポーネントはtitleというPropsを持っています。
```js:PageTitleLavel.vue
<template>
  <h2 class="title indigo--text text--darken-4 font-weight-bold">
    {{ title }}
  </h2>
</template>
<script>
export default {
  props: {
    title: {
      type: String,
      default: "",
      required: true
    }
  }
}
</script>
```

このコンポーネントを登録し、propsに受け取ったデータを与える実装を行います。
先ほどの`pages/chart/index.vue`の実装で`promoter_id`をURLに設定し、`http://localhost.../chart/1`のようなURLを生成しました。このid`1`という値は`params`で取得することが出来ます。
実装部分は以下のようになります。`asyncData`はページ由来のparamsを受け取ることが出来ます。
```js
  asyncData({ params }) {
    const id = params.id
    return { id }
  }
```
これをtemplates部分に渡すことで使用することも出来るようになります。

# Chart.jsでChartを作成する
いよいよ本題になります。
作成したAPIから生成したparamsや取得しdataを使用してChartを作成していきます。
Vue.jsでChartを作成するにはChart.jsのラッパーライブラリであるvue-chartjsが便利です。
シンプルなChartであれば大体実装することが出来ます。
https://vue-chartjs.org/

vue-chartjsのもとはchart.jsなのでどのようなChartが作成出来るかはこちらでご確認ください。
https://www.chartjs.org/

インストールします。
```
npm install --save chart.js vue-chartjs
```

:::message
  vue-chartjsはchart.jsv3.0~に対応されていません。使用する場合はchart.jsのバージョンを下げる。もしくは以下の設定を行う必要があります。
  https://www.chartjs.org/docs/latest/getting-started/integration.html#bundlers-webpack-rollup-etc
  具体的には、`node-modules/vue-chartjs/es/BaseCharts.js`に以下の修正を行います。
  ```diff js
-   import Chart from 'chart.js';
+   import Chart from 'chart.js/auto';
  ```
:::

実装してみましょう。以下を参考にしつつ、進めていきます。
https://vue-chartjs.org/guide/#chart-with-api-data

```js:organisms/chart/LineChart.vue
<script>
import { Line, mixins } from 'vue-chartjs'
const { reactiveProp } = mixins

export default {
  extends: Line,
  mixins: [reactiveProp],
  props: {
    chartData: {
      type: Object,
      default: null
    },
    options: {
      type: Object,
      default: null
    }
  },
  mounted() {
    this.renderChart(this.chartData, this.options)
  }
}
</script>
```
vue-chartjsでは折れ線グラフ、棒グラフなどchart.jsで作成出来るグラフであれば実装することが出来ます。今回は折れ線グラフを作成するのでLineをインポートしています。
Sampleに追加したのがmixinの設定です。
この設定を行わないと、APIのデータが変更したりした場合、更新が行われません。
今回はmockなので変わることはあまりないですが、実際のAPIを叩く際にはdataが追加されていたりするのが普通だと思いますのでこの設定を忘れずにしましょう。
vue-chart.jsではrenderChartというAPIを実行して、グラフを描画します。
第一引数にdata、第二引数に設定を受け取ります。
dataの名前はchartDataとしないとうまく描画させないので注意してください。
ここではpropsを渡して、実装はContainer的なComponentで行います。

```js:components/organisms/chart/FriendscountLineChart.vue
<template>
  <div class="chart-container">
    <line-chart
      :chart-data="chartData"
      :options="options"
    />
  </div>
</template>
<script>
import LineChart from "@/components/organisms/chart/LineChart.vue"

export default {
  name: "FriendscountLineChart",
  components: { LineChart },
  props: {
    id: {
      type: String,
      default: null
    },
  },
  data: () => ({
    chartData: null,
    options: {}
  }),
  mounted() {
    this.fillData()
  },
  methods: {
    async fillData() {
      const response = await this.$axios.$get(`/promoters/${this.id}`)
      const date = response.list.map(list => list.date)
      const friendsCount = response.list.map(list => list.friends_count)

      this.chartData = {
        labels: date,
        datasets: [
          {
            label: "フォロー",
            backgroundColor: 'rgb(232, 234, 246, 0.1)',
            borderColor: "#C5CAE9",
            data: friendsCount,
            fill: true
          }
        ]
      }
      this.options = {
        responsive: true,
        maintainAspectRatio: false,
        scales: {
          y: {
            stacked: true,
            ticks: {
              color: "#E8EAF6",
            },
            grid: {
              color: "rgb(232, 234, 246, 0.1)"
            },
          },
          x: {
            ticks: {
              color: "#E8EAF6",
            },
            grid: {
              color: "rgb(232, 234, 246, 0.1)"
            }
          }
        }
      }
    }
  }
}
</script>
```

template部分で先ほど作成したLineChartコンポーネントにchartDataとoptionsを渡します。
```js
<template>
  <div class="chart-container">
    <line-chart
      :chart-data="chartData"
      :options="options"
    />
  </div>
</template>
```
propsにidを渡しています。これは`pages/chart/_id.vue`でグラフをparamsに応じて表示したいからです。
```js
  props: {
    id: {
      type: String,
      default: null
    },
  },
```

data()で`chartData`と`options`の初期値をリアクティブに登録します。
mounded()はVue.jsのライフサイクルメソッドで、dataよりも後に実行されます。`chartData`と`options`にthisでアクセスしてdataを追加していきます。
実装自体はmethods()の`fillData`というasync関数です。

まずはmockAPIからデータを取得する実装です。
```js
  const response = await this.$axios.$get(`/promoters/${this.id}`)
  const date = response.list.map(list => list.date)
  const friendsCount = response.list.map(list => list.friends_count)
```
getAPIを呼び出します。URLには`/promoters/${this.id}`を設定します。
これはpagesで受け取ったparamsを
```js
 <friendscount-line-chart :id="id" />
```
とうように渡すことでpropsから値が渡されるという仕組みです。
APIを呼び出せたら`map`を使って配列からデータを取得します。
APIのresponseはこんな感じなので、listから必要なデータを取得します。
```
  promoter_id: 1,
  list: [
    { date: "2021-01-01", friends_count: 100, followers_count: 200 },
    { date: "2021-01-02", friends_count: 120, followers_count: 210 },
  ]
```
dateはこのままだとただの文字列になってしまうので、必要に応じて`dayjs`などのライブラリを用いると良いでしょう。
https://github.com/iamkun/dayjs

```js
import dayjs from "dayjs"

const date = response.list.map(list => dayjs(list.date).format("MM/DD"))
```

これらのデータをchartに渡します。dataで作成した値にthisでアクセスして追加します。
```js
methods: {
    async fillData() {
      // 省略
      this.chartData = {
        labels: date,
        datasets: [
          {
            label: "フォロー",
            backgroundColor: 'rgb(232, 234, 246, 0.1)',
            borderColor: "#C5CAE9",
            data: friendsCount,
            fill: true
          }
        ]
      }
      this.options = {
        responsive: true,
        maintainAspectRatio: false,
      }
    }
  }
```

labelはここではx軸の値になります。日付のdataであるdataを渡します。
dataは`dataset`に渡します。dataがChartの値になります。
これでmockをもとに、Chartグラフの大元が作成出来ました！

# Chartのカスタマイズ

## axes(軸の目盛)のカスタマイズ
optionsで行います。目盛を変えたい場合などはscalesのx軸、y軸に設定を以下のように加えていきます。
```js
  this.options = {
    responsive: true,
    maintainAspectRatio: false,
    scales: {
      y: {
        stacked: true,
        ticks: {
          color: "#E8EAF6",
        },
        grid: {
          color: "rgb(232, 234, 246, 0.1)"
        },
      },
      x: {
        ticks: {
          color: "#E8EAF6",
        },
        grid: {
          color: "rgb(232, 234, 246, 0.1)"
        }
      }
    }
  }
```
ticksは目盛の値、gridは目盛の線の値です。
以下の公式ドキュメントをみながら、参照していくと良いと思います。
https://www.chartjs.org/docs/latest/general/options.html
https://www.chartjs.org/docs/latest/axes/

## レスポンシブ/レイアウト対応
Chart.jsはcanvas要素を用いてChartを描画するので、ここが辛い部分かもしれません。要素自体にスタイルを当てても、canvasはデフォルトの値を持っているのでうまく大きさなどを指定することが出来ません。
container要素を作成してそちらにスタイルを当てる必要があります。
```css
.chart-container {
  border-radius: 12px;
  padding: 16px;
  width: 45%;
}
```
大きさ等をコントロールするにはDOMにアクセス必要があるので、そこは仮想DOMと相反する実装になってしまいます。
なのでcanvasでグラフを描画する際にはそのデメリットを受け入れる必要があるのかもしれません。
(良い実装ありましたら教えてください)

## 完成系

実装の全体像は以下のようになります。
Chartの上部にグラフのタイトルなどを載せたりするカスタマイズを行っています。

```js:components/organisms/chart/FriendscountLineChart.vue
<template>
  <div class="chart-container">
    <div class="chart-heading">
      <h3 class="chart-heading-title indigo--text text--lighten-5 font-weight-bold">
        {{ title }}
      </h3>
      <p class="chart-heading-count indigo--text text--lighten-5 font-weight-bold">
        {{ latestFriendsCount }}
      </p>
    </div>
    <line-chart
      :chart-data="chartData"
      :options="options"
    />
  </div>
</template>
<script>
import LineChart from "@/components/organisms/chart/LineChart.vue"

import dayjs from "dayjs"

export default {
  name: "FriendscountLineChart",
  components: { LineChart },
  props: {
    id: {
      type: String,
      default: null
    },
    title: {
      type: String,
      default: null
    }
  },
  data: () => ({
    chartData: null,
    latestFriendsCount: null,
    options: {}
  }),
  mounted() {
    this.fillData()
  },
  methods: {
    async fillData() {
      const response = await this.$axios.$get(`/promoters/${this.id}`)
      const date = response.list.map(list => dayjs(list.date).format("MM/DD"))
      const friendsCount = response.list.map(list => list.friends_count)

      this.latestFriendsCount = friendsCount.slice(-1)[0]
      this.chartData = {
        labels: date,
        datasets: [
          {
            label: "フォロー",
            backgroundColor: 'rgb(232, 234, 246, 0.1)',
            borderColor: "#C5CAE9",
            data: friendsCount,
            fill: true
          }
        ]
      }
      this.options = {
        responsive: true,
        maintainAspectRatio: false,
        scales: {
          yAxes: {
            stacked: true,
            ticks: {
              color: "#E8EAF6",
            },
            grid: {
              color: "rgb(232, 234, 246, 0.1)"
            },
          },
          x: {
            ticks: {
              color: "#E8EAF6",
            },
            grid: {
              color: "rgb(232, 234, 246, 0.1)"
            }
          }
        }
      }
    }
  }
}
</script>
<style scoped>
.chart-container {
  background-image: linear-gradient(to right top, #5c6bc0, #5766be, #5261bc, #4c5cba, #4757b8, #4353b5, #3f50b2, #3b4caf, #3849ab, #3645a7, #3342a3, #303f9f);
  border-radius: 12px;
  padding: 16px;
  margin: 16px auto;
  width: 45%;
}
.chart-heading {
  display: flex;
  align-items: center;
}
.chart-heading-title {
  padding: 12px 28px;
  font-size: 20px;
}
.chart-heading-count {
  font-size: 44px;
  margin-left: auto;
  padding: 12px 28px;
}
</style>
```
ちなみにグラフの背景色はgradient generatorを使用しています。
https://mycolor.space/gradient

# 別のChartコンポーネントを作成

コピペして、データ取得の部分のみ実装を変更します。

```js:components/organisms/chart/FollowerscountLineChart.vue
<template>
  <div class="chart-container">
    <div class="chart-heading">
      <h3 class="chart-heading-title indigo--text text--lighten-5 font-weight-bold">
        {{ title }}
      </h3>
      <p class="chart-heading-count indigo--text text--lighten-5 font-weight-bold">
        {{ latestFollowersCount }}
      </p>
    </div>
    <line-chart
      :chart-data="chartData"
      :options="options"
    />
  </div>
</template>
<script>
import LineChart from "@/components/organisms/chart/LineChart.vue"

import dayjs from "dayjs"

export default {
  name: "FriendscountLineChart",
  components: { LineChart },
  props: {
    id: {
      type: String,
      default: null
    },
    title: {
      type: String,
      default: null
    }
  },
  data: () => ({
    chartData: null,
    latestFollowersCount: null,
    options: {}
  }),
  mounted() {
    this.fillData()
  },
  methods: {
    async fillData() {
      const response = await this.$axios.$get(`/promoters/${this.id}`)
      const date = response.list.map(list => dayjs(list.date).format("MM/DD"))
      const FollowersCount = response.list.map(list => list.followers_count)

      this.latestFollowersCount = FollowersCount.slice(-1)[0]
      this.chartData = {
        labels: date,
        datasets: [
          {
            label: "フォロー",
            backgroundColor: 'rgb(232, 234, 246, 0.1)',
            borderColor: "#C5CAE9",
            data: FollowersCount,
            fill: true
          }
        ]
      }

      this.options = {
        responsive: true,
        maintainAspectRatio: false,
        scales: {
          yAxes: {
            stacked: true,
            ticks: {
              color: "#E8EAF6",
            },
            grid: {
              color: "rgb(232, 234, 246, 0.1)"
            },
          },
          x: {
            ticks: {
              color: "#E8EAF6",
            },
            grid: {
              color: "rgb(232, 234, 246, 0.1)"
            }
          }
        }
      }
    }
  }
}
</script>
<style scoped>
.chart-container {
  background-image: linear-gradient(to right top, #5c6bc0, #5766be, #5261bc, #4c5cba, #4757b8, #4353b5, #3f50b2, #3b4caf, #3849ab, #3645a7, #3342a3, #303f9f);
  border-radius: 12px;
  padding: 16px;
  margin: 16px auto;
  width: 45%;
}
.chart-heading {
  display: flex;
  align-items: center;
}
.chart-heading-title {
  padding: 12px 28px;
  font-size: 20px;
}
.chart-heading-count {
  font-size: 44px;
  margin-left: auto;
  padding: 12px 28px;
}
</style>
```

この辺りのコンポーネントの共通化は次回の課題としたいと思います。
これらのコンポーネントを呼び出します。
コンポーネントのpropsのidに`params.id`を与えている点に注目してください。

```js:pages/chart/_id.vue
<template>
  <v-container>
    <page-title-label :title="id" />
    <div class="chart">
      <friendscount-line-chart :id="id" title="フォロー数" />
      <followerscount-line-chart :id="id" title="フォロワー数" />
    </div>
  </v-container>
</template>

<script>
import FriendscountLineChart from '@/components/organisms/chart/FriendscountLineChart.vue'
import FollowerscountLineChart from '@/components/organisms/chart/FollowerscountLineChart.vue'
import PageTitleLabel from "@/components/atoms/labels/PageTitleLabel"

export default {
  components: {
    FriendscountLineChart,
    FollowerscountLineChart,
    PageTitleLabel
  },
  asyncData({ params }) {
    const id = params.id
    return { id }
  }
}
</script>
<style scoped>
.chart {
  display: flex;
  justify-content: space-between;
}
</style>
```

## まとめ
vue-chartjsを使用すれば、vueでも簡単に良い感じのグラフを作成することが出来ます。ぜひ、やってみてください。


また、今回のコードにはComponentsの設計の部分で改善の余地があると感じています。
コンポーネントの共通化、どのコンポーネントでAPIを叩くのか？などまだまだ考えなくては行けないことが多いので改善を進めていきたいと思います。
そのあたりの話はまた記事にしたいと思います。

最後まで読んでくださりありがとうございました。

https://id.park-lot.com/

# PR
現在、7~9月に向けて転職活動中です。面接等してくださる会社様、TwitterのDMにてご連絡お待ちしております。
