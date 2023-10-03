---
title: "ChatGPTとDeepLのAPIを組み合わせて回答の精度を上げつつコストも抑える | Offers Tech Blog"
emoji: "🐱"
type: "tech"
topics:
  - "typescript"
  - "deepl"
  - "chatgpt"
published: true
published_at: "2023-03-06 09:00"
publication_name: "overflow_offers"
---

## はじめに

こんにちは！

プロダクト開発人材の副業転職プラットフォーム [Offers](https://offers.jp/) を運営する株式会社 [overflow](https://overflow.co.jp/) で Offers のフロントエンドを開発している [fumiya](https://twitter.com/fumiya_itsc) です。

この記事では、[ChatGPT API gpt-3.5-turbo](https://platform.openai.com/docs/guides/chat/chat-completions-beta)（以下、ChatGPT API）と [DeepL API](https://www.deepl.com/ja/docs-api) を組み合わせて、 ChatGPT API の回答の精度を上げる方法について説明します。
また、トークン使用数が減ることになるので ChatGPT API の実行にかかる料金を低く抑えることも出来るかもしれません。

## 背景

現在、日本語圏で ChatGPT API を利用する場合にまだ課題があると考えています。

### 課題

1. 英語の方が日本語よりも ChatGPT の精度が高い

このことは実際に両方の言語を使い比べると納得していただけるかと思います。

別観点ではそもそも ChatGPT を開発している会社の [OpenAI](https://ja.wikipedia.org/wiki/OpenAI) がアメリカの会社であることからもこの問題が出てしまうのは想像に難くないと思います。

2. ChatGPT API の料金は日本語での入力だと英語での入力と比べ割高になってしまう

まずは ChatGPT API の料金体系を説明すると、ChatGPT API は 1,000 トークン単位(イチ英単語=1 トークン)での課金になり、入力の文字と出力の文字の総量がカウントされて計算されます。
簡単な例だと「i am a pen.」は「”i” “am” “a” “pen” “.”」の 5 単語で 5 トークンになります。
しかし、日本語の場合はより複雑で「私はペンです。」という文は 7 文字ですが 10 トークン使用してしまいます。[^1]

上記の例だと同じ内容の文でかかる金額が（送信・受信も合わせて）4 倍になっています。
ChatGPT に前提などの設定をして実用的な質問をしたい場合、質問の文字数はさらに文字数が多くなってきてしまいます。
このままではトークンの消費が多くなり、多くのコストを払うことになってしまいます。[^2][^3]

### 解決案

この課題を解決するために今回は DeepL API を使用してみます。
（GUI の方の）ChatGPT を使用する際に DeepL のサービスを使っている方も多いのではないでしょうか？

DeepL API と ChatGPT API を使い、回答の精度を上げ、あわよくば費用を抑えられたらと思います。

1. 入力された日本語の質問を英語に変換する
2. 英語の質問を ChatGPT API に渡し、英語の回答を受け取る
3. 英語の回答を日本語に変換して出力する

## 今回のソリューションで得られるメリット

- 回答の精度が高くなる
- トークン数を節約できるため、質問文を長く出来る
- 1 単語 1 トークンなのでコストを事前予測出来るようになる
- コストを安く抑えられるかも

## 入力された質問を翻訳する

各サービスの API を利用するためには API Key を取得しなければいけません。
ここからは API Key を取得してからお試しください。

```typescript:deeplAPI.ts
export const EN_TO_JP = { source_lang: "EN", target_lang: "JA" } as const;
export const JP_TO_EN = { source_lang: "JA", target_lang: "EN" } as const;

export const translate = async (
  text: string,
  translationDirection: typeof JP_TO_EN | typeof EN_TO_JP
): Promise<string> => {
  const body = JSON.stringify({
    text: [text],
    ...translationDirection,
  });

  const res = await fetch("https://api-free.deepl.com/v2/translate", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: "DeepL-Auth-Key ${YOUR_DEEPL_API_Key}",
    },
    body,
  });
  const data = await res.json();
  return data.translations[0].text;
};
```

## ChatGPT API で回答を得る

```typescript:chatGPTAPI.ts
export const chatCompletion = async (
  message: string
): Promise<string | undefined> => {
  const body = JSON.stringify({
    messages: [
      {
        role: "user",
        content: message,
      },
    ],
    model: "gpt-3.5-turbo",
  });

  const res = await fetch("https://api.openai.com/v1/chat/completions", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: "Bearer ${YOUR_OPENAPI_API_Key}",
    },
    body,
  });
  const data = await res.json();
  return data?.choices[0]?.message.content;
};
```

## 一連のプログラムを組み合わせる

```typescript:main.ts
import { chatCompletion } from "./chatGPTAPI";
import { translate, EN_TO_JP, JP_TO_EN } from "./deeplAPI";

const main = async (text: string) => {
  const translatedText = await translate(text, JP_TO_EN);
  const res = await chatCompletion(translatedText);
  if (res) {
    const backTranslationText = await translate(res, EN_TO_JP);
    console.log(backTranslationText);
  }
};
```

## Deno の REPL を使って実験してみた

### 日本語の質問

```text
# 設定
あなたは経験年数15年以上のメガベンチャーの経験豊富なフロントエンジニアです。

# 質問
フロントエンジニアとして絶対に身につけておくべき技術を5つ教えて下さい。

# 前提
5つの中見出しをつけ、中見出しごとに小見出しを２つずつつけてください
```

### 英語に変換

```text
# Setup
You are an experienced front desk engineer from a mega-venture with over 15 years of experience.

# Question.
What are the 5 technologies that you absolutely need to acquire as a front engineer?

# Assumptions
Give 5 middle headings, with 2 subheadings for each middle heading
```

### 英語の質問を元に回答

```text
## Front-end Frameworks
- React
  - JSX
  - Virtual DOM
- Angular
  - TypeScript
  - Dependency Injection

## CSS Preprocessors
- SASS
  - Variables and Mixins
  - Nesting
- LESS
  - Functions and Operations
  - Lazy Loading

## Build Tools
- Webpack
  - Entry and Output
  - Loaders and Plugins
- Gulp
  - Task Runners
  - Stream Pipelines

## Package Managers
- NPM
  - Package.json
  - Scripts
- Yarn
  - Workspaces
  - Versioning

## Testing Libraries
- Jest
  - Snapshot Testing
  - Mocking
- Cypress
  - End-to-End Testing
  - Debugging Tools
```

### 回答を日本語に翻訳

```text
## フロントエンドフレームワーク
- リアクト
  - JSX
  - 仮想DOM
- アンギュラー
  - タイプスクリプト
  - 依存性注入

## CSSプリプロセッサー
- SASS
  - 変数とミキシン
  - ネスティング
- LESS
  - 関数と演算
  - 遅延ロード

## ビルドツール
- ウェブパック
  - 入力と出力
  - ローダーとプラグイン
- ガルプ
  - タスクランナー
  - ストリームパイプライン

## パッケージマネージャ
- NPM
  - パッケージ.json
  - スクリプト
- Yarn
  - ワークスペース
  - バージョン管理

## テスティングライブラリ
- ジェスト
  - スナップショットテスト
  - モッキング
- サイプレス
  - エンド・ツー・エンド・テスト
  - デバッグツール
```

## 料金に関しての考察 \*2023/03/03 時点

この節はサービスの料金改定や為替レートの関係で増減することがあります。
また、ChatGPT API（gpt-3.5-turbo）のトークンのエンコード処理が改善された場合も同様です。

ここでは日本語 1 文字 1 トークンとして（漢字などは 1 文字 2 トークンになったりします）、100 万トークンのやり取りした時にかかる金額を確認しています。
英語に変換することで何パーセント改善されるかは使い続けてログを確認しなければ傾向が分かりませんが、一旦 30%のトークン削減が出来ると仮定して考えてみます。

それぞれの料金

- DeepL
  定額 630 円
  100 万文字あたり 2,500 円

- ChatGPT
  1000 トークンあたり 0.27 円

100 万トークン分を ChatGPT のみを使った場合
合計金額：270,000 円

翻訳して 100 万トークンが 70 万トークンに圧縮出来た場合
合計金額：192,130 円（DeepL 定額 630 円 + DeepL 使用量 2,500 円 + ChatGPT 70 万トークン 189,000 円）

差額 77,870 円

## 所感

今回は回答の精度を上げるために DeepL を使用しました。
しかし、DeepL に任せきりだと細かいチューニングが翻訳時に意図通り反映されない可能性があります。
ビジネスで ChatGPT を使用する場合はそういった点に気をつけながら最適化していく必要がありそうです。

この点を解決するためには

- そもそも最初から英語で書く（DeepL は参考程度に使う）
- 割りきって日本語で書く

が良いのではないでしょうか。

## まとめ

巷で話題の ChatGPT で使用されている API が公開されたということで、少し触ってみました。
こういったチャット型の AI に関しては最初は間違えることも多い（しかもあたかも本当のように伝えてくる…）ので自分で検索したほうが良いと触らずにいました。
しかし結局今の Google もたくさんの情報の中から本当に欲しい情報を探し出す必要があり、どちらも使用者のネットリテラシーに依存するんだなと使っていて感じました。

使ってみた感想としては楽しかったです。
これからも使っていって、ゆくゆくは実態のない猫型ロボットみたいになれば嬉しいなと思いました。

## おまけ

とある業務のイチ風景

![会社のSlackでChatGPTを使ってわいわいしている画像](/images/20230306-chatgpt_and_deepl/noisy-on-company-slack.png)


## 参考

[Chat completions 公式 Doc](https://platform.openai.com/docs/guides/chat)
[DeepL API：文書ファイル翻訳時の文字数カウント](https://support.deepl.com/hc/ja/articles/360020715799-DeepL-API-%E6%96%87%E6%9B%B8%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E7%BF%BB%E8%A8%B3%E6%99%82%E3%81%AE%E6%96%87%E5%AD%97%E6%95%B0%E3%82%AB%E3%82%A6%E3%83%B3%E3%83%88)
[ChatGPT の API を使う](https://okumuralab.org/~okumura/python/chatgpt_api.html)
[How to count tokens with tiktoken](https://github.com/openai/openai-cookbook/blob/main/examples/How_to_count_tokens_with_tiktoken.ipynb)

[^1]: [こちらのサイト](https://platform.openai.com/tokenizer) で文をトークンに変換して確認できます。また、API レスポンスからもかかったトークン数を確認することが出来ます。
[^2]:
    良さげな回答を得るためにテンプレートを使用すると、日本語の場合 417tokens、英語の場合 158tokens という結果が出ました。
    これは英語に比べ、約 6 倍のコストを支払うことを意味します。

[^3]: [gpt-3.5-turbo は日本語の消費トークン数に改善が入った可能性も指摘されています](https://twitter.com/santa128bit/status/1631072216187019265?s=20) が、公式からの言及は確認できませんでした。
