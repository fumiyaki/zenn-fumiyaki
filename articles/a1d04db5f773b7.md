---
title: "AmplifyとGraphQL Code Generatorを使ってお手軽GraphQL環境の構築ハンズオン"
emoji: "🤖"
type: "tech"
topics:
  - "react"
  - "reactquery"
published: false
---

こんにちは！
Offers を運営している株式会社 overflow の[fumiya](https://twitter.com/fumiya_itsc)です。

10月1日からFlexibleメンバーとして週3で働かせていただいております！
11月16日からは正式にFullメンバーとして働きます！

前職でAmplifyを使っていたので忘れてしまわないうちに棚卸しするためこの記事を書きます。

今回のハンズオンで登場する技術は以下になります。
- Amplify API
- Amplify Auth
- React
- GraphQL Code Generator
- @tanstack/react-query


# フロントエンドの準備
AmplifyやGraphQL Code Generatorは様々なフロントエンドライブラリ・フレームワークに対応しているため、ご自身の環境に合わせて読み替えていただけたらと思います。
この記事ではReactで進めていきます。

## React init
```:terminal
❯❯❯ npx create-react-app amplify_graphqlcodegenerator_template --template typescript

Creating a new React app in amplify_graphqlcodegenerator_template.

Installing packages. This might take a couple of minutes.
Installing react, react-dom, and react-scripts with cra-template-typescript...
~~~~~~~~~~~~省略~~~~~~~~~~~~
We suggest that you begin by typing:

  cd amplify_graphqlcodegenerator_template
  npm start

Happy hacking!
```

# Amplifyの導入
次はAmplifyの環境を作っていきます。[^1]

## IAMユーザーの作成
Amplify　CLIでリソースをごにょごにょするためにAmplify用のIAMユーザーを作ります。
`アクセスキー - プログラムによるアクセス`にチェックを入れ、次に進みます。（名前は何でも大丈夫です。）
`既存のポリシーを直接アタッチ`から`AdministratorAccess-Amplify`にチェックを入れて、ユーザーの作成をします。
後ほど使うのでアクセスキーとIDシークレットアクセスキーをメモしておいてください。

## Amplify init
プロジェクトにAmplifyを導入していきます。
先程メモしたアクセスキーとIDシークレットアクセスキーは`Select the authentication method you want to use:AWS access keys`の箇所で使用します。
```:terminal
amplify_graphqlcodegenerator_template ❯❯❯ amplify init             ✘ 2 
Note: It is recommended to run this command from the root of your app directory
? Enter a name for the project amplifygraphqlcodege
The following configuration will be applied:

Project information
| Name: amplifygraphqlcodege
| Environment: dev
| Default editor: Visual Studio Code
| App type: javascript
| Javascript framework: react
| Source Directory Path: src
| Distribution Directory Path: build
| Build Command: npm run-script build
| Start Command: npm run-script start

? Initialize the project with the above configuration? Yes
Using default provider  awscloudformation
? Select the authentication method you want to use: AWS access keys
? accessKeyId:  ********************
? secretAccessKey:  ****************************************
? region:  ap-northeast-1

~~~~~~~~~~~~省略~~~~~~~~~~~~

✔ Successfully created initial AWS cloud resources for deployments.
✔ Help improve Amplify CLI by sharing non sensitive configurations on failures (y/N) · no

✔ Initialized provider successfully.
✅ Initialized your environment successfully.

Your project has been successfully initialized and connected to the cloud!

Some next steps:
"amplify status" will show you what you've added already and if it's locally configured or deployed
"amplify add <category>" will allow you to add features like user login or a backend API
"amplify push" will build all your local backend resources and provision it in the cloud
"amplify console" to open the Amplify Console and view your project status
"amplify publish" will build all your local backend and frontend resources (if you have hosting category added) and provision it in the cloud

Pro tip:
Try "amplify add api" to create a backend API and then "amplify push" to deploy everything
```
これでプロジェクトにAmplifyを導入できました！

## AmplifyAPI init
次はAmplify　APIの追加をしていきます。
Authorization modesを`API key (default, expiration time: 7 days from now)`から`Amazon Cognito User Pool`に変更するところだけ間違えずに出来れば後はデフォルトの設定でどんどん進めていく事ができます。
```:terminal
amplify_graphqlcodegenerator_template ❯❯❯ amplify add api
? Select from one of the below mentioned services: GraphQL
? Here is the GraphQL API that we will create. Select a setting to edit or continue Authorization modes: API key (default, expiration time: 7 days f
rom now)
? Choose the default authorization type for the API Amazon Cognito User Pool
Using service: Cognito, provided by: awscloudformation
 
 The current configured provider is Amazon Cognito. 
 
 Do you want to use the default authentication and security configuration? Default configuration
 Warning: you will not be able to edit these selections. 
 How do you want users to be able to sign in? Email
 Do you want to configure advanced settings? No, I am done.
✅ Successfully added auth resource amplifygraphqlcodege5a93a4de locally

✅ Some next steps:
"amplify push" will build all your local backend resources and provision it in the cloud
"amplify publish" will build all your local backend and frontend resources (if you have hosting category added) and provision it in the cloud

? Configure additional auth types? No
? Here is the GraphQL API that we will create. Select a setting to edit or continue Continue
? Choose a schema template: Single object with fields (e.g., “Todo” with ID, name, description)

⚠️  WARNING: Some types do not have authorization rules configured. That means all create, read, update, and delete operations are denied on these types:
         - Todo
Learn more about "@auth" authorization rules here: https://docs.amplify.aws/cli/graphql/authorization-rules
✅ GraphQL schema compiled successfully.

Edit your schema at /Users/fumiya/programing/react/amplify_graphqlcodegenerator_template/amplify/backend/api/amplifygraphqlcodege/schema.graphql or place .graphql files in a directory at /Users/fumiya/programing/react/amplify_graphqlcodegenerator_template/amplify/backend/api/amplifygraphqlcodege/schema
✔ Do you want to edit the schema now? (Y/n) · no
✅ Successfully added resource amplifygraphqlcodege locally

✅ Some next steps:
"amplify push" will build all your local backend resources and provision it in the cloud
"amplify publish" will build all your local backend and frontend resources (if you have hosting category added) and provision it in the cloud

```
`amplify add api`の中でschemaのテンプレートにTodoを選択しました。
どういったものがものが生成されたか見てみましょう。
```graphql:amplify_graphqlcodegenerator_template/amplify/backend/api/amplifygraphqlcodege/schema.graphql
# This "input" configures a global authorization rule to enable public access to
# all models in this schema. Learn more about authorization rules here: https://docs.amplify.aws/cli/graphql/authorization-rules
input AMPLIFY { globalAuthRule: AuthRule = { allow: public } } # FOR TESTING ONLY!

type Todo @model {
  id: ID!
  name: String!
  description: String
}
```
後でTodoを登録したいので、以下のように変更しておきます。
```diff graphql:amplify_graphqlcodegenerator_template/amplify/backend/api/amplifygraphqlcodege/schema.graphql
- # This "input" configures a global authorization rule to enable public access to
- # all models in this schema. Learn more about authorization rules here: https://docs.amplify.aws/cli/graphql/authorization-rules
- input AMPLIFY {
-   globalAuthRule: AuthRule = { allow: public }
- } # FOR TESTING ONLY!
- type Todo @model {
+ type Todo @model @auth(rules: [{ allow: owner }]) {
  id: ID!
  name: String!
  description: String
}
```
@auth周りを詳しく知りたい場合は[こちら](https://docs.amplify.aws/cli/graphql/authorization-rules/)を参考にしてください。

ここまででAmplifyを使ってGraphQLサーバーを作成する準備が整いました。
実際にAWSにデプロイしてみましょう。
```:terminal
amplify_graphqlcodegenerator_template ❯❯❯ amplify push
~~~~~~~~~~~~省略~~~~~~~~~~~~
    Current Environment: dev
    
┌──────────┬──────────────────────────────┬───────────┬───────────────────┐
│ Category │ Resource name                │ Operation │ Provider plugin   │
├──────────┼──────────────────────────────┼───────────┼───────────────────┤
│ Auth     │ amplifygraphqlcodege5a93a4de │ Create    │ awscloudformation │
├──────────┼──────────────────────────────┼───────────┼───────────────────┤
│ Api      │ amplifygraphqlcodege         │ Create    │ awscloudformation │
└──────────┴──────────────────────────────┴───────────┴───────────────────┘
? Are you sure you want to continue? Yes
~~~~~~~~~~~~省略~~~~~~~~~~~~
✔ Would you like to create an API Key? (y/N) · no
~~~~~~~~~~~~省略~~~~~~~~~~~~
? Do you want to generate code for your newly created GraphQL API Yes
? Choose the code generation language target typescript
? Enter the file name pattern of graphql queries, mutations and subscriptions src/graphql/**/*.ts
? Do you want to generate/update all possible GraphQL operations - queries, mutations and subscriptions Yes
? Enter maximum statement depth [increase from default if your schema is deeply nested] 2
? Enter the file name for the generated code src/API.ts
~~~~~~~~~~~~省略~~~~~~~~~~~~
✔ Generated GraphQL operations successfully and saved at src/graphql
✔ Code generated successfully and saved in file src/API.ts
✔ All resources are updated in the cloud

GraphQL endpoint: https://wr7antkkkzayvd6pugpfwfj3gy.appsync-api.ap-northeast-1.amazonaws.com/graphql

GraphQL transformer version: 2
```
少し時間がかかりますが、終わったらGraphQLサーバーが完成しています。
`amplify_graphqlcodegenerator_template/amplify/backend/api/amplifygraphqlcodege/schema.graphql`にあるSchemaを編集することでGraphQLのデータ構造を編集できます！

## フロントエンドとAmplifyの紐付け
[Amplify公式ページ](https://docs.amplify.aws/lib/auth/getting-started/q/platform/js/#configure-your-application)を参考にフロントとAmplifyの紐付けを行っていきます。
まずは必要なライブラリをインストールしましょう。
Amplifyが用意しているコンポーネントも使いたいので`@aws-amplify/ui-react`を一緒にインストールしておきます。
```:terminal
npm install aws-amplify @aws-amplify/ui-react
```
インストールが完了したらconfiguration fileを読み込むようにApp.tsxのコードを追加します。
また、認証周りを楽出来るように`withAuthenticator`も使用しておきます。
```jsx:src/App.tsx
import { Amplify, Auth } from "aws-amplify";

import { withAuthenticator } from "@aws-amplify/ui-react";
import "@aws-amplify/ui-react/styles.css";

import awsExports from "./aws-exports";
Amplify.configure(awsExports);

const App = () => {
  return (
    <>
      <h1>Hello Amplify</h1>
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
これで一度`npm start`してみましょう。
![npm startで初めに表示される画面の画像](https://storage.googleapis.com/zenn-user-upload/29655028afb4-20221014.png)
適当にアカウントを作成してログインするとHello Amplifyが表示されます。
![新規登録後のTop画面の画像](https://storage.googleapis.com/zenn-user-upload/5afffe4a1bbb-20221014.png)

## 実際にデータを入れてうまく表示出来るかの確認
ここまででフロントの準備とAmplifyを使ってGraphQLサーバーを作ることが出来ました。
実際にAWSにデータを持たせてフロント側で表示をさせてみましょう。

まずはAWSコンソールを操作してデータを保存します。
AWS AppSyncの画面を開きましょう。
![AWS AppSyncの画面](https://storage.googleapis.com/zenn-user-upload/eec3fc92a734-20221014.png)
クエリを実行ボタンを押します。
![](https://storage.googleapis.com/zenn-user-upload/de3492eb48df-20221014.png)
ユーザープールでログインボタンを押し、先程アカウント登録したユーザーの情報でログインします。（ClientIdはどちらを選んでも大丈夫です）
![](https://storage.googleapis.com/zenn-user-upload/d156b25fbada-20221014.png)
ログインが出来たらデータを登録しましょう。
```:AWS AppSync
mutation m {
  createTodo(input: {name: "todo1", description: "todo1 desc"}) {
    name
    updatedAt
    id
    description
    createdAt
  }
}
```
![](https://storage.googleapis.com/zenn-user-upload/87aeca302b45-20221014.png)
再生ボタンを押すとinputに渡したデータで登録が出来ます。
![](https://storage.googleapis.com/zenn-user-upload/bd2dbdfc2b85-20221014.png)
これでサーバー側の準備が整いました。
それではフロント側で今登録したデータが表示出来るかを試してみましょう。
```diff jsx:src/App.tsx
- import { Amplify, Auth } from "aws-amplify";
+ import { Amplify, API, Auth } from "aws-amplify";
+ import { GraphQLResult } from "@aws-amplify/api-graphql";
+ import { listTodos } from "./graphql/queries";
+ import { useEffect, useState } from "react";
+ import { ListTodosQuery } from "./API";

import { withAuthenticator } from "@aws-amplify/ui-react";
import "@aws-amplify/ui-react/styles.css";

import awsExports from "./aws-exports";
Amplify.configure(awsExports);

const App = () => {
+  const [todoList, setTodoList] = useState<ListTodosQuery | undefined>();
+  useEffect(() => {
+    const fetchTodos = async () => {
+      const todos = (await API.graphql({
+        query: listTodos,
+      })) as GraphQLResult<ListTodosQuery>;
+      setTodoList(todos.data);
+    };
+    fetchTodos();
+  }, []);
+ 
  return (
    <>
      <h1>Hello Amplify</h1>
+      {todoList?.listTodos?.items.map((item) => {
+        return (
+          <>
+            <div>id: {item?.id}</div>
+            <div>name: {item?.name}</div>
+            <div>desc: {item?.description}</div>
+            <div>createdAt: {item?.createdAt}</div>
+            <div>updatedAt: {item?.updatedAt}</div>
+            <br />
+          </>
+        );
+      })}
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
![](https://storage.googleapis.com/zenn-user-upload/62d4fb6ceec8-20221014.png)
GraphQLサーバーからデータが取得できて、表示されていることが確認できました。

# 小休止(リファクタリングとTanStack Queryの導入)
ここで本題とは逸れますがリファクタリングをして、TanStack Query（元React Query）を導入していきます。
リファクタリングではTodoList.tsxを作り、コンポーネント毎の役割を分けます。
TanStack Queryの導入に関してはフロントエンドでサーバーからのデータを宣言的に扱えるようになり、コードの見通しが良くなります。
## リファクタリング
```:terminal
touch src/TodoList.tsx
```
Todoのデータ取得や描画周りはTodoList.tsxに引っ越します。
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
+       <h1>Hello Amplify and @tanstack/react-query</h1>
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
これでApp.tsxからTodo関連のコードを分離できました。

## TanStack Queryの導入
コードの見通しが良くなったのでTanStack Queryの導入をしていきます。
```:terminal
npm i @tanstack/react-query
```

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
        <h1>Hello Amplify and @tanstack/react-query</h1>
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
@tanstack/react-queryを導入することによってuseStateやuseEffectがTodoListコンポーネントから消えて、`useQuery(["todoList"], fetchTodos)`に集約されました。
ここにGraphQL Code Generatorを導入して柔軟なデータ操作を実現しましょう。

# GraphQL Code Generator　init
それでは本題に戻りGraphQL Code Generatorを導入していきます。
まずは必要なパッケージをインストールします。
```:terminal
npm install graphql
npm install -D @graphql-codegen/client-preset @graphql-codegen/cli @graphql-codegen/introspection @graphql-codegen/typescript-react-query
```
次にgraphql-code-generator initをします。
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
生成されたcodegen.tsを開き、pluginを追加します。
:::message alert
筆者の環境で試したところ、`preset: "client"`の設定で生成される文と後ほど加える`plugin`で生成される文が競合してしまうバグがあったため、回避策として`preset: "client"`を削除しています。
また、`preset: "client"`で生成されるコードについて正直よく分かっておらず有識者の方がいたら教えていただけると嬉しいです。
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
次にgraphqlのコードを書いていきます。
graphql-codegenフォルダーを作り、graphqlファイルを作ります。
```:terminal
mkdir src/graphql-codegen
touch src/graphql-codegen/todo.graphql
```
todo.graphqlに取得したいデータを書いていきましょう。
:::message
筆者はAmplifyでgenerateされたsrc/graphql/内のファイルから必要なモノをコピペして必要なフィールドを残すように修正して使っています。
:::
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
GraphQL Code Generatorでコードを生成してみましょう。
```:terminal
npm run codegen
```
これでgenerated/graphql.tsが生成されています。
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

3 directories, 19 files
```
生成されたgraphql.tsを見てみましょう。
自身で定義した`todo.graphql`を元に`useListTodosQuery`関数が作られていることが確認できます。
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
さっそく使ってみたいところですが、このuseOOOを使うたびに`dataSource: { endpoint: string; fetchParams?: RequestInit }`を要求されるのは面倒です。
graphql code generatorのcustom fetcherの機能を使い、`dataSource`を毎回書かなくても済むようにしていきましょう。
```:terminal
touch src/graphql-codegen/customFetcher.ts
```
データを取得する処理を書いていきます。
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
`@aws-amplify/api`を使うことによって特に認証周りで楽が出来るようになります。
次に自分で実装したcustom fetcherを使うことをgraphql code generatorの設定に書いていきます。
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
気をつけたいのがfetcherのvalueはconfigからみた相対パスではなく、生成される`src/generated/graphql.ts`からみた相対パスであることと、#以降に自分で実装した関数の名前を書くことです。
ここまで出来たら再度graphql code generatorを動かしてコードを生成してみましょう。
```:terminal
npm run codegen
```
生成されたコードをみてみると、自分で実装した`fetchWithAmplify`が使用されていることが確認できます。
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
最後に`TodoList.tsx`内で`useListTodosQuery`を使ってデータが取得出来ているかを確認してみましょう。
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
![](https://storage.googleapis.com/zenn-user-upload/cb7916039387-20221019.png)
表示は今までと変わらないものの`TodoList.tsx`が`useListTodosQuery`だけに依存するようになり、とても見通しが良くなりました。

# まとめ
今回はリストの表示しかしていませんが、`src/graphql-codegen/todo.graphql`にmutationを書いてあげれば`useCreateTodoMutation`が簡単に手に入ります。
これだけでシンプルなサービスであれば簡単に高速で開発することが出来ると思います。
皆さんの参考になれば幸いです。

[^1]: Amplify CLIの設定はこちらを参考にしてください。https://docs.amplify.aws/cli/start/install/#option-2-follow-the-instructions