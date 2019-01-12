# Next.jsの基礎
この章では、Next.jsのインストールから基礎を説明します。

## Next.jsのインストール
最近では`create-next-app`(`create-react-app`ライクなコマンド一発で`Next.js`の環境を構築できる便利なツール)が開発されたことにより、簡単に環境構築することができます。  
```bash
npm install -g create-next-app
create-next-app sample-app
cd sample-app
npm run dev
```

`--example サンプル名`のオプションを付与することで、用途に応じた雛形を用意することもできます。例えば、Next.jsでReduxを使用したい場合、下記のように指定することで、Reduxのサンプルプロジェクトが生成されます。
```bash
create-next-app --example with-redux sample-redux-app
```
【参考URL】
- [GitHub - segmentio/create-next-app](https://github.com/segmentio/create-next-app)
- [Create Next App公式サイト](https://open.segment.com/create-next-app/)
- [オプション一覧](https://github.com/zeit/next.js/tree/master/examples)

今回は、`create-next-app`を使用せずに進めていきましょう。

```bash
mkdir next-js-app
cd next-js-app
npm init

// Next.jsのパッケージのインストール
npm install nextjs@latest --save-dev
npm install react@latest react-dom@latest --save
npm install next --save
```

プロジェクトをgit管理する場合は、下記のファイル・ディレクトリを`.gitignore`しましょう。
```bash
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

pagesディレクトリを作成して、`npm start`します。
```bash
mkdir pages
npm start
```

ブラウザから`localhost:3000`にアクセスし、サーバが起動していることを確認しましょう。 
著者の環境だと`404 This page could not be found.` が表示されます。


## Next.jsのHello, world!
`pages`ディレクトリ以下に簡単なコンポーネントを作成しましょう。
```js
// pages/index.js
import React from "react";
export default () => (<div>Hello, world!</div>);
```

`npm start`して`localhost:3000`にアクセスすると、`Hello, world!`と表示されているのが確認できます。


## Productionビルド
Next.jsを使用して本番運用するにあたって、静的なWebサイトの場合と動的なWebアプリケーションの場合とでビルドのスクリプトが異なります。

**静的なWebサイトの場合**
1. `package.json`に下記のように記述します。
```json
{
  "scripts": {
    "build": "next build",
	"static": "next export"
  }
}
```

2. `next.config.js`にpathマップを記述します。
```js
module.exports = {
  exportPathMap: () => ({
    '/': {page: '/'}
  })
}
```

3. 動作テスト
下記コマンドを実行しましょう
```bash
npm run build
npm run static
```

`npm run static`すると、`/out`ディレクトリが生成されるので、その中の`index.html`を開きましょう。
先ほど作成した`Hello, world!`が表示されるはずです。  
このようにしてProductionビルドすると、実体は静的なファイル(htmlファイル)になるため、サーバを選ばずデプロイすることができます。

**動的なWebサイトの場合**
1. `package.json`に下記のように記述します。
```json
{
  "scripts": {
    "build": "next build",
	"server": "next start"
  }
}
```

2. 動作テスト
```bash
npm run build
npm run server
```


## ルーティング
Next.jsでURLに対応してページの出し分け(=ルーティング)を行いましょう。  
まず、下記のように`index.js`とは別のページを用意しましょう。
```js
// pages/another.js
import React from "react";
export default () => (<div>Another</div>)
```

anotherページに遷移するリンクを`index.js`に記述しましょう。
```js
// pages/index.js
import React from "react";
import Link from "next/link";
export default () => (<div><Link href="/another"><a>Another</a></Link></div>);
```

`aタグ`ではなく`Link`を使用することで、ページ全体のリロードが発生せず差分レンダリングされるため、高速化が期待できます。
さらなる高速化を行いたい場合、`Link`に`prefetch`を付与してください。
```html
<Link prefetch>
```
`Link`の中に`aタグ`を記述したのは、ページ遷移(`Link`)とスタイル(`aタグ`)という二つの役割を分離するためです。  


続いて、2つのページで共通して使用するNavigationコンポーネントを作成し、ページ間の行き来をできるようにしましょう。

1. 現在のURLに応じて文字の太さが変化するコンポーネントを作成

```js
// components/Btn.js
import React from "react";
import {withRouter} from "next/router";

const Btn = ({href, onClick, children, router}) => (
  <span>
    <button onClick={onClick} style={{fontWeight : router.pathname === href ? 'bold' : ''}}>
	  { children }
	</button>
  </span>
);

export default withRouter(Btn)
```

2. `Navigation`コンポーネントの作成
```js
// components/Navigation.js
import React from "react";
import Link from "next/link";
import Btn from "./Btn";

export default () => (
  <div>
    <Link href="/" passHref><Btn>Index</Btn></Link>
    <Link href="/another" passHref><Btn>Another</Btn></Link>
  </div>
);
```

3. 各ページにNavigationコンポーネントを設置します。
```js
// pages/index.js
import React from "react";
import Navigation from "../components/Navigation";

export default () => (
  <div>
    <Navigation/>
	<hr/>
	Index
  </div>
);
```

`another.js`でNavigationコンポーネントをimportし、renderします。

```js
// pages/another.js
import React from "react";
import Navigation from "../components/Navigation";

export default () => (
  <div>
    <Navigation/>
	<hr/>
	Another
  </div>
);
```

4. 動作テスト
`localhost:3000`にアクセスして各Navigationをクリックして、ページ遷移することを確認しましょう。


## ダイナミックルーティング
実際のWebアプリケーションはページとURLが1対1に対応しているわけではなく、idやタイトルに応じてコンテンツが変化します。
ここでは、ダイナミックルーティングについて学んでいきましょう。

1. ダイナミックルーティングに使用するデータファイルを定義します

```js
// data/posts.js
export default [
  {title: 'Foo'},
  {title: 'Hoge'},
  {title: 'Bar'},
];
```

2. `index.js`に上記のページに対応する`Link`を記述します

```js
// pages/index.js
import React from "react";
import Link from "next/link";
import Navigation from "../components/Navigation";
import posts from "../data/posts";

export default () => (
  <div>
    <Navigation/>
    <hr/>
	{posts.map((post, index) => (
	  <li key={index}>
	    <Link href={{pathname: '/another', query: {id: index}}}>
		  <a>{post.title}</a>
		</Link>
	  </li>
	))}
  </div>
);
```
Next.jsでは、Linkにオブジェクトを渡すことで、自動でStringに変換してくれます。

3. `another.js`でページのタイトルを表示するようにします

```js
// pages/another.js
import React from "react";
import Navigation from "../components/Navigation";
import posts from "../data/posts";

export default ({url: {query: {id}}}) => (
  <div>
    <Navigation/>
    <hr/>
	<h1>{posts[id].title}</h1>
  </div>
);
```

この状態で、`localhost:3000`にアクセスしいずれかのリンクをクリックしてください。
anotherページに`title`が表示されるはずです。  
＊ 直接、`localhost:3000/another`にアクセスする、あるいは`posts.js`の配列のインデックスに存在しないidを渡した場合、`posts[id]`が未定義のためエラーになります。

4. `posts[id]`が未定義の場合はエラーページを表示するようにする
`posts[id]`が未定義の場合、三項演算子を使ってシンプルにNext.jsが用意している`Error`コンポーネントを表示しましょう。
```js
import React from "react";
import Error from "next/error";
import Navigation from "../components/Navigation";
import posts from "../data/posts";

export default ({url: {query: {id}}}) => (
  (posts[id]) ? (
    <div>
      <Navigation/>
      <hr/>
      <h1>{posts[id].title}</h1>
    </div>
  ) : (
    <Error statusCode={404}/>
  )
);
```


## SEOフレンドリーなURL
先ほどのようにクエリパラメータ(`localhost:3000/another?id=1`)でidを指定した場合、SEOフレンドリーではありません。
そこで、idがURLの一部(`localhost:3000/another/1`)となるように実装してみましょう。  
`Link`の`prop`として`as`を渡します。
```html
<Link as={`/another/${index}`} href={{pathname: '/another, query: {id: index}}}>
  <a>{post.title}</a>
</Link>
```

`localhost:3000`にアクセスし、`Foo`, `Hoge`, `Bar`のいずれかのリンクをクリックすると、URLが`localhost:3000/another/1`のようになっていることを確認できます。  
しかし、このままの状態では一点問題があります。それはリロードした時です。リロードすると`404 Page Not Found`のエラーが発生します。
これは、URLのマスキングがクライアントサイドで行われており、サーバサイドでは`localhost:3000/another/1`のようなURLにアクセスした時に、どのページを表示すれば良いかわからないからです。
そこで、expressをインストールしてこの問題を解決しましょう。

1. `express`のインストール
```bash
npm install --save-dev express
```

2. `server.js`の作成
```js
// server.js
const express = require('express');
const next = require('next');

const port = 3000;
const dev = process.env.NODE_ENV !== 'production';
const app = next({dev});
const handle = app.getRequestHandler();
const server = express();

server.get('/another/:id', (req, res) => {
  const actualPage = '/another';
  const queryParams = {id: req.params.id};
  app.render(req, res, actualPage, queryParams);
});

server.get('*', (req, res) => {
  return handle(req, res)
})

app.prepare().then(() => {
  server.listen(port, (err) => {
    if(err) throw err
    console.log('NextJS is ready on https://localhost:' + port);
  });
}).catch(e => {
  console.error(e.stack);
  process.exit(1);
});
```

3. `package.json`の`start`を下記のように書き換えnodeサーバを起動
```json
{
  "scripts": {
    "start": "node server.js"
  }
}
```

4. 動作テスト
`npm start`し、`localhost:3000/another/1`にアクセスし、リロードしてみましょう。
```bash
npm start
```
正常にページが表示されるはずです。


## ダイナミックローディング
Next.jsはルーティングをベースとして複数の`chunk`に分割する`Code Splitting`に対応していますが、それだけでは十分とは言えません。
ここでは、必要なタイミングで動的に読み込みを行うダイナミックローディングについて説明します。

1. `dynamic`のimport
```bash
import dynamic from 'next/dynamic';
```

2. 任意の場所でロードする
下記のようにHogeコンポーネントをimportする際に、`dynamic`を呼びだすことで、Hogeコンポーネントが`render`されるまでロードされなくなります。
```js
import dynamic from 'next/dynamic';

const DynamicHoge = dynamic(import('../components/Hoge'))

export default class Page extends React.Component {
  state = {show: false};
  show = () => this.setState({show: true});
  render() {
    return (
	  this.state.show ? <DynamicHoge/> : <button onClick={this.show}>Show!</button>
	);
  }
}
```

3. ロードが完了するまで、ローディング表示する
```js
const loading = () => <div>Loading...</div>;
const DynamicHoge = dynamic(
  import('../components/Foo'),
  {loading}
);
```

4. 複数のコンポーネントを一度に読み込む
```js
const Bundle = dynamic({
  modules: props => ({
    Foo: import('../components/Foo'),
    Bar: import('../components/Hoge')
  }),
  render: (props, {Foo, Bar}) => (
    <div>
	  <h1>{props.title}</h1>
	  <Foo/>
	  <Bar/>
	</div>
  )
});

export default () => ( <Bundle title="Dynamic Bundle"/> );
```


## CSS in JS
コンポーネントにstyleをあてる方法はいろいろあります。  
著者のおすすめは、サードパーティ製の[`styled-components`](https://www.styled-components.com/ "styled-components")を使うことですが、ここでは`Next.js`で用意されている`JSS(CSS in JS)`の仕組みを紹介します。  
下記のように`<style jsx></style>`で囲むことで直接cssを記述することができます。このようにして記述したCSSのスコープは記述したCSS内に限定されるため、CSSでありがちなクラス間の依存関係が複雑になりメンテナンスコストが高くなってしまうといった問題が起こりにくく、1ファイルにまとまるという利点があります。  
＊ `<style jsx global></style>`のように`global`を付与することで、グローバルスコープにすることも可能です。
```js
// components/Btn.js
import React from "react";
import {withRouter} from "next/router";

const Btn = ({href, onClick, children, router}) => (
  <span>
    <button onClick={onClick} className={router.pathname === href ? 'selected' : ''}>
      { children }
    </button>
    <style jsx>{`
      button {
        color: blue;
        border: solid 1px;
        cursor: pointer;
      }

      button:hover {
        color: red;
      }

      button.selected {
        font-weight: bold;
      }
    `}</style>
  </span>
);

export default withRouter(Btn)
```

また、`Next.js`のバージョン5以上からWebpackローダープラグインの使用が可能になりました。
1. プラグインのインストール
```bash
npm install @zeit/next-css --save-dev
```

2. `next.config.js`に下記を追記する
```js
// next.config.js
const withCss = require('@zeit/next-css');
module.exports = withCss({});
```

3. カスタムドキュメントの追加
＊カスタムドキュメントについては2章で詳しく説明します。ここでは、`css`の部分に着目してください。
```js
// pages/_document.js
import Document, {Head, Main, NextScript} from 'next/document';

export default class MyDocument extends Document {
  render() {
    return (
      <html>
        <Head>
          <link rel="stylesheet" href="/_next/static/style.css"/>
          <title>NextJS</title>
        </Head>
        <body>
          <Main/>
          <NextScript/>
        </body>
      </html>
    )
  }
}
```

4. CSSファイルのimport
上記のようにすることで、`.css`ファイルを`.js`ファイルと同じようにインポートできるようになります。
```js
// components/Navigation.js
import React from "react";
import Btn from "./Btn";
import Link from "next/link";
import './Navigation.css';

export default () => (
  <nav>
    <Link href="/" passHref><Btn>Index</Btn></Link>
	<Link href="/another" passHref><Btn>Another</Btn></Link>
  </nav>
);
```

`Navigation.css`では変化がわかりやすく`background-color`を指定しましょう。

```css
// pages/Navigation.css
nav {
  background-color: #fbfbfb;
}
```


## 画像を表示する
画像を表示する方法もいろいろあります。ここでは3種類紹介します。

### CSSの`background`に指定
1. `static`ディレクトリーを作成し、`hoge.jpg`を格納する

2. CSSのbackgroundに指定
```css
// pages/Navigation.css
.logo {
  background: url(/static/hoge.jpg) no-repeat center center;
  background-size: cover;
}
```

3. classをあてる
```html
<span className="logo"/>
```

### コンポーネントに直接記述する
```html
<span src="/static/hoge.jpg"/>
```

### `next-images`を使う
1. `next-images`をインストール
```bash
npm install --save-dev next-images
```

2. `next.config.js`に下記を追記する
```js
// next.config.js
const withCss = require('@zeit/next-css');
const withImages = require('next-images');
module.exports = withImages(withCss({}));
```

3. 画像ファイルを`import`する
下記のように画像ファイルを`import`することで、画像そのものではなくその`URL`をインポートすることができます。
```js
import Image from '../static/hoge.js'

<img src={Image} alt="hoge"/>
```

次の章ではコンフィグについて学んでいきます。
