---
title: "実務で使うGitコマンドのメモ"
emoji: "🧞‍♂️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["git"]
published: true
---

実務で使うGitの備忘録。
自分用のメモ

# git stash

一億回はググっている

変更を退避。別の作業に移行したい時に
```
git stash
```

stashの一覧
```
git stash list
```

stashを戻す
```
git stash apply stash@{0}
```

参考
https://qiita.com/chihiro/items/f373873d5c2dfbd03250

# conflictが起きた時

まずは
```
git fetch
```

対象のブランチで、mergeする
```
git merge --no-ff origin/master
```
`non fast-forward`をオプションにつけるとブランチがそのまま残るので、作業の特定がしやすい

conflict部分を解消して再度`push`

# 特定のコミットに戻りたい
`git reset`を使う
https://qiita.com/kmagai/items/6b4bfe3fddb00769aec4

mixedでコミット、インデックス削除の変更のみ残せる。
```
git reset --mixed HEAD^
```
そしたら、VSCodeでぽちぽちして取り込みたいファイルを取捨選択。

HEADについてはこれ。
^をつけると１つ前のコミットを指定できる
https://bleis-tift.hatenablog.com/entry/20100922/1285140344#_commitname


# 宣言
Gitでもうやらかさないようにする！！
