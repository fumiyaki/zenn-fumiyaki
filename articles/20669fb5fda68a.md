---
title: "Amplifyを用いてパスワードレス認証(SMS)を超簡単に実現する"
emoji: "🥺"
type: "tech"
topics:
  - "react"
  - "amplify"
  - "cognito"
  - "sms認証"
  - "パスワードレス認証"
published: true
published_at: "2022-05-10 22:57"
---

:::details 宣誓(宣伝)
~~次の技術書店でAmplifyについての本を出します。(執筆中ぅ)
Amplifyの入門から実践で使えるテクニックをまとめているので良ければ購入していただけると嬉しいです。
Twitterをフォローして執筆状況の情報を受け取っていただけたらと思います。~~

無理でした^^
またのチャレンジに期待しています！
:::

# モチベーション
皆さんはログインが必要なサービスをいくつ利用されていますか？
自分はだいたい50個くらいのサービスを利用していて、皆さんも10〜100くらいのサービスを登録・利用されているのではないでしょうか？
ここで質問です。
皆さんはサービスごとにパスワードを変えているでしょうか？

昨今の情報漏洩のニュース[^1]を見ているととても怖い気持ちになり、自分は数年前にiCloudのキーチェーン[^2]やChromeのパスワード管理[^3]でサービスごとに別のランダムのパスワードを設定しました。
しかし、上記の両方を使っているとこっちでは保存して出るけどこっちでは出ないみたいな問題や、サービスの登録時にパスワードを保存しますか？が出なかったりで面倒くさい思いをしていました。
~~1Passwordは毎月課金されるのが嫌で使ってません。これ使ったらええやんって話は受け付けません。はい。~~
ある時YahooでログインをしようとしたときにSMSだけでログイン出来ることを発見しました。[^4]
見つけたときに超画期的じゃ〜んって思いまして、今回の記事はそんなパスワードレス認証を実現するための方法を布教するために書きました。
この世からパスワードを殲滅するための一助となれば幸いです。

## 対象読者
Amazon Cognitoでパスワードレス認証を実装したいすべての人

# プロジェクトの作成
今回はReactを使ってフロントを作っていきます。
しかしReactは話のメインではないのでReactを知らなくてもなんとなく分かるように進めていくのですが、実際に試す場合は調べながら頑張ってください。
では、始めます。

## Reactアプリの作成
さっそくReactプロジェクトの作成をして、VSCodeで開いていきましょう。
```terminal:terminal
$ npx create-react-app passwordless-app --template typescript
$ code passwordless-app
```
これでReactアプリの作成は終わりです！簡単ですね。

## Amplifyプロジェクトの作成
次にReactアプリにAmplifyを導入していきます。(AWSは登録していて、ログイン済みを想定しています。)
まずはamplifyコマンドをインストールして、設定をしましょう。
```terminal:terminal
$ npm install -g @aws-amplify/cli
$ amplify configure
Follow these steps to set up access to your AWS account:

Sign in to your AWS administrator account:
https://console.aws.amazon.com/
Press Enter to continue

Specify the AWS Region
? region:  ap-northeast-1
Specify the username of the new IAM user:
? user name:  passwordless-app-user
Complete the user creation using the AWS console
https://console.aws.amazon.com/iam/home?region=ap-northeast-1#/users$new?step=final&accessKey&userNames=passwordless-app-user&permissionType=policies&policies=arn:aws:iam::aws:policy%2FAdministratorAccess-Amplify
Press Enter to continue
```
リージョンをap-northeast-1に設定して、Amplifyの操作をするIAMユーザー名をpasswordless-app-userと設定しました。
そうするとブラウザが開くのでIAMユーザーを作成します。
既に必要なパラメーターは設定されているので次のステップ押していってユーザーの作成まですすめます。
制作されたユーザーのAccessキーとシークレットアクセスキーはメモしておいてください。
ターミナルに戻ってEnterを押して、Accessキーとシークレットアクセスキーを入力します。
Profile Nameはdefaultで大丈夫です。
```terminal:terminal
Enter the access key of the newly created user:
? accessKeyId:  ********************
? secretAccessKey:  ****************************************
This would update/create the AWS Profile in your local machine
? Profile Name:  default

Successfully set up the new user.
```
これでamplifyコマンドの設定が完了しました。
次はReactアプリにAmplifyを導入していきます。
```terminal:terminal
$ amplify init

? Enter a name for the project passwordlessapp
The following configuration will be applied:

Project information
| Name: passwordlessapp
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
? Select the authentication method you want to use: AWS profile

For more information on AWS Profiles, see:
https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html

? Please choose the profile you want to use default
Adding backend environment dev to AWS Amplify app: dx4eg3hyhy7i0
⠧ Initializing project in the cloud...

// 多くのリソースを作るログが出るが省略

Initialized your environment successfully.

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
ここまででAmplifyのリソースを作っていく雛形が出来ました。
Amplifyが現在のプロジェクトを察してよしなに設定してくれるので楽ちんですね。

しかし、まだReactとAmplifyの連携が出来ていないので、ちょちょいとやってしまいましょう。
まずはAmplifyのライブラリのインストールです。
```terminal:terminal
npm install aws-amplify
```
次に、Reactの初期化時にAmplifyを使えるように設定をしていきましょう。
ついでにいらないコードも消してスッキリさせましょう。
諸々追加したコードは下記になります。
```TypeScript:src/index.tsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";

import { Amplify } from "aws-amplify";
import awsExports from "./aws-exports";

Amplify.configure(awsExports);

const root = ReactDOM.createRoot(
  document.getElementById("root") as HTMLElement
);
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

```TypeScript:src/App.tsx
function App() {
  return <div>Hello TypeScript &amp; React &amp; Amplify!</div>;
}

export default App;
```
これでReactアプリとAmplifyの連携が出来ました。

ここまでの内容をCommitしておきます。
Commitする際にはamplify/team-provider-info.jsonのファイルはgit管理から外しておきましょう。[^5]

### ここまでの編集履歴
https://github.com/fumiyaki/passwordless-app/commit/2141de126d5e3f05d4842169fe4cfa7830ff24b0

# パスワードレス認証の作成
ここまででReactとAmplifyの連携が取れるようになりましたが、Amplifyはまだ何もしてくれていません。
Amplifyにはいろいろな機能[^6]がありますが、必要なものを1つ1つaddしていく事によってそれらの機能が使えるようになります。
では、認証機能を使えるようにしていきましょう。

## パスワードレス認証の準備(重要1)
ここから認証の追加をしていきます。
メールアドレス/パスワード認証なら最初の質問でDefault configurationを選択すると一瞬でサービスに認証を追加することが出来ますが、今回はパスワードレス認証を実現したいのでManual configurationを選択します。
では早速流れを確認しましょう。
```terminal:terminal
$ amplify add auth

Using service: Cognito, provided by: awscloudformation
 
 The current configured provider is Amazon Cognito. 
 
 Do you want to use the default authentication and security configuration? Manual configuration
 Select the authentication/authorization services that you want to use: User Sign-Up & Sign-In only (Best used with a cloud API only) // 今回はSignIn, SginUpが出来れば良いのでこちらを選択したが、Amplifyの機能をフルで使いたいならUser Sign-Up, Sign-In, connected with AWS IAM controlsを選択する。
 Provide a friendly name for your resource that will be used to label this category in the project: passwordlessappec74566a
ec74566a
 Provide a name for your user pool: passwordlessappec74566a_userpool_ec74566a
 Warning: you will not be able to edit these selections. 
 How do you want users to be able to sign in? Phone Number
 Do you want to add User Pool Groups? No
 Do you want to add an admin queries API? No
 Multifactor authentication (MFA) user login options: ON (Required for all logins, can not be enabled later)
 For user login, select the MFA types: SMS Text Message
 Specify an SMS authentication message: Your authentication code is {####}
 Email based user registration/forgot password: Disabled (Uses SMS/TOTP as an alternative)
 Please specify an SMS verification message: Your verification code is {####}
 Do you want to override the default password policy for this User Pool? No
 Warning: you will not be able to edit these selections. 
 What attributes are required for signing up? 
 ◯ Zone Info (This attribute is not supported by Facebook, Google, Login With Amazon, Signinwithapple.)
 ◯ Address (This attribute is not supported by Facebook, Google, Login With Amazon, Signinwithapple.)
 ◯ Birthdate (This attribute is not supported by Login With Amazon, Signinwithapple.)
❯◯ Email // Emailが最初から選択されているが今回はEmailは使わないのでチェックを外す
 ◯ Family Name (This attribute is not supported by Login With Amazon.)
 ◯ Middle Name (This attribute is not supported by Google, Login With Amazon, Signinwithapple.)
 ◯ Gender (This attribute is not supported by Login With Amazon, Signinwithapple.)
(Move up and down to reveal more choices)
Specify the app's refresh token expiration period (in days): 30
 Do you want to specify the user attributes this app can read and write? No
 Do you want to enable any of the following capabilities? (Press <space> to select, <a> to toggle all, <i> to invert select
ion) // 全て必要がないのでそのままエンターを押して次へ進む
❯◯ Add Google reCaptcha Challenge
 ◯ Email Verification Link with Redirect
 ◯ Add User to Group
 ◯ Email Domain Filtering (denylist)
 ◯ Email Domain Filtering (allowlist)
 ◯ Custom Auth Challenge Flow (basic scaffolding - not for production)
 ◯ Override ID Token Claims
 Do you want to use an OAuth flow? No
? Do you want to configure Lambda Triggers for Cognito? No
✅ Successfully added auth resource passwordlessappec74566aec74566a locally

✅ Some next steps:
"amplify push" will build all your local backend resources and provision it in the cloud
"amplify publish" will build all your local backend and frontend resources (if you have hosting category added) and provision it in the cloud

⚠️ You have enabled SMS based auth workflow. Verify your SNS account mode in the SNS console: https://console.aws.amazon.com/sns/v3/home#/mobile/text-messaging
If your account is in "Sandbox" mode, you can only send SMS messages to verified recipient phone numbers.
```
これでローカル上では認証の追加が完了しました。後はAWSにこの情報を送れば認証の構築が完了します。
それではAWS上に先程作ったリソースの情報を送りましょう。

```terminal:terminal
$amplify push
✔ Successfully pulled backend environment dev from the cloud.

    Current Environment: dev
    
┌──────────┬─────────────────────────────────┬───────────┬───────────────────┐
│ Category │ Resource name                   │ Operation │ Provider plugin   │
├──────────┼─────────────────────────────────┼───────────┼───────────────────┤
│ Auth     │ passwordlessappec74566aec74566a │ Create    │ awscloudformation │
└──────────┴─────────────────────────────────┴───────────┴───────────────────┘
? Are you sure you want to continue? Yes

// 多くのリソースを作るログが出るが省略

✔ All resources are updated in the cloud
```
これでAmplify Authの環境が完成しました。
今の設定で電話番号とパスワードでのSignUpとSignInとSMSによる2段階認証(登録時・ログイン時)が設定されています。

これは余談で、すでに気づいている方もいるかとは思いますがAmplify AuthはAWSのCognitoが裏で動いています。
Amplify Authのことをより深く知りたくなったときはAmplify Authで調べても良いですし、Cognitoの情報を探って両面から調べることが可能です。

## パスワードレス認証の実装(重要2)
ここまででReactのセットアップ、Amplifyとの連携、Amplify Authの追加が完了しました。
既にCognitoでは電話番号とパスワードでのSignUpとSignInとSMSによる2段階認証(登録時・ログイン時)が設定されていますので、後は実装でパスワードレスにしていきます。
:::message alert
先にネタバラシをしておくと、SignUp時にユーザーにパスワードの入力をさせずに裏でパスワードを固定の値にしてユーザーの登録をし、SMSでの本人確認を経て登録を完了します。
その後、SignIn時に電話番号をユーザーに入力してもらい、パスワードは固定値を飛ばすようにしておけば最初のログインが成功してSMSの2段階認証が走ります。これはユーザーの目にはパスワードレス認証(SMS)に見えるはずです。
:::

それではネタバラシも済んだところで実装を確認していきましょう。
実装での気をつけたい点は電話番号に国コードが必要というところとパスワードを固定値にしているところです。
後はシンプルな実装なのでサッと読めるかと思います。
```TypeScript:src/App.tsx
import { useState } from "react";
import { Auth } from "aws-amplify";
import { CognitoUser } from "@aws-amplify/auth";

const JAPAN_PHONE_COUNTRY_CODE = "+81";
const PASSWORD = "e2D3ZfMT";
function App() {
  const [signUpPhoneNumber, setSignUpPhoneNumber] = useState("");
  const [confirmSignUpCode, setConfirmSignUpCode] = useState("");

  const [signInPhoneNumber, setSignInPhoneNumber] = useState("");
  const [cognitoUser, setCognitoUser] = useState<CognitoUser | null>(null);
  const [confirmSignInCode, setConfirmSignInCode] = useState("");

  const signUp = async () => {
    try {
      const { user } = await Auth.signUp(
        JAPAN_PHONE_COUNTRY_CODE + signUpPhoneNumber,
        PASSWORD
      );
      console.log("仮登録をしました。user:", { user });
    } catch (error) {
      console.log("error signing up:", { error });
    }
  };

  const confirmSignUp = async () => {
    try {
      await Auth.confirmSignUp(
        JAPAN_PHONE_COUNTRY_CODE + signUpPhoneNumber,
        confirmSignUpCode
      );
      alert("登録完了しました。");
    } catch (error) {
      console.log("error confirming sign up", error);
    }
  };

  async function signIn() {
    try {
      const user = await Auth.signIn(
        JAPAN_PHONE_COUNTRY_CODE + signInPhoneNumber,
        PASSWORD
      );
      setCognitoUser(user);
    } catch (error) {
      console.log("error sign in", error);
    }
  }

  const signInChallenge = async () => {
    if (!cognitoUser) {
      return;
    }
    try {
      const loggedUser = await Auth.confirmSignIn(
        cognitoUser,
        confirmSignInCode,
        "SMS_MFA"
      );
      console.log(loggedUser);
      alert("ログインしました。");
    } catch (error) {
      console.log("error confirming sign in", error);
    }
  };

  return (
    <div>
      <div className="signUpSeriesContainer">
        <div className="signUpContainer">
          <input
            type="text"
            placeholder="signUpPhoneNumber"
            value={signUpPhoneNumber}
            onChange={(e) => setSignUpPhoneNumber(e.target.value)}
          />
          <button onClick={() => signUp()}>仮登録</button>
        </div>
        <div className="confirmSignUpContainer">
          <input
            type="text"
            placeholder="confirmSignUpCode"
            value={confirmSignUpCode}
            onChange={(e) => setConfirmSignUpCode(e.target.value)}
          />
          <button onClick={() => confirmSignUp()}>登録</button>
        </div>
      </div>
      <div className="signInSeriesContainer">
        <div className="signInContainer">
          <input
            type="text"
            placeholder="signInPhoneNumber"
            value={signInPhoneNumber}
            onChange={(e) => setSignInPhoneNumber(e.target.value)}
          />
          <button onClick={() => signIn()}>SMSを送る</button>
        </div>
        <div className="confirmSignUpContainer">
          <input
            type="text"
            placeholder="confirmSignInCode"
            value={confirmSignInCode}
            onChange={(e) => setConfirmSignInCode(e.target.value)}
          />
          <button onClick={() => signInChallenge()}>確認する</button>
        </div>
      </div>
      <div
        className="singOutContainer"
        onClick={() => {
          Auth.signOut();
        }}
      >
        ログアウト
      </div>
    </div>
  );
}

export default App;
```
`npm start`して実際に試してみてください。
1個めのinputで仮登録が出来、2個めのinputで登録が完了します。
3個めのinputでSMSを送信し、4個めのinputでログインが完了します。
ここまででAmplifyを用いてパスワードレス認証(SMS)を超簡単に実現出来ました。

### ここまでの編集履歴2
https://github.com/fumiyaki/passwordless-app/commit/aa534f07b0c2cd62f533fc91804afc938b4918a3

# このやり方はセキュアなのか
ここからはこの実装が本番環境で使用しても良いものかどうかを考えてみます。
セキュリティの専門家では無いため有識者の意見を伺いたいところですが、自分なりにどうしてこの実装でも安全だと思うのかについて箇所書きレベルで記しておきます。

## 安全だと思う理由
- ログイン時に毎回SMS認証が必要で、SMSに書かれたコードを第三者が入手できない
この1点に尽きる気がします

## 不安に思う点
- パスワードが固定
	- パスワードを把握されランダムな電話番号で試された場合SMS送信料金が心配・・・
	- Try回数の上限を決めるなどの対策は可能そう
	- 実在する電話番号が見つかり、先程のコードで言うconfirmSignInCodeを6桁分総なめされるといつかうまくいく？
	- それもTry回数の上限を決めたり、いくつかの失敗後にCodeを再発行をするなどして対策は可能そう
- 電話番号をMNPせずに無くした場合、数年後に別のユーザーの電話番号となりログインをされる可能性
	- 無いとは言い切れない問題
	- 規約で数年間ログインがない場合データの削除を行う
	- FIDOを利用してパスワードレスで2段階認証(今回のものを使うなら実質3段階認証笑)にする
	- どれもあまり良い解決策では無い気がする・・・
	- パスワード漏洩によるアクセス被害とこの問題のどっちを重要視するかの天秤か？
	- それならパスワードとSMS認証の組み合わせが最強か・・・？
	- セキュリティ上危険なのはITリテラシーが高くない人と想定した時に2段階認証の設定は望みは薄い
	- その時にパスワードが同一・単純になるくらいならSMSを利用することの方がセキュリティ事故等の被害者の数が少なくなる気もする。
	- とはいえどうしても防ぎきれない問題なことには変わりはないか・・・？
- SMS送信自体のセキュリティに問題があり、盗聴される可能性
	- メールアドレス/パスワードでも同じリスクはある。
	- 結局誰をどのようにして守るかの観点が大事かと思う
	- つまり、ITリテラシーが高くない人にとってはSMSの方が安全そうで、リテラシーが高くもっとセキュアにしたい人がFIDOなどの利用やパスワード付きの2段階認証をすれば良いのかなと

最後かなりごった煮になってしまいましたが思いつく限りを出してみました。
他にも想定されることや安全性についての言及はコメント欄にて教えていただけるととても嬉しいです。

# まとめ
友人とアプリ開発をする際にパスワードレス認証にして欲しいとの強い要望があり、その際に思いついた手段を書いてみました。
もしここまでの説明・実装でセキュリティホールがあればコメントで教えていただけると嬉しいです。
もし、安全そうだと思う場合もその旨を書いていただけると安心して布教が出来るので是非コメントいただきたいです。
また、Amazon Cognito公式でのパスワードレス認証の追加をしていただけると嬉しいのでもしこの記事が届いたら強く実装を望みます。もっと多くの選択肢が増えると嬉しと思います。

ここまで読んでいただきありがとうございました。
以上。

[^1]:個人情報漏洩事件・被害事例一覧がこちらで確認ができます。
https://cybersecurity-jp.com/leakage-of-personal-information
[^2]:https://support.apple.com/ja-jp/HT204085
[^3]:https://support.google.com/chrome/answer/95606?hl=ja&co=GENIE.Platform%3DDesktop
[^4]:https://about.yahoo.co.jp/info/blog/20200914/passwordless.html
[^5]:https://docs.amplify.aws/
のFeaturesを確認してみてください。
認証、ストレージ、API、DataStore、マップ、Analytics、通知、XR、IoT、チャットボット、AI/MLなどの機能が（2022/05/10時点）あります。色々あってワクワクしますね。
[^6]:https://www.bioerrorlog.work/entry/public-amplify-project
Amplifyプロジェクトのgitリポジトリを公開するときの注意点