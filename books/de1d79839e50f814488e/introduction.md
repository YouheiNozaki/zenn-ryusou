---
title: "はじめに"
---
# 自己紹介
はじめまして。
りゅーそうと申します。

普段は高校で教員をやっているものです。
学校の様々なサービスを作ることを目標にプログラミングの勉強をしています。
間近の目標はJamstack構成の学校HPを作ることです。
Twitterもやっているのでぜひ、フォローしてください。
https://twitter.com/ryusou_mtkh

# この本について

Next.jsとmicroCMSでブログみたいなサービスを作ります。

## Next.js
Next.jsは先日のnext.js confが盛り上がっていましたね。この本ではリリースされたばかりのv10~を対象に実装していきます。

## microCMS
いわゆるHeadlessCMSです。日本製でUIもシンプルで私みたいな非エンジニアでも使いやすいところが気に入っています。

## TypeScript
v4.0.5を使用します。TypeScriptを入れると型安全にコードを書けます。
今回作成するWEBサイトは正直TypeScriptなしでも書けないことは全然ないですが、TypeScriptを入れると補完がきく。変なコードを書いた時にコンパイルエラーをVSCode上で出してくれるなど、このくらいの規模でも開発効率がとても良くなると思います。

## この本で扱うこと一覧
 - create-next-app
 - TypeScriptの環境構築
 - NextImageを使用する
 - next/routerを使ったルーティング
 - fetchの型定義（ラッパーを作成）
 - 環境変数の扱い
 - getStaticPropsでSSGを実装
 - getStaticPathでDynamic Routing

## これから書く予定
 - dayjs
 - rehype
 - Incremental Static Regeneration
 - Preview Mode
 - verselでデプロイ

## 書かない
 - microCMSのAPI作成方法
 - CSS全般

## サンプルリポジトリ
サンプルサイトをデプロイするつもりはありませんが、リポジトリがあります。
これを読めばわかる人は読み必要一切ありません。
https://github.com/YouheiNozaki/history-video

## スターター
環境構築がめんどくさい方はぜひ使ってください(Next.jsは環境構築が簡単ですが)。
eslint,prettierなどが設定してあります。
https://github.com/YouheiNozaki/next.js-starterv10
