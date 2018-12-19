# Next.jsの基礎
この章では、Next.jsのインストールから基礎を説明します。

## Next.jsのインストール
最近では`create-next-app`(`create-react-app`ライクなコマンド一発で`Next.js`の環境を構築できる便利なツール)が開発されたことにより、簡単に環境構築することができます。  
```sh
npm install -g create-next-app
create-next-app sample-app
cd sample-app
npm run dev
```

`--example サンプル名`のオプションを付与することで、用途に応じた雛形を用意することもできます。例えば、Next.jsでReduxを使用したい場合、下記のように指定することで、Reduxのサンプルプロジェクトが生成されます。
```sh
create-next-app --example with-redux sample-redux-app
```
【参考URL】
- [GitHub - segmentio/create-next-app](https://github.com/segmentio/create-next-app)
- [Create Next App公式サイト](https://open.segment.com/create-next-app/)
- [オプション一覧](https://github.com/zeit/next.js/tree/master/examples)

今回は、`create-next-app`を使用せずに進めていきましょう。

```sh
mkdir next-js-app
cd next-js-app
npm init

// Next.jsのパッケージのインストール
npm install nextjs@latest --save-dev
npm install react@latest react-dom@latest --save
```

プロジェクトをgit管理する場合は、下記のファイル・ディレクトリを`.gitignore`しましょう。
```.gitignore
.DS_Store
.idea
.next
build
coverage
node_modules
npm-debug*
out
yarn-debug*
yarn-error*
```

## development環境でNext.jsアプリケーションを実行しよう
`package.json`に下記を追記しましょう。
```json
{
  "scripts": {
    "start": "next"
  }
}
```

```sh
npm start
```

ブラウザから`localhost:3000`にアクセスし、サーバが起動していることを確認しましょう。

## 最初のNext.jsページを作ろう
`pages`ディレクトリ以下に簡単なコンポーネントを作成しましょう。
```js
// pages/index.js
import React from "react";
export default () => (<div>Hello, world!</div>);
```

`npm start`して`localhost:3000`にアクセスすると、`Hello, world!`と表示されているのが確認できます。
