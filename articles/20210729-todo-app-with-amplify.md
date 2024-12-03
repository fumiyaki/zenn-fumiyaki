---
title: "ToDoアプリをReact×Amplify(×色んなライブラリ)×Atomicデザインで実装する"
emoji: "😇"
type: "tech"
topics:
  - "react"
published: true
published_at: "2021-07-29 22:28"
---

:::message alert
この記事は執筆途中です。
あえて執筆途中を公開することで早い段階から間違い等に対応出来ると考えました。
また、現段階の知恵を絞って執筆していますのでこれが限界です。
「この記事を書き終えて俺はインターンの課題としてこれを提出するんだ。」
インターンに使われることが決まったら急に記事が非公開になるかもしれません、ごめんなさい。
:::

タイトルが長くて入らなかったので以下でタイトル宣言します。
# ToDoアプリをReact×TypeScript×Material-UI×Emotion×Apollo×date-fns×Amplify×Atomicデザインで実装する

この記事ではToDoリストを作りながら色んなライブラリ[^1][^2]を学んでいきます。
コードはgithubで管理しています。
[https://github.com/fumiyaki/todo_apps_with_atomic_design](https://github.com/fumiyaki/todo_apps_with_atomic_design)

最初に完成図を御覧ください。
![ToDoアプリの完成図１]()
![ToDoアプリの完成図２]()
![ToDoアプリの完成図３]()

ではプロジェクトを作成していきます。

## プロジェクトの雛形の作成

### プロジェクトのセットアップ
```terminal:プロジェクトの作成〜プロジェクトのスタートまで
$npx create-react-app todo_apps_with_atomic_design --template typescript
$cd todo_apps_with_atomic_design
$npm start
```
![プロジェクトの初期画像](https://storage.googleapis.com/zenn-user-upload/b0857bacf8ff377f272b522b.jpg)

初期のプロジェクトは以下の構成になっています。
```tmp:todo_apps_with_atomic_designのフォルダ構成
├── README.md
├── node_modules/
├── package-lock.json
├── package.json
├── public/
├── src/
│   ├── App.css
│   ├── App.test.tsx
│   ├── App.tsx
│   ├── index.css
│   ├── index.tsx
│   ├── logo.svg
│   ├── react-app-env.d.ts
│   ├── reportWebVitals.ts
│   └── setupTests.ts
└── tsconfig.json
```
ここからAtomicデザインの構成で作るためのファイルとフォルダを作成していきます。
また、今回使わないファイルも削除します。
```diff:Atomicデザイン用の構成
  ├── src/
  │   ├── components/
+ │   │   ├── atoms/
+ │   │   │   └── index.ts
+ │   │   ├── molecules/
+ │   │   │   └── index.ts
+ │   │   ├── organisms/
+ │   │   │   └── index.ts
+ │   │   ├── pages/
+ │   │   │   └── index.ts
+ │   │   └── templates/
+ │   │       └── index.ts
- │   ├── App.css
- │   ├── App.test.tsx
  │   ├── App.tsx
- │   ├── index.css
  │   ├── index.tsx
- │   ├── logo.svg
  │   ├── react-app-env.d.ts
- │   ├── reportWebVitals.ts
- │   └── setupTests.ts
  └── tsconfig.json
```

### Hello World
先程のファイル・フォルダの削除でindex.tsxにErrorが出ています。
それらを解消しつつ、必要ない行も削除してHello Worldを表示させましょう。

```diff react:index.tsx
import React from 'react';
import ReactDOM from 'react-dom';
- import './index.css';
import App from './App';
- import reportWebVitals from './reportWebVitals';

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);

- // If you want to start measuring performance in your app, pass a function
- // to log results (for example: reportWebVitals(console.log))
- // or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
- reportWebVitals();
```

```diff react: App.tsx
- import React from 'react';
- import logo from './logo.svg';
- import './App.css';
- 
- function App() {
+ const App = () => {
  return (
    <div className="App">
+     Hello World
-       <header className="App-header">
-         <img src={logo} className="App-logo" alt="logo" />
-         <p>
-           Edit <code>src/App.tsx</code> and save to reload.
-         </p>
-         <a
-           className="App-link"
-           href="https://reactjs.org"
-           target="_blank"
-           rel="noopener noreferrer"
-         >
-           Learn React
-         </a>
-       </header>
    </div>
  );
}

export default App;

```

これでシンプルなコードのみになり、Hello Worldも表示されました。
![Hello Worldと書かれた画面](https://storage.googleapis.com/zenn-user-upload/3c71eca65b1560ae981e6a59.jpg)
:::message
エラーで困っている方は[こちら](https://qiita.com/ryokkkke/items/390647a7c26933940470#isolatedmodules)を参照して自力で頑張ってください。
:::


### ライブラリのインストール
最後にプロジェクトで使用するライブラリをインストールします。
先程起動したプログラムを```Control+C```で止めて以下のコマンドでインストールしていきます。

```terminal:ライブラリのインストール
npm i @emotion/react @apollo/client graphql date-fns @material-ui/core recharts aws-amplify
```

これで開発準備が出来ましたので次回の更新でToDoアプリを作っていきます。
では、今日は風呂に入って寝ます。おやすみなさい。




[^1]: 使用ライブラリ
[React](https://github.com/facebook/react)
[TypeScript](https://github.com/microsoft/TypeScript)
[Material UI](https://github.com/mui-org/material-ui)
[Emotion](https://github.com/emotion-js/emotion/tree/main/packages/react)
[Apollo](https://github.com/apollographql/apollo-client)
[date-fns](https://github.com/date-fns/date-fns)
[recharts](https://github.com/recharts/recharts)
[amplify](https://github.com/aws-amplify/amplify-js)

[^2]: 使用バージョン
"dependencies": {
    "@apollo/client": "^3.3.21",
    "@emotion/react": "^11.4.0",
    "@material-ui/core": "^4.12.2",
    "@testing-library/jest-dom": "^5.14.1",
    "@testing-library/react": "^11.2.7",
    "@testing-library/user-event": "^12.8.3",
    "@types/jest": "^26.0.24",
    "@types/node": "^12.20.17",
    "@types/react": "^17.0.15",
    "@types/react-dom": "^17.0.9",
    "aws-amplify": "^4.2.2",
    "date-fns": "^2.23.0",
    "graphql": "^15.5.1",
    "react": "^17.0.2",
    "react-dom": "^17.0.2",
    "react-scripts": "4.0.3",
    "typescript": "^4.3.5",
    "web-vitals": "^1.1.2"
  },