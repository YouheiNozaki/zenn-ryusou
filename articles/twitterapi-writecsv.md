---
title: "TwitterAPIのデータをCSVに出力する"
emoji: "😺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Node.js", "TwitterAPI"]
published: false
---

こんにちは。りゅーそう()です。
ご縁があり、PARKLoT(https://id.park-lot.com/)さんのサービス開発のお手伝いをさせていただいています。
TwitterAPIを使って関数を書く機会がありましたので、そこで得た知見や実装に至ったプロセスを紹介したいと思います。

# 作成したもの

TwitterAPIを叩いて、「Twitterキャンペーン」を行っているツイートを取得し、CSVに出力するツールを作成しました。
Twitterキャンペーンとは以下のようなツイートを指します。
https://twitter.com/PARKLoT1/status/1371068694718148614

## 実装の要件
このようなツイートを取得するためには、企業などが行っているキャンペーンツイートのみを取得する必要があります。
キャンペーンツイートには上記のツイート以外にも、応募者の「キャンペーンに応募しました！」というツイートやリプライなどがある可能性があります。
そういったツイートを除いて、純粋なキャンペーンツイートのみを取得する必要があります。

## 実装プロセス
要件をみたす機能を実装するためにまずは、キャンペーンツイートをツイッターで検索して「どのようなキャンペーンツイートがあるのか？」「キャンペーンツイートの共通点はなにか」を考えながら、スプレッドシートに数十件ほど書き込んでみました。
そのようにしていく中で以下のような共通点が見えてきました。
- 「キャンペーン」「フォロー」「開催」「RT」などの用語を含む
- ある程度の数のRT,いいねを含む
- プロモーションサイトへのリンクまたは画像を含む

この条件の中から、実装の要件を以下のように絞り込みました。
- 「キャンペーン」「フォロー」という用語を含む
- いいねの数30以上、RT100以上
- 画像もしくはリンクを含む

この条件は実際にサービスを開発・運用していく中で改善していくことになりそうです。
TwitterAPIを用いてこの条件を含めるには、クエリに
`キャンペーン フォロー min_faves:30 min_retweets:100 (filter:images OR filter:link)`
を含めれば実装できそうです。

参考：https://gist.github.com/cucmberium/e687e88565b6a9ca7039

# TwitterAPIを用いての実装
TwitterAPIを叩くために[node-twitter](https://github.com/desmondmorris/node-twitter)というライブラリを利用しました。
TwitterAPIからツイートを取得して、検証をするためにCSVに出力する関数を実装していきます。

## TwitterAPIの権限取得とClientの作成
TwitterAPIのアカウント申請は実際に行っていないのですが、以下のようなプロセスを踏まえてアカウント申請を行います。
https://dev.classmethod.jp/articles/twitter-api-approved-way/

取得したAPIキーを用いてクライアントを作成します。
dotenvなどを用いてAPIキーは.envファイルで管理します。
```javascript
require('dotenv').config()
const Twitter = require('twitter')

const client = new Twitter({
  consumer_key: `${process.env.TWITTER_CONSUMER_KEY}`,
  consumer_secret: `${process.env.TWITTER_CONSUMER_SECRET}`,
  access_token_key: `${process.env.TWITTER_ACCESS_TOKEN_KEY}`,
  access_token_secret: `${process.env.TWITTER_ACCESS_TOKEN_SECRE}`
})
```

##　データを追加
配列に取得したデータを追加します。
以下のページを参考に実装を進めました。
https://github.com/desmondmorris/node-twitter/tree/master/examples#search
https://developer.twitter.com/en/docs/twitter-api/v1/tweets/search/api-reference/get-search-tweets

client.getでツイートを取得します。
引数に(endpoint、params、data)を指定します。

```javascript
const data = []

async function searchTweet(count) {
  await client.get('search/tweets', {
    q: `キャンペーン フォロー min_faves:30 min_retweets:100 (filter:images OR filter:link)`,
    count: count,
    tweet_mode: 'extended'
  }, (error, searchData) => {
    if(error) {
      console.log("キャンペーンのツイート取得に失敗しました。")
      console.error(error)
      return error
    }
    for (item in searchData.statuses) {
      const tweet = searchData.statuses[item];
      data.push({
        id: tweet.id,
        name: tweet.user.name,
        screen_name: tweet.user.screen_name,
        image_url: tweet.user.profile_image_url,
        created_at: tweet.created_at,
        text: tweet.full_text.replace(/\r?\n/g, ""),
        favorite_count: tweet.favorite_count,
        retweet_count: tweet.retweet_count
      })
    }
  })
}

searchTweet(100)
```

注意点はツイートのテキストを取得する実装です。
標準ではsearchData.statusesにtextという値があります。
しかし、デフォルトではこのツイートの内容が全文取得されずに、省略されてしまいます。
なので`tweet_mode: 'extended'`を設定して、`full_text`を取得します。

また、full_textには改行が含まれてしまい、データをCSVなどに入れる際に整合性が取れなくなってしまいます。
ここでは、`tweet.full_text.replace(/\r?\n/g, "")`のように実装することで改行コードをなくしています。
参考: https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/String/replace

## CSVにデータを追加

今回はCSVにデータを書き込むためにcsv-writerというライブラリを使用しました。
https://www.npmjs.com/package/csv-writer

実装は以下のようになります。

```javascript
require('dotenv').config()
const Twitter = require('twitter')
const createCsvWriter = require('csv-writer').createObjectCsvWriter;

const client = new Twitter({
  consumer_key: `${process.env.TWITTER_CONSUMER_KEY}`,
  consumer_secret: `${process.env.TWITTER_CONSUMER_SECRET}`,
  access_token_key: `${process.env.TWITTER_ACCESS_TOKEN_KEY}`,
  access_token_secret: `${process.env.TWITTER_ACCESS_TOKEN_SECRE}`
})

const csvWriter = createCsvWriter({
  path: 'out.csv',
  header: [
    {id: 'id', title: 'ID'},
    {id: 'name', title: 'Name'},
    {id: 'screen_name', title: 'Screenname'},
    {id: 'image_url', title: 'Image'},
    {id: 'created_at', title: 'Created_at'},
    {id: 'text', title: 'Text'},
    {id: 'favorite_count', title: 'favorite_count'},
    {id: 'retweet_count', title: 'retweet_count'},
  ]
});

const data = []

async function searchTweet(count) {
  await client.get('search/tweets', {
    q: `キャンペーン フォロー min_faves:30 min_retweets:100 (filter:images OR filter:link)`,
    count: count,
    tweet_mode: 'extended'
  }, (error, searchData) => {
    if(error) {
      console.log("キャンペーンの一覧取得に失敗しました。")
      console.error(error)
      return error
    }
    for (item in searchData.statuses) {
      const tweet = searchData.statuses[item];
      data.push({
        id: tweet.id,
        name: tweet.user.name,
        screen_name: tweet.user.screen_name,
        image_url: tweet.user.profile_image_url,
        created_at: tweet.created_at,
        text: tweet.full_text.replace(/\r?\n/g, ""),
        favorite_count: tweet.favorite_count,
        retweet_count: tweet.retweet_count
      })
    }
    // CSVに出力
    csvWriter
      .writeRecords(data)
      .then(()=> console.log('The CSV file was written successfully'))
    },
  );
}


searchTweet(100)
```

これでディレクトリに`out.csv`が作成され、参照出来るようになりました！
```
ID,Name,Screenname,Image,Created_at,Text,favorite_count,retweet_count
1373288469963755500,【公式】ポイ活ならワラウ（warau）,warau_official,http://pbs.twimg.com/profile_images/1275338413768536064/SPKPHpwm_normal.png,Sat Mar 20 15:01:00 +0000 2021,／大吉を当てろ🎊おみくじキャンペーン🎉＼大吉のおみくじを選んだ人の中から、抽選で毎日3名様にAmazonギフト券「500円分」を #プレゼント🎁7日連続 #キャンペーン の「1日目」▼参加方法1.フォロー＆RT2.画像からおみくじを1個選んで本投稿にコメント■期限3/21(日)迄 #ポイ活 #懸賞 https://t.co/dsgQdYqXx4,125,556
1373288251130253300,Qoo10,Qoo10_Shopping,http://pbs.twimg.com/profile_images/1249926973016526848/yrxqGqnZ_normal.jpg,Sat Mar 20 15:00:08 +0000 2021,＼＼TIRTIRプレゼントキャンペーン最終日スタート／／ベッキョンのサイン入り商品が10名様に当たる👏マスクフィットクッション SPF50+PA+++とサインを合わせてプレゼント🎁■応募方法①Qoo10公式アカウントをフォロー②この投稿をRT🔄③リプライをチェック🔔#TIRTIR #Qoo10 https://t.co/6WHgqLwp7I,552,3240
1373288242162765800,コリュパ！《交流／交友／交遊》イベント情報サイト,koryupa,http://pbs.twimg.com/profile_images/1332880618410000384/QbNd7hzD_normal.jpg,Sat Mar 20 15:00:06 +0000 2021,コリュパ【フォロー＆RT #キャンペーン 】1名様にamazonギフト券《5000円分》プレゼント🎊3月号：7日目🎊＼毎日RTで当選確率UP／1️⃣@koryupaをフォロー2️⃣この投稿をRT🤤＜出産したらお寿司をたらふく食べたいです🍣特にイクラが好きです✨▼詳細https://t.co/W0NL4wlKpW https://t.co/oCDbSPNP97,208,987
1373288228149624800,うまさにいちず　みやぎ米,miyagimaija,http://pbs.twimg.com/profile_images/1545162362/twitter_ico_normal.jpg,Sat Mar 20 15:00:02 +0000 2021,＼その場で当たるキャンペーン実施中！／ 期間中99名に #みやぎ米 2kgと買物バッグセットがその場で当たる🍚外れても１名に人気家電が当たるチャンス 🎁ご飯は、みやぎ米できまり！①@miyagimaijaをフォロー②このツイートを3/21 23:59までにRT③結果が自動返信で届く！（家電当選者は後日通知） https://t.co/5jagzXzdJA,426,2658
1373288221916852200,別冊少年マガジン【公式】,BETSUMAGAnews,http://pbs.twimg.com/profile_images/709387261897342978/3DvyjBK2_normal.jpg,Sat Mar 20 15:00:01 +0000 2021,『 #進撃の巨人 』終了直前！！4/9の最終話までカウントダウン！５週連続フォロー＆RTキャンペーン開催！『進撃の巨人』特製QUOカードが毎週3名様に当たる！1⃣フォロー2⃣このツイートをRT3週目：3/27（土）23:59まで詳細はhttps://t.co/sAqMFSqRfW#進撃の巨人最終話直前　#プレゼントCP https://t.co/RuFObR9zsX,693,1936
<!-- 以下省略 100件取得される -->
```

# まとめ
実際にチーム開発を行うのは実は初めてでした。
単純な実装でしたが、要件などをチームで確認したりしながらする開発は楽しかったです。

今後もツールの改善・開発を進めていきたいと思います。

# 参考記事
https://gist.github.com/cucmberium/e687e88565b6a9ca7039
https://github.com/desmondmorris/node-twitter
https://dev.classmethod.jp/articles/twitter-api-approved-way/
https://github.com/desmondmorris/node-twitter/tree/master/examples#search
https://developer.twitter.com/en/docs/twitter-api/v1/tweets/search/api-reference/get-search-tweets
https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/String/replace
https://www.npmjs.com/package/csv-writer
