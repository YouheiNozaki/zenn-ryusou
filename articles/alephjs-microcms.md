---
title: "microCMSとDeno(Aleph.js)で少し未来のWEBサイト開発"
emoji: "🦕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["microcms", "deno"]
published: true
---

今日は少しだけ未来のWEBサイト開発のお話をします。
ですが、ある程度形になりそうなので、それほど遠くはない未来かもしれません。
Aleph.jsとmicroCMSで簡単なWEBサイトを作成してみました。
開発したサイトは以下のシンプルなサイトになります。
https://microcms-aleph.vercel.app/

:::message
この記事で紹介するAleph.jsは2021/7/3現在、v0.3.0-alpha.33とalpha版です🙏
:::

# Deno製フレームワークAleph.js
今回はDeno製のSSGフレームワークであるAleph.jsを使用してWEBページを作成してみました。Aleph.jsとはNext.jsにインスパイアを受けたフレームワークであり、ドキュメントの構成もとてもよく似ています(というかNext.jsのドキュメントを引用して作成されています)。

https://alephjs.org/

# microCMS
HeadlessCMSは国産のCMSであるmicroCMSを利用しました。最近ではJavaScriptSDKが開発され、さらに開発がしやすくなりました。

https://microcms.io/

https://document.microcms.io/tutorial/javascript/javascript-sdk

# Deno(Aleph.js)の開発は何が嬉しいか

## Next.js風フレームワーク
著者は普段はReact(Next.js)を利用しています。Aleph.jsはNext.jsインスパイアということもあり、APIやルーティングの方法もNext.jsとほぼ同様に書くことができます。もちろんJSXを用いて開発することが出来るので、Reactを普段から使用している方は違和感なく開発することができます。

## TypeScript対応
DenoはデフォルトでTypeScriptをサポートしているので、ts.configなどの設定が必要ありません。

## node_modulesとpackage.jsonが必要ない
Denoはnpmを使用していないので、巨大なnode_moduleがDenoでは必要ありません。これはコンパイルやビルドの時間短縮に大きな効果を発揮します。
特にSSGフレームワーク＋HeadlessCMS構成(いわゆるJamstackのカテゴリ)ではビルド時間が問題になりやすいので、大きなメリットになります。
かわりにDenoでは、Third Party ModulesをCDNで配信することでpackageを利用します。
https://deno.land/x
それ以外にESMなどのCDNから対応しているnpm pakageを使用することができます(他にもskypackなどが存在します)。
https://esm.sh/


このようにAleph.jsは現在α版ではありますが、これまでのリソースを生かしつつ、Jamstackが抱えている問題点などを解決するアプローチの一つになりうる可能性があります。


## セットアップ
では、実際にAleph.jsとmicroCMSを用いて開発をする方法を紹介します。
今回デプロイはVerselを利用します。

brewなどを用いて環境にDenoをインストールしてください。
https://deno.land/#installation


今回使用するバージョンは以下の通りです。
```
$ aleph --version
aleph.js 0.3.0-alpha.33
deno 1.11.2
v8 9.1.269.35
typescript 4.3.2
```

セットアップは基本的には公式Docsの手順で進めます。ただ、ドキュメントのバージョンでは動かない可能性があります。上記に記載のバージョンで進めるのが良いかと思います(言わずもがなですが、この記事の方法は今後動かなくなる可能性があります)。

aleph.jsをインストールします
```
deno install --unstable -A -f -n aleph https://deno.land/x/aleph@0.3.0-alpha.33/cli.ts
```

プロジェクトの作成
```
aleph init project
cd project
```

開発モードでビルド(HMRが効きます。ビルドが体感めちゃくちゃ速いです)
```
aleph dev
```

今回のサンプルのリポジトリは以下になります。ご参照ください。要点だけ絞って解説します。
https://github.com/YouheiNozaki/microcms-aleph


# Aleph.jsの開発方法

## ディレクトリ構成
以下のようになります(設定を加えるとsrc Dirにすることもできます)。
基本的にはNext.jsと同じなので、使用経験がある方は違和感ないと思います。
```
project
 - .vscode(vscodeの設定ファイル)
 - dist(build時に作成されるファイル.Verselではこのファイルを実行する)
 - components(コンポーネントのファイル)
 - lib(関数とか)
 - pages(Next.jsのFile Systemと同じくこのファイルがルーティングになる)
 - public(画像などの静的ファイル)
 - style(グローバルスタイルとか、pagesのスタイルとか)
 - types(typescriptの型定義ファイル)
 - .env(環境変数)
 - .gitignore
 - aleph.config.js(alephのビルド設定など。今回は使用しないがSSR,SSGの設定などはこのファイルで行う)
 - app.tsx(レイアウトファイル。Next.jsのapp.tsxと同じ)
 - import_map.json(importのURLを指定できる)
```

ここではAleph.js特有の部分を解説します。
 - aleph.config.js
 こちらは今回の記事では特別な設定はしませんが、SSRとSSGをページごとに設定したり、Next.jsでいうところの`getStaticPath`のパラメータ設定はこのファイルで行います。
 https://alephjs.org/docs/basic-features/ssr-and-ssg
 - import_map.json
 先にも述べましたが、Denoではパッケージをimportする際にCDNを利用します。
 具体的には以下のようにしてパッケージをimportする必要があります。
 ```
 import React from "https://esm.sh/react@17.0.2"
 ```
 毎回このようにURLを入力するのは面倒なので、このURLの省略の設定をこのファイルに書きます。具体的には以下のようになります。
 ```
 {
  "imports": {
    "~/": "./",
    "aleph/": "https://deno.land/x/aleph@v0.3.0-alpha.33/",
    "aleph/types": "https://deno.land/x/aleph@v0.3.0-alpha.33/types.ts",
    "framework": "https://deno.land/x/aleph@v0.3.0-alpha.33/framework/core/mod.ts",
    "framework/react": "https://deno.land/x/aleph@v0.3.0-alpha.33/framework/react/mod.ts",
    "react": "https://esm.sh/react@17.0.2",
    "react-dom": "https://esm.sh/react-dom@17.0.2",
    "microcms": "https://esm.sh/microcms-js-sdk"
  },
  "scopes": {}
}
```
これで、パッケージ名を書くだけで使用することができます。

:::message
これでもVSCodeでimport文にエラーが出てしまう場合は`deno info　[パッケージのURL]`とコマンド入力してみてください
https://deno.land/manual/tools/dependency_inspector
:::

# microCMSと連携
https://github.com/YouheiNozaki/microcms-aleph
リポジトリの概要を解説します。
Next.jsの開発と非常に似通っているので、以下のmicroCMSとNext.jsのチュートリアルを理解していると良いかもしれません。
https://blog.microcms.io/microcms-next-jamstack-blog/

## microCMS Clientの準備
適当にmicroCMSでコンテンツAPIを作成します。
ちなみに今回は`article`APIの中に以下のようなスキーマを作成しました。
```
{
    "id": "zenn6",
    "title": "nuxt.jsでChartグラフを作るチュートリアル",
    "url": "https://zenn.dev/ryusou/articles/nuxt-chartjs",
    "publish_article": "2021-04-05T15:00:00.000Z",
}
```

作成できたら、microcms-js-sdkを使用してmicroCMSのデータを呼ぶ設定を行います。
https://github.com/microcmsio/microcms-js-sdk

コードは以下のようになります。
```js:lib/microcmsClient.ts
import { createClient } from "microcms";

export const microcmsClient = createClient({
  serviceDomain: "ryusou-portfolio",
  apiKey: `${Deno.env.get("X_API_KEY")}`,
});
```
serviceDomainには`xxxxx.microcms.io`の独自のドメイン名`xxxx`の部分を入力します。
apiKeyはGitHub等に公開することができないので`.env`ファイルなどに保存します。
```:.env
X_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxx
```
Nodeの`process.env`の代わりに`Deno.env`を使用して値を参照します。
これでmicroCMSで作成したAPIを呼び出すことができるようになります。SDKのおかげでだいぶコードが簡略化されるようになりました！

## データを呼び出す
`pages/index.tsx`に以下のようにしてデータを呼び出します。

```ts:pages/index.tsx
import { useDeno } from "framework/react";
import React from "react";
import "../style/home.css";
import { microcmsClient } from "../lib/microcmsClient.ts";

export type Post = {
  contents: [{
    id: string;
    url: string;
    title: string;
    publish_article: string;
  }];
};

export default function Home() {
  const articles = useDeno(async () => {
    return await microcmsClient.get<Post>({
      endpoint: "articles",
      queries: { limit: 99 },
    });
  });

  return (
    <div className="page">
      <head>
        <title>Ryusou Profile</title>
      </head>
      <section>
        {articles.contents.map((content) => {
          return (
            <React.Fragment key={content.id}>
              <a href={content.url} alt={content.title}>
                <p>{content.title}</p>
                <p>{content.publish_article}</p>
              </a>
            </React.Fragment>
          );
        })}
      </section>
    </div>
  );
}
```
Aleph.jsではData fetchを`useDeno`を用いて行います。これはNext.jsの`getStaticProps`や`getServerSideProps`に当たる機能です。
https://alephjs.org/docs/advanced-features/use-deno-hook

Reactのように処理を関数のように書けるのは直感的で良い印象です。今回は`microcms-js-sdk`を使用しているので必要はありませんが、`useDeno<Type>(() => ....)`のように型を簡単にレスポンスにつけることが出来ます。

## スタイル
ドキュメントによるとCSSを書くだけでhash.js形式でコンパイルされると書いてありますが、コンソールをみたところ、`<style text/css data-module-id="style/index.css"/>`のような形式で表示されていました。
本当にmodule化されているのか、よく分かりません.....分かり次第追記します。
https://alephjs.org/docs/basic-features/built-in-css-support


## デプロイ
デプロイはVerselを使用します。基本的にはドキュメント通りでデプロイできるはずです。
- Buildコマンドのaleph.jsのバージョンを合わせること
- Output Dirをdistにすること
- VerselのEnvironment Variablesに`X_API_KEY`を設定することを忘れないようにしてください。

Git連携すると簡単にデプロイできると思います。


# まとめ
DenoのSSGフレームワークAleph.jsはNext.jsのように扱えて、さらにDenoのメリットを享受できるので、将来性に期待出来ますね。
Denoの今後の発展が楽しみです！！

microCMSは`microcms-js-sdk`の開発により、より簡潔にコードを書けるようになりました！また、サービス自体の機能もどんどん充実しているので、今後が更に楽しみです!
https://microcms.io/

# 補足
開発中に調べたDenoTips
## formatter
やっぱり欲しいですよね。Denoでは`deno fmt`でformatすることが出来ます。とても速いです。
VSCodeの拡張があると嬉しいです。

## linter
自分は使用していませんが、独自にLinerがあるそうです。
https://zenn.dev/magurotuna/articles/66618f26475702



