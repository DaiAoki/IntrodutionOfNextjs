# Next.jsのデータフロー
この章では、`Next.js`のデータフローについて説明します。
Reduxを導入し、データフローの理解を深めましょう。

## データの取得方法
クライアントサイドとサーバサイドでは、データの取得方法が異なります。
クライアントサイド(ブラウザ)では、`XMLHttpRequest`と`WhatWG Fetch API`使用しますが、サーバサイドでは`Node.js`(デフォルトでは`http`および`https`)を使用します。
クライアントサイドとサーバサイドでデータの取得方法を統一するには、ネットワーク層のライブラリの導入が必要です。

主な方法は2通りあります。

**WhatWG Fetch API**

1. `WhatWG Fetch API`の`pollyfill`をインストールする
```bash
npm install isomorphic-fetch --save
```

2. `fetch`する前に`isomorphic-fetch`を`import`する
```js
import 'isomorphic-fetch';

(async () => {
  const res = await fetch(...);
})();
```

**ネットワーク層をケアーしたライブラリの導入**
メジャーなものには、[`Axios`](https://github.com/axios/axios "Axios")や[`SuperAgent`](http://visionmedia.github.io/superagent/ "SuperAgent")があります。ここでは`Axios`を紹介します。

1. `Axios`をインストールする
```bash
npm install axios --save
```

2. `Axios`を使用する
```js
import axios from 'axios';

(async () => {
  const res = await axios.get(...);
})();
```


## `vanilla Next.js`を使用してリモートサーバからデータを取得する
通常の`React`アプリケーションの場合は、下記のように`componentWillMount`を使用して、コンポーネントがマウントされる前にデータを取得します。

```js
class Hoge extends React.Component {
  state = {data: null};

  loadData = async () => await (await fetch('...')).json();

  async componentWillMount() {
    const data = await this.loadData();
	this.setState({data});
  }

  render() {
    if(!this.state.data) return (<div>Loading...</div>);
	return (
	  <pre>
	    { JSON.stringify(this.state.data) }
	  </pre>
	);
  }
}
```

`Next.js`では`componentWillMount`の代わりに`getInitialProps`を使用します。
```js
class Hoge extends React.Component {

  loadData = async () => await (await fetch('...')).json();

  async static getInitialProps() {
    const data = await this.loadData();
	return {data}
  }

  render() {
    if(!this.props.data) return (<div>Loading...</div>);
	return (
	  <pre>
	    { JSON.stringify(this.props.data) }
	  </pre>
	);
  }
}
```

実際には、`ID`や`slug`を受け取ってダイナミックにデータを取得する場合が多いです。そのような場合には以下のように記述します。
```js
class Hoge extends React.Component {

  loadData = async (id) => await (await fetch('...')).json();

  async static getInitialProps() {
    const {id} = context.query;
    const data = await this.loadData(id);
	return {data}
  }

  render() {
    if(!this.props.data) return (<div>Loading...</div>);
	return (
	  <pre>
	    { JSON.stringify(this.props.data) }
	  </pre>
	);
  }
}
```

この時の`express`サーバのコードは下記のようになります。
```js
server.get('/post/:id', (req, res) => {
  const actualPage = '/another';
  const queryParams - {id: req.params.id};
  app.render(req, res, actualPage, queryParams);
});
```


## `Next.js`に`Redux`を導入する
`Redux`とは`Flux`アーキテクチャーの一種です。
アプリケーション全体の状態を一つの`Store`で管理し、ユーザーの操作によって発行された`Action`によって、`Reducer`が処理を`dispatch`し`Store`の状態を変更します。
`View`では`Store`の状態を`props`として受け取って、値に応じてコンポーネントの状態が変化するように宣言的に記述します。
Webのメジャーなアーキテクチャーである`MVC`との最も大きな違いは、データフローが双方向であるか(`MVC`)、一方向であるか(`Redux(Flux)`)です。

それでは、`Next.js`に`Redux`を導入してみましょう。

1. 必要なライブラリのインストール
```bash
npm install redux react-redux next@6 next-redux-wrapper react react-dom redux react-redux --save
```

2. `_app.js`を`HOC(Higher Order Component)`でラップする
```js
// pages/_app.js
import React from "react";
import {createStore} from "redux";
import withRedux from "next-redux-wrapper";

const reducer = (state = {hoge: ''}, action) => {
  switch (action.type) {
    case 'HOGE':
	  return {...state, hoge: action.payload};
	default:
	  return state;
  }
};

class MyApp extends React.Component {
  static async getInitialProps({ Component, ctx }) {
    ctx.store.dispatch({type: 'HOGE', payload: 'hoge'});

    if(Component.getInitialProps) return {pageProps: await Component.getInitialProps};
    return {};
  }

  render() {
    const {Component, pageProps = {}} = this.props;
    return (
      <Component {...pageProps} />
    );
  }
}

export default withRedux(makeStore)(MyApp);
```

3. 各コンポーネントではシンプルに`connect`することができる
```
// pages/index.js
import React, {Component} from "react";
import {connect} from "react-redux";

export default connect(
  (state) => ({hoge: state.hoge})
)(class extends Component {
  static getInitialProps({store, isServer, pathname, query}) {
    store.dispatch({type: 'HOGE', payload: 'hoge'});
	return {custom: 'custom'};
  }

  render() {
    return (
	  <div>
	    <p>{ this.props.hoge }</p>
		<p>{ this.props.custom }</p>
	  </div>
	)
  }
});
```

ソースコード全体を見たい場合は、下記の`Next.js`の公式リポジトリにサンプルのプロジェクトを立ち上げる方法が記載されているため、そちらに参照してください。  
[https://github.com/zeit/next.js/blob/master/examples/with-redux/README.md](https://github.com/zeit/next.js/blob/master/examples/with-redux/README.md "【公式】Next.js")
