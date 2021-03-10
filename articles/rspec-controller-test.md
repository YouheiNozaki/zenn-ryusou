---
title: "「EverydayRails -RspecによるRailsテスト入門」のcontrollersのテストをrequest specで書き換える"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Rails", "Rspec"]
published: false
---

Railsに最近入門中のりゅーそうです。
RailsのテストではRspecが用いられることが多いようです。
Rspecの使用方法から、テストの手法・考え方まで体系的に学べるものとして「[EverydayRails -RspecによるRailsテスト入門](https://leanpub.com/everydayrailsrspec-jp)」という素晴らしい書籍があります。
この本には現在にも通じるようなテストの手法が体系的に書かれておりまさに満足といった感じなのですが、書かれてから歳月が立っているということもありバージョン等で差異が多少出てきているようです(翻訳者である伊藤さんによる[サポートページ](https://blog.jnito.com/entry/2019/10/25/053521)があるため大体のことはこれを見ればRails6以降のヴァージョンにも対応出来ると思います)。

書籍でも触れられていますが、RailsのControllerをテストする際にはRails4系統の時にはController specが用いられていましたが、現在ではそれらのメソッドは非奨励となりRequest specを用いたテストが奨励されているそうです。
本書でも7章でrequest specでControllerのテストを書き換える実装が紹介されています。
しかし、全てのコードをrequest specで書き換える例は見られませんでした。
当記事ではController specを書き換える方法を紹介したいと思います。

# お断り
Railsの実務経験はないので、Controllerをどのようにテストすべきかの明確な答えを持っていません。
単に本書のコード例をrequest specの手法に沿って書き換えた記事になります。
実装するに当たって翻訳者である伊藤さんにアドバイスをいただきました。
https://teratail.com/questions/324966

# 前提条件とControllerテストについて
