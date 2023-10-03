---
title: "【React/GraphQL】Amplifyを使ってGraphQL環境の構築ハンズオン vol.2 | Offers Tech Blog"
emoji: "🖋️"
type: "tech"
topics:
  - "graphql"
  - "react"
  - "amplify"
published: true
published_at: "2022-11-24 09:00"
publication_name: "overflow_offers"
---

# はじめに

こんにちは！

プロダクト開発人材の副業転職プラットフォーム [Offers](https://offers.jp/) を運営する株式会社 [overflow](https://overflow.co.jp/) で Offers のフロントエンドを開発している [fumiya](https://twitter.com/fumiya_itsc) です。

10 月 1 日から Flexible メンバーとして週 3 で働かせていただいており、翌月の 11 月 16 日からは正式に Full メンバーとして参画させていただきます！

# 概要

この記事は React と [Amplify](https://aws.amazon.com/jp/amplify/) を使って開発するための環境を整える記事になります。


分量が多くなったため、[vol.1](https://zenn.dev/offers/articles/20221121-hello-amplify-react-vol1) と vol2 に分けてお送りいたします。

React と Amplify だけでも開発を始めることが出来ますが、効率的に開発を進めるために [GraphQL Code Generator](https://www.the-guild.dev/graphql/codegen) や [TanStack Query](https://tanstack.com/query/v4/) を取り入れ、
似たようなコードを減らし、開発者体験を良くすることが出来ます。

これらのライブラリを用いて、少しでも楽に開発していくことが出来るように開発環境を事前に整えることがこの記事のゴールになります。

流れは以下になります。

1. Amplify API と Auth を使って GraphQL サーバーを構築(vol.1)
1. React hooks を使い、GraphQL サーバーからデータを取得して画面に表示する(vol.1)
1. GraphQL サーバーからのデータ取得を GraphQL Code Generator と TanStack Query を使って宣言的に書く(本記事)

ハンズオンで登場する技術は以下になります。

- Amplify API
- Amplify Auth
- React
- GraphQL Code Generator
- TanStack Query

:::message
React を用いて進めますが紹介するライブラリは様々なフロントエンドライブラリ・フレームワークに対応しているため、ご自身の環境に合わせて読み替えていただけたらと思います。
:::

## 1. TanStack Query の導入

さっそく TanStack Query の導入したいところですが vol.1 で少し複雑になった前回のコードをリファクタリングをしておきましょう。

ここでは TodoList.tsx を作り、コンポーネント毎の役割を分けます。

### リファクタリング

```:terminal
touch src/TodoList.tsx
```

Todo のデータ取得や描画周りは TodoList.tsx に引っ越します。

```jsx:src/TodoList.tsx
import { GraphQLResult } from "@aws-amplify/api-graphql";
import { listTodos } from "./graphql/queries";
import { useEffect, useState } from "react";
import { ListTodosQuery } from "./API";
import { API } from "aws-amplify";

export const TodoList = () => {
  const [todoList, setTodoList] = useState<ListTodosQuery | undefined>();
  useEffect(() => {
    const fetchTodos = async () => {
      const todos = (await API.graphql({
        query: listTodos,
      })) as GraphQLResult<ListTodosQuery>;
      setTodoList(todos.data);
    };
    fetchTodos();
  }, []);
  return (
    <div>
      {todoList?.listTodos?.items.map((item) => {
        return (
          <>
            <div>id: {item?.id}</div>
            <div>name: {item?.name}</div>
            <div>desc: {item?.description}</div>
            <div>createdAt: {item?.createdAt}</div>
            <div>updatedAt: {item?.updatedAt}</div>
            <br />
          </>
        );
      })}
    </div>
  );
};
```

```diff jsx:src/App.tsx
- import { Amplify, API, Auth } from "aws-amplify";
- import { GraphQLResult } from "@aws-amplify/api-graphql";
- import { listTodos } from "./graphql/queries";
- import { useEffect, useState } from "react";
- import { ListTodosQuery } from "./API";
+ import { Amplify, Auth } from "aws-amplify";
+ import { TodoList } from "./TodoList";

import { withAuthenticator } from "@aws-amplify/ui-react";
import "@aws-amplify/ui-react/styles.css";

import awsExports from "./aws-exports";
Amplify.configure(awsExports);

const App = () => {
-   const [todoList, setTodoList] = useState<ListTodosQuery | undefined>();
-   useEffect(() => {
-     const fetchTodos = async () => {
-       const todos = (await API.graphql({
-         query: listTodos,
-       })) as GraphQLResult<ListTodosQuery>;
-       setTodoList(todos.data);
-     };
-     fetchTodos();
-   }, []);
-
-   return (
-     <>
-       <h1>Hello Amplify</h1>
-       {todoList?.listTodos?.items.map((item) => {
-         return (
-           <>
-             <div>id: {item?.id}</div>
-             <div>name: {item?.name}</div>
-             <div>desc: {item?.description}</div>
-             <div>createdAt: {item?.createdAt}</div>
-             <div>updatedAt: {item?.updatedAt}</div>
-             <br />
-           </>
-         );
-       })}
+   return (
+     <>
+       <h1>Hello Amplify and TanStack Query</h1>
+       <TodoList />
      <button
        onClick={() => {
          Auth.signOut();
        }}
      >
        Sign out
      </button>
    </>
  );
};

export default withAuthenticator(App);
```

これで App.tsx から Todo 関連のコードを分離できました。

綺麗に分離できたところで TanStack Query（元 React Query）を導入していきましょう。

### TanStack Query の導入

TanStack Query を導入することによりサーバーから取得するデータを宣言的に扱えるようになり、コードの見通しが良くなります。

まずは必要なライブラリをインストールしましょう。

```:terminal
npm i @tanstack/react-query
```

次に TanStack Query をアプリ全体で使えるように QueryClient のインスタンスを作り、App コンポーネントを QueryClientProvider で括ってあげましょう。

```diff jsx:src/App.tsx
import { Amplify, Auth } from "aws-amplify";
import { TodoList } from "./TodoList";

+ import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

import { withAuthenticator } from "@aws-amplify/ui-react";
import "@aws-amplify/ui-react/styles.css";

import awsExports from "./aws-exports";
Amplify.configure(awsExports);

+ const queryClient = new QueryClient();

const App = () => {
  return (
-     <>
+     <QueryClientProvider client={queryClient}>
        <h1>Hello Amplify and TanStack Query</h1>
        <TodoList />
        <button
          onClick={() => {
            Auth.signOut();
          }}
        >
          Sign out
        </button>
+     </QueryClientProvider>
-     </>
  );
};

export default withAuthenticator(App);
```

TanStack Query を使う準備が出来たので、早速 Todo の一覧を TanStack Query を使用して取得してみましょう。

```diff jsx:src/TodoList.tsx
import { GraphQLResult } from "@aws-amplify/api-graphql";
import { listTodos } from "./graphql/queries";
- import { useEffect, useState } from "react";
import { ListTodosQuery } from "./API";
import { API } from "aws-amplify";
+ import { useQuery } from "@tanstack/react-query";

+ const fetchTodos = async () => {
+   const todos = (await API.graphql({
+     query: listTodos,
+   })) as GraphQLResult<ListTodosQuery>;
+   return todos;
+ };

export const TodoList = () => {
-   const [todoList, setTodoList] = useState<ListTodosQuery | undefined>();
-   useEffect(() => {
-     const fetchTodos = async () => {
-       const todos = (await API.graphql({
-         query: listTodos,
-       })) as GraphQLResult<ListTodosQuery>;
-       setTodoList(todos.data);
-     };
-     fetchTodos();
-   }, []);
  const todoList = useQuery(["todoList"], fetchTodos);

  return (
    <div>
-     {todoList?.listTodos?.items.map((item) => {
-       return (
+     {todoList.data?.data?.listTodos?.items.map((item) => (
        <>
          <div>id: {item?.id}</div>
          <div>name: {item?.name}</div>
          <div>desc: {item?.description}</div>
          <div>createdAt: {item?.createdAt}</div>
          <div>updatedAt: {item?.updatedAt}</div>
          <br />
        </>
-       );
-     })}
+     ))}
    </div>
  );
};
```

TanStack Query を導入することによって useState や useEffect が TodoList コンポーネントから消えて、`useQuery(["todoList"], fetchTodos)` に集約されました。
ここに GraphQL Code Generator を導入して柔軟なデータ操作を実現しましょう。

## 2. GraphQL Code Generator の導入

次は GraphQL Code Generator を導入していきます。

必要なパッケージをインストールします。

```:terminal
npm install graphql
npm install -D @graphql-codegen/client-preset @graphql-codegen/cli @graphql-codegen/introspection @graphql-codegen/typescript-react-query
```

次に `graphql-code-generator init` を実行して必要なファイルを生成します。

```:terminal
amplify_graphqlcodegenerator_template ❯❯❯ npx graphql-code-generator init

    Welcome to GraphQL Code Generator!
    Answer few questions and we will setup everything for you.

? What type of application are you building? Application built with React
? Where is your schema?: (path or url) src/graphql/schema.json
? Where are your operations and fragments?: src/graphql-codegen/**/*.graphql
? Where to write the output: src/generated/graphql.ts
? Do you want to generate an introspection file? Yes
? How to name the config file? codegen.ts
? What script in package.json should run the codegen? codegen
Fetching latest versions of selected plugins...

    Config file generated at codegen.ts

      $ npm install

    To install the plugins.

      $ npm run codegen

    To run GraphQL Code Generator.
```

生成された codegen.ts を開き、plugin を追加します。

:::message alert
筆者の環境で試したところ、`preset: "client"` の設定で生成される文と後ほど加える `plugin` で生成されるコードが競合してしまうバグ？があったため、ここでは回避策として `preset: "client"` を削除しています。
また、`preset: "client"` で生成されるコードについて調べたのですが正直よく分かっておらず有識者の方がいたら教えていただけると嬉しいです。
:::

```diff js:codegen.js
import type { CodegenConfig } from "@graphql-codegen/cli";

const config: CodegenConfig = {
  overwrite: true,
  schema: "./src/graphql/schema.json",
  documents: "src/graphql-codegen/**/*.graphql",
  generates: {
    "src/generated/graphql.ts": {
-     preset: "client",
      plugins: [
+       "typescript",
+       "typescript-operations",
+       "typescript-react-query",
      ],
    },
    "./graphql.schema.json": {
      plugins: ["introspection"],
    },
  },
};

export default config;
```

次に graphql のコードを書いていきます。
graphql-codegen フォルダーを作り、graphql ファイルを作ります。

```:terminal
mkdir src/graphql-codegen
touch src/graphql-codegen/todo.graphql
```

todo.graphql に取得したいデータを書いていきましょう。

ちなみに筆者は Amplify で generate された src/graphql/内のファイルから必要なモノをコピペして必要なフィールドを残すように修正して使っています。

```graphql:src/graphql-codegen/todo.graphql
query ListTodos(
  $filter: ModelTodoFilterInput
  $limit: Int
  $nextToken: String
) {
  listTodos(filter: $filter, limit: $limit, nextToken: $nextToken) {
    items {
      id
      name
      description
      createdAt
      updatedAt
    }
    nextToken
  }
}
```

これで大方準備が整いました。

GraphQL Code Generator でコードを生成してみましょう。

```:terminal
npm run codegen
```

これで generated/graphql.ts が生成されています。

```diff:tree
amplify_graphqlcodegenerator_template ❯❯❯ tree ./src -L 2
./src
├── API.ts
├── App.css
├── App.test.tsx
├── App.tsx
├── TodoList.tsx
├── aws-exports.js
+├── generated
+│   └── graphql.ts
├── graphql
│   ├── mutations.ts
│   ├── queries.ts
│   ├── schema.json
│   └── subscriptions.ts
├── graphql-codegen
│   ├── customFetcher.ts
│   └── todo.graphql
├── index.css
├── index.tsx
├── logo.svg
├── react-app-env.d.ts
├── reportWebVitals.ts
└── setupTests.ts
```

生成された graphql.ts を見てみましょう。
先程定義した `todo.graphql` を元に `useListTodosQuery` 関数が作られていることが確認できます。

```js:src/generated/graphql.ts
~~~~~~~~~~~~省略~~~~~~~~~~~~
export const useListTodosQuery = <TData = ListTodosQuery, TError = unknown>(
  dataSource: { endpoint: string; fetchParams?: RequestInit },
  variables?: ListTodosQueryVariables,
  options?: UseQueryOptions<ListTodosQuery, TError, TData>
) =>
  useQuery<ListTodosQuery, TError, TData>(
    variables === undefined ? ["ListTodos"] : ["ListTodos", variables],
    fetcher<ListTodosQuery, ListTodosQueryVariables>(
      dataSource.endpoint,
      dataSource.fetchParams || {},
      ListTodosDocument,
      variables
    ),
    options
  );
```

さっそく使ってみたいところですが、この useOOO を使うたびに `dataSource: { endpoint: string; fetchParams?: RequestInit }` を要求されるのは面倒です。

graphql code generator の custom fetcher の機能を使い、`dataSource` を毎回書かなくても済むようにしていきましょう。

```:terminal
touch src/graphql-codegen/customFetcher.ts
```

GraphQL サーバーからデータを取得する処理を書いていきます。

```js:src/graphql-codegen/customFetcher.ts
import { API, GraphQLResult } from "@aws-amplify/api";

export const fetchWithAmplify = <TData, TVariables>(
  query: string,
  variables?: TVariables
): (() => Promise<TData>) => {
  return async () => {
    const result = await (API.graphql({
      query,
      variables: variables || {},
      authMode: "AMAZON_COGNITO_USER_POOLS",
    }) as Promise<GraphQLResult<TData>>);

    if (result.errors) {
      const message = result.errors
        ? result.errors[0].message
        : "GraphQL fetching error";
      throw new Error(message);
    }

    return result.data!;
  };
};
```

`@aws-amplify/api` を使うことによって認証周りで楽が出来るようになります。

次に custom fetcher を使うことを graphql code generator の設定に書いていきます。

```js diff:codegen.ts
import type { CodegenConfig } from "@graphql-codegen/cli";

const config: CodegenConfig = {
  overwrite: true,
  schema: "src/graphql/schema.json",
  documents: "src/graphql-codegen/**/*.graphql",
  generates: {
    "src/generated/graphql.ts": {
      plugins: [
        "typescript",
        "typescript-operations",
        "typescript-react-query",
      ],
+     config: {
+       fetcher: "../graphql-codegen/customFetcher#fetchWithAmplify",
+     },
    },
    "./graphql.schema.json": {
      plugins: ["introspection"],
    },
  },
};

export default config;
```

気をつけたいのが fetcher の value は config からみた相対パスではなく、生成される `src/generated/graphql.ts` からみた相対パスであることと、#以降に自分で実装した関数の名前を書くことです。

ここまで出来たら再度 graphql code generator を動かしてコードを生成してみましょう。

```:terminal
npm run codegen
```

生成されたコードをみてみると、実装した `fetchWithAmplify` が使用されていることが確認できます。

```js:src/graphql-codegen/customFetcher.ts
import { useQuery, UseQueryOptions } from '@tanstack/react-query';
import { fetchWithAmplify } from '../graphql-codegen/customFetcher';

~~~~~~~~~~~~省略~~~~~~~~~~~~
export const useListTodosQuery = <
      TData = ListTodosQuery,
      TError = unknown
    >(
      variables?: ListTodosQueryVariables,
      options?: UseQueryOptions<ListTodosQuery, TError, TData>
    ) =>
    useQuery<ListTodosQuery, TError, TData>(
      variables === undefined ? ['ListTodos'] : ['ListTodos', variables],
      fetchWithAmplify<ListTodosQuery, ListTodosQueryVariables>(ListTodosDocument, variables),
      options
    );
```

最後に `TodoList.tsx` 内で `useListTodosQuery` を使ってデータが取得出来ているかを確認してみましょう。

```js diff:src/TodoList.tsx
+import { useListTodosQuery } from "./generated/graphql";
-import { GraphQLResult } from "@aws-amplify/api-graphql";
-import { listTodos } from "./graphql/queries";
-import { ListTodosQuery } from "./API";
-import { API } from "aws-amplify";
-import { useQuery } from "@tanstack/react-query";
-
-const fetchTodos = async () => {
-  const todos = (await API.graphql({
-    query: listTodos,
-  })) as GraphQLResult<ListTodosQuery>;
-  return todos;
-};


export const TodoList = () => {
- const todoList = useQuery(["todoList"], fetchTodos);
+ const { data } = useListTodosQuery();

  return (
    <div>
-     {todoList.data?.data?.listTodos?.items.map((item) => (
+     {data?.listTodos?.items.map((item) => (
        <>
          <div>id: {item?.id}</div>
          <div>name: {item?.name}</div>
          <div>desc: {item?.description}</div>
          <div>createdAt: {item?.createdAt}</div>
          <div>updatedAt: {item?.updatedAt}</div>
          <br />
        </>
      ))}
    </div>
  );
};
```

![Hello Amplifyが表示された画面の画像](/images/20221124-hello-amplify-react-vol2/01.png)

表示は今までと変わらないものの `TodoList.tsx` が `useListTodosQuery` だけに依存するようになり、とても見通しが良くなりました。

# まとめ

開発者体験を良くするために GraphQL Code Generator と TanStack Query を Amplify と React の環境に導入してきました。

開発を進めていく中でリストの表示だけでなく Todo の作成も必要になると思います。

そういうときは `src/graphql-codegen/todo.graphql` に mutation を書いてあげれば `useCreateTodoMutation` が簡単に手に入ります。

このサイクルで開発を進めていけば効率的に保守性の高いコードを書くことが出来ると思います。

今回の記事が皆さんの参考になれば幸いです。

最後にはなりますが、本記事を最後まで読んで頂き、ありがとうございました。「**こんな記事を書いてほしい！**」などありましたらコメントいただけると幸いです。


# 関連記事

https://zenn.dev/offers/articles/20221017-definition-of-frontend
https://zenn.dev/offers/articles/20220523-component-design-best-practice
https://zenn.dev/offers/articles/20220418-what-is-bff-architecture

[^1]: Amplify CLI の設定はこちらを参考にしてください。https://docs.amplify.aws/cli/start/install/#option-2-follow-the-instructions/
[^2]: データにアクセス出来るユーザーを制御したい場合はこちらを参考にしてください。https://docs.amplify.aws/cli/graphql/authorization-rules/
[^3]: データの CRUD 操作はこちらを参考にしてください。https://docs.amplify.aws/lib/graphqlapi/mutate-data/q/platform/js/
