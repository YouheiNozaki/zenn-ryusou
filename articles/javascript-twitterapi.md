---
title: "モダンJavaScriptへの第一歩 ~「TwitterAPIのデータをCSVに出力する」反省編~"
emoji: "😽"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["JavaScript"]
published: true
---
https://id.park-lot.com/

以前[TwitterAPI](https://zenn.dev/ryusou/articles/twitterapi-writecsv)に関する記事を書かせていただきました。
今回はそのコードで指摘を受けた問題点などをもとに、JavaScriptを書く際に心がけることについて考えていこうと思います。


上記の記事で書いたのは、以下のようなコードです。

```javascript

// 一部抜粋
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
   searchData.statuses.forEach(tweet => {
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
    })
    // CSVに出力
    csvWriter
      .writeRecords(data)
      .then(()=> console.log('The CSV file was written successfully'))
    },
  );
}


searchTweet(100)
```

上記のコードの問題点はなんでしょうか？ぜひ、考えてみてください。
色々突っ込みどころがありそうですが、一部書き換えた中で感じた自分の解釈を今回は述べていこうと思います。

# 1.グローバル空間にdataという汎用的な定数を定義してしまっている。
まず`const data = []`というようなコードをsearchTweetの外で定義しています。この関数がサービスでモジュールとして使用することを考えるとこのようなdataという汎用的な定数をグローバル空間に宣言するべきではありません。
定数名が衝突したり、意図せずに配列操作がされてしまう恐れがあります。
不要な定数を定義しないように心がける必要がありそうです。

# 2.破壊的メソッドでの配列操作
1にも関わる話ですが、いわゆる破壊的なメソッドで配列操作を行うことは避けるべきです。
参考:https://jsprimer.net/basic/array/#mutable-immutable

`data.push`は元の配列の値の要素を追加して、上書きするメソッドです。
破壊的メソッドの使用はさけ、immutable(非破壊的)なコードを書くように心がける必要があります。

ここでは配列操作に`map`メソッドを使用するのが良いでしょう。
`forEach`メソッドは破壊的メソッドによる配列操作を可能にするため、なるべく避けるのが良いです。また速度も`map`の方が速いそうです。
参考：[Qiita forEach vs map in JavaScript](https://qiita.com/Uuparu/items/e9867cb6c48736700a4f#:~:text=%E3%83%BB%20map()%20%E3%81%AF%E6%96%B0%E3%81%97%E3%81%84%E9%85%8D%E5%88%97,%E3%81%9A%E3%80%81%E5%B8%B8%E3%81%AB%20undefined%20%E3%82%92%E8%BF%94%E3%81%99%E3%80%82&text=map()%20%E3%81%AF%E5%85%83%E3%81%AE,%E3%81%BB%E3%81%86%E3%81%8C%E5%87%A6%E7%90%86%E9%80%9F%E5%BA%A6%E3%81%8C%E9%AB%98%E3%81%84%E3%80%82)

1.2を合わせると以下のようなコードになります。

```javascript
const data = searchData.statuses.map(tweet => {
  return {
    id: tweet.id,
    name: tweet.user.name,
    screen_name: tweet.user.screen_name,
    image_url: tweet.user.profile_image_url,
    created_at: tweet.created_at,
    text: tweet.full_text.replace(/\r?\n/g, ""),
    favorite_count: tweet.favorite_count,
    retweet_count: tweet.retweet_count
  }
})
```
ここでは、余計な配列を生成せずにTwitterAPIが返すオブジェクトの配列からデータをreturnするコードを書いてみました。このように定数を宣言することを減らし、immutableなメソッドを使用することで、意図しない配列操作をなくすことができます。

# 3.Promise処理
async,awaitを使用することで、可読性が高まります。
API通信やPromiseは非同期に処理が行われるのでawaitで処理の順番を制御します。
エラーは`try...catch`で処理しました。

```javaScript
async function searchTweet(count) {
  try {
    const searchData = await client.get('search/tweets', {
      q: `キャンペーン フォロー min_faves:30 min_retweets:100 (filter:images OR filter:link)`,
      count: count,
      tweet_mode: 'extended'
    })
    const data = searchData.statuses.map(tweet => {
    return {
      id: tweet.id,
      name: tweet.user.name,
      screen_name: tweet.user.screen_name,
      image_url: tweet.user.profile_image_url,
      created_at: tweet.created_at,
      text: tweet.full_text.replace(/\r?\n/g, ""),
      favorite_count: tweet.favorite_count,
      retweet_count: tweet.retweet_count
    }
  })
    // CSVに出力
    await csvWriter.writeRecords(data)
    console.log('The CSV file was written successfully')
  } catch (error) {
    console.log("キャンペーンの一覧取得に失敗しました。")
    console.error(error)
  }
}
```

エラー処理はこのメソッドだけでは判断出来ませんが、例えば`throw new Error()`でエラーを投げて処理を止める。別のコンポーネントにroutingする。alertを出す。などサービスに応じてエラー処理を行います。
このメソッドは管理システムのため、consoleでエラー出力をするのに留めました。

# まとめ
意図しない挙動を防ぐために、
- 不要な定数・変数を宣言しない
- forEachなどを使用する前に配列操作はimmutableなメソッドを使用できないか検討する

コードが良いものになるようにさらに改善していきます。

# 参考

このあたりをよく読んで、配列操作に強くなりましょう。
https://jsprimer.net/basic/array/
https://ics.media/entry/200825/
