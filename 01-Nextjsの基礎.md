# Next.jsの基礎
この章では、Next.jsのインストールから基礎を説明します。

## Next.jsのインストール
最近では`create-next-app`(`create-react-app`ライクなコマンド一発で`Next.js`の環境を構築できる便利なツール)が開発されたことにより、簡単に環境構築することができます。  
```
npm install -g create-next-app
create-next-app sample-app
cd sample-app
npm run dev
```

`--example サンプル名`のオプションを付与することで、用途に応じた雛形を用意することもできます。例えば、Next.jsでReduxを使用したい場合、下記のように指定することで、Reduxのサンプルプロジェクトが生成されます。
```
create-next-app --example with-redux sample-redux-app
```
【参考URL】
- [GitHub - segmentio/create-next-app](https://github.com/segmentio/create-next-app)
- [Create Next App公式サイト](https://open.segment.com/create-next-app/)
- [オプション一覧](https://github.com/zeit/next.js/tree/master/examples)

今回は、`create-next-app`を使用せずに進めていきましょう。

```
mkdir next-js-app
cd next-js-app
npm init

// Next.jsのパッケージのインストール
npm install nextjs@latest --save-dev
npm install react@latest react-dom@latest --save
npm install next --save
```

プロジェクトをgit管理する場合は、下記のファイル・ディレクトリを`.gitignore`しましょう。
```
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
```
{
  "scripts": {
    "start": "next"
  }
}
```

pagesディレクトリを作成して、`npm start`します。
```
mkdir pages
npm start
```

ブラウザから`localhost:3000`にアクセスし、サーバが起動していることを確認しましょう。 
著者の環境だと`404 This page could not be found.` が表示されます。


## Next.jsのHello, world!
`pages`ディレクトリ以下に簡単なコンポーネントを作成しましょう。
```
// pages/index.js
import React from "react";
export default () => (<div>Hello, world!</div>);
```

`npm start`して`localhost:3000`にアクセスすると、`Hello, world!`と表示されているのが確認できます。


## Productionビルド
Next.jsを使用して本番運用するにあたって、静的なWebサイトの場合と動的なWebアプリケーションの場合とでビルドのスクリプトが異なります。

**静的なWebサイトの場合**
1. `package.json`に下記のように記述します。
```
{
  "scripts": {
    "build": "next build",
	"static": "next export"
  }
}
```

2. `next.config.js`にpathマップを記述します。
```
module.exports = {
  exportPathMap: () => ({
    '/': {page: '/'}
  })
}
```

3. 動作テスト
下記コマンドを実行しましょう
```
npm run build
npm run static
```

`npm run static`すると、`/out`ディレクトリが生成されるので、その中の`index.html`を開きましょう。
先ほど作成した`Hello, world!`が表示されるはずです。  
このようにしてProductionビルドすると、実体は静的なファイル(htmlファイル)になるため、サーバを選ばずデプロイすることができます。

**動的なWebサイトの場合**
1. `package.json`に下記のように記述します。
```
{
  "scripts": {
    "build": "next build",
	"server": "next start"
  }
}
```

2. 動作テスト
```
npm run build
npm run server
```


## ルーティング
Next.jsでURLに対応してページの出し分け(=ルーティング)を行いましょう。  
まず、下記のように`index.js`とは別のページを用意しましょう。
```
// pages/another.js
import React from "react";
export default () => (<div>Another</div>)
```

anotherページに遷移するリンクを`index.js`に記述しましょう。
```
// pages/index.js
import React from "react";
import Link from "next/link";
export default () => (<div><Link href="/another"><a>Another</a></Link></div>);
```

`aタグ`ではなく`Link`を使用することで、ページ全体のリロードが発生せず差分レンダリングされるため、高速化が期待できます。
さらなる高速化を行いたい場合、`Link`に`prefetch`を付与してください。
```
<Link prefetch>
```
`Link`の中に`aタグ`を記述したのは、ページ遷移(`Link`)とスタイル(`aタグ`)という二つの役割を分離するためです。  


続いて、2つのページで共通して使用するNavigationコンポーネントを作成し、ページ間の行き来をできるようにしましょう。

1. 現在のURLに応じて文字の太さが変化するコンポーネントを作成
```
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
```
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
```
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

```
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
```
// data/posts.js
export default [
  {title: 'Foo'},
  {title: 'Hoge'},
  {title: 'Bar'},
];
```

2. `index.js`に上記のページに対応する`Link`を記述します
```
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
```
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
```
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
