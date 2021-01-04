---
title: "prettier,eslintを導入する際にハマったこと2021新年"
emoji: "😤"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["prettier", "eslint"]
published: true
---

# prettier,eslintについて
[prettier](https://prettier.io/)、[eslint](https://eslint.org/)使っていますか？
フォッマーターprettierとJavaScriptのコードをチェックするeslintはJavaScript開発においてなくてはならないものになっています。
変化の多いフロントエンドに合わせて、prettier/eslintも各フレームワークやパッケージに追従して進化を続けています(ありがたいです)。
しかし、それ故に以前の設定では現在は非奨励になっていたり、書き方も大きく変化しているようです。

この記事では自分がJavaScript(TypeScript)で適用している設定を紹介しようと思います。ややこしくなりがちなVSCodeのformatについても触れていきますのでぜひ参考にしてください。

# 要約
- フォーマットはprettierに任せよう(eslint-config-prettierを使用)
- VSCodeの設定はeditor.codeActionsOnSaveとeditor.defaultFormatterを設定しよう

# 環境
- Node.js v14.4.0
- @typescript-eslint/eslint-plugin　4.11.1
- @typescript-eslint/parser"　4.11.1
- eslint"　7.17.0
- eslint-config-prettier　7.1.0
- prettier　2.2.1
- typescript　4.1.3
  
概要は参考記事が詳しいので、要点だけを説明していきます。


# フォーマットはprettierに任せよう

他の優れた記事でも言及されていますが、フォーマットはprettierに委ねましょう。
[Integrating with Linters](https://prettier.io/docs/en/integrating-with-linters.html)
eslintなどのリンターにはルールだけでなく、コードをフォーマットする機能も含まれています。これらフォーマットの機能を無効化して機能の競合を防ぎ、prettierにフォーマットを任せるように設定を行いましょう。
具体的には**eslint-config-prettier**を導入します。

設定ファイルは以下のようになります。eslintの設定については[Configuring ESLint](https://eslint.org/docs/user-guide/configuring)をご覧ください。

```js:eslintrc.js
module.exports = {
  root: true,
  env: {
    es2020: true,
    node: true,
  },
  parser: '@typescript-eslint/parser',
  parserOptions: {
    sourceType: 'module',
    ecmaVersion: 2020,
    tsconfigRootDir: __dirname,
    project: ['./tsconfig.eslint.json'],
  },
  plugins: ['@typescript-eslint'],
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'prettier',
    'prettier/@typescript-eslint',
  ],
  rules: {},
};
```

extendsの最後にprettierを読み込むことで、eslintのフォーマットの機能を上書きして機能の競合をなくします。

parserでは@typescript/eslint/parserを使用してTypeScriptファイルを読み込みます。eslint用の設定ファイルを作成しましょう。
```json:tsconfig.eslint.json
{
  "extends": "./tsconfig.json",
  "include": [
    "src/**/*.ts",
    ".eslintrc.js"
  ],
  "exclude": [
    "node_modules",
    "dist"
  ]
}
```

フォーマット形式は.prettierrcに設定していきます(他にもYAMLやJavaScriptで記述できます)。

```json:.prettierrc
{
  "singleQuote": true,
  "trailingComma": "all"
}
```
設定項目は多くのものがありますが、デフォルト値が設定されているので、それに従い設定はシンプルなものに留めましょう。
https://prettier.io/docs/en/options.html

# VSCodeの設定はeditor.codeActionsOnSaveとeditor.defaultFormatterを設定しよう

次にVSCodeのformatの設定方法です。
以下の拡張機能をインストールします。
[ESlint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
[Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)

VSCodeではワークスペース直下の`.VSCode`というファイルに`settings.json`を置くことで、そこにVSCodeに関する設定をすることができます。
今回はファイルを保存したら自動でファイルをフォーマットするように設定をしていきます。


```json:.vscode/settings.json
{
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "eslint.format.enable": false,
  "editor.formatOnSave": true,
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
  },
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
  },
  "editor.lineNumbers": "on",
  "editor.rulers": [80],
  "editor.wordWrap": "on",
  "eslint.packageManager": "yarn",
  "files.insertFinalNewline": true,
  "files.trimTrailingWhitespace": true,
  "npm.packageManager": "yarn",
  "typescript.enablePromptUseWorkspaceTsdk": true
}
```

ここでも基本的な原則は**フォーマットはprettierに任せよう**です。
以下の設定はvscode-eslintv2で変更された保存時にeslintのコードアクションを実行する機能になります。eslint.autoFixOnSaveは廃止になっています。
```json
 "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
```

自動フォーマットはprettierに任せます。
editor.formatOnSaveを使用します。これは保存時にフォーマットを行う設定です。
ファイル拡張子ごとに`"editor.defaultFormatter": "esbenp.prettier-vscode"`を設定することによって、フォーマットをprettierで実行しています。

これでeslintとprettierの設定が完了しました。
上記の設定でtypescript,jsonファイルなどのeslintによる構文チェックとprettierによるformatが可能になっています。コードの全体像は以下のリポジトリをご覧ください。
https://github.com/YouheiNozaki/todo-app-server

# 追加すると良い
husky,lint-stagedを用いてgit commit時にeslint,prettierを実行する。
npm-run-allを使用して、package.jsonを効率よく実行する。
詳しくは上記リポジトリをご覧ください。

# 参考記事
https://qiita.com/mysticatea/items/3f306470e8262e50bb70
https://qiita.com/notakaos/items/85fd2f5c549f247585b1
https://blog.ojisan.io/prettier-eslint-cli
