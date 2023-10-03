---
title: "Emotionを使ってきた私がCSS Modulesの環境で感じたそれぞれの良いところ | Offers Tech Blog"
emoji: "💎"
type: "tech"
topics:
  - "css"
  - "react"
  - "scss"
  - "cssmodules"
  - "emotion"
published: true
published_at: "2023-01-19 09:00"
publication_name: "overflow_offers"
---

# はじめに

こんにちは！

プロダクト開発人材の副業転職プラットフォーム [Offers](https://offers.jp/) を運営する株式会社 [overflow](https://overflow.co.jp/) で Offers のフロントエンドを開発している [fumiya](https://twitter.com/fumiya_itsc) です。

2023 年は記事をたくさん書こうと思っています。

この記事はこれまで Emotion を使ってきた私が、現在のプロジェクトで CSS Modules を使うようになって感じているそれぞれの使っていて良いなと思った点をまとめた記事になります。

# 比較する開発環境について

前の会社で経験した Emotion の環境と現在の CSS Modules を使った環境について説明します。
掲載しているコードは雰囲気を感じてもらえる程度のものを作りました。

## 以前の環境（Emotion）

これまでやってきたプロジェクトでは Emotion を使用していました。
また、[String Styles](https://emotion.sh/docs/@emotion/css#string-styles) の形式を採用していました。

その他は Material-UI を使っていましたが今回のコードでは省略しています。

```tsx:Emotion String Stylesの例
import React from 'react';
import { css } from "@emotion/react";

type Props = {
  children: React.ReactNode;
};

const buttonText = css`
  color: red;
  background-color: blue;
  font-size: 12px;
`

const Button = ({ children }: Props) => (
  <button css={css`padding: 8px`}>
    <span css={buttonText}>{children}</span>
  </button>
);

export default Button;
```

## 現在の環境（CSS Modules）

現在の環境では CSS Modules を使用しています。
[こちら](https://zenn.dev/offers/articles/20220804-css_design_with_css_modules) で紹介されているように型を生成しています。
SCSS 記法を採用して、palette や space などを定義してデザインシステムを作っています。

```scss:palettes.scss
$palettes: (
  primary-black: #000,
  primary-white: #fff,
  // …
)
```

```tsx:Button.tsx
import styles from './style.module.scss';
import { HtmlProps } from '@/types/html';
import { LegacyReactFC } from '@/types/react';

type BaseProps = {
  color?: string;
};

type MainProps = HtmlProps<'button'> & BaseProps;

export const Button: LegacyReactFC<MainProps> = (props) => {
  const {
    children,
    className,
    color,
    ...rest
  } = props;

  const classNames = [
    className,
    styles.button,
    styles[`palette-${color}`],
  ];

  return (
    <button {...rest} className={classNames.join(' ')}>
      <span className={styles.contents}>{children}</span>
    </button>
  );
};

Button.defaultProps = {
  color: 'primary-100',
};
```

```scss:style.module.scss
.button {
  display: inline-flex;
  justify-content: center;
  align-items: center;
  @each $key, $value in $palettes {
    #{'&.palette-' + $key } {
      background-color: $value;
      border: 1px solid $value;
    }
  }
}
.contents {
  display: flex;
  align-items: center;
}
```

Emotion と CSS Modules をどのように使っているかなんとなく掴んでもらえたと思います。
次は私が思うそれぞれの良いところを紹介していきます。

# Emotion の良いところ

## JavaScript であること

スタイルを props で渡ってきた変数を用いて変化させることが出来ます。

```tsx:Paragraph.tsx
const Paragraph = ({ color, text }: Props) => (
  <p css={css`color: ${color};`}>
    {text}
  </p>
);
```

計算した結果を使いスタイルを動的に変えることが出来る

```tsx:Numerical.tsx
const Numerical = ({ num }: Props) => {
  const plusMinusColor = num > 0? "blue" : "red"
  return (
    <span css={css`color: ${plusMinusColor};`}>
      {num}
    </span>
  )
};
```

細かく変数にして、後でガッチャンコ出来る

```tsx:Button.tsx
const Button = ({ isActive, text }: Props) => {

  const buttonColor = css`
    background-color: red;
  `;

  const buttonTextStyle = css`
    font-size: 16px;
    font-weight: 700;
  `;

  const buttonDisable = css`
    opacity: 0.7;
  `;

  const buttonStyle = css`
    ${buttonColor}
    ${buttonTextStyle}
    ${!isActive && buttonDisable}
  `;

  return <button css={buttonStyle}>{ text }</button>
}
```

## 1 コンポーネント 1 ファイルで作れる

コンポーネントは 1 つの役割を持ち、1 ファイルに構造・スタイル・処理が定義されるべきだと考えています。
もともと Vue をやっていたためこのような考えを持っているのだと思います。
ファイルが増えすぎず、ファイル移動の手間も無くなるのが良いと思います。

## 使っているのか使っていないのかが VSCode の恩恵でわかりやすい

スタイルを変数で管理すると使用していない変数を VSCode が教えてくれます。
![使用していないスタイル変数を教えてくる](/images/20230119-advantages-of-emotion-and-css-modules/01.png)
結局いらないとなった時に、視覚的にここ使ってないよって教えてくれるのは助かります。

## 素の CSS が書ける

これは慣れの問題ではあるのですが、オブジェクト型での書き方が慣れないため素の CSS が書けるのは自分にはメリットでした。
あと、Web で出てくる CSS もオブジェクト型ではないのでコピペするときに貼り付けで済むのも楽です。

# CSS Modules の良いところ

## ひとつのコンポーネントについてファイル分割して構成することに抵抗がなくなった

これまで 1 コンポーネント 1 ファイルスタイルとしていましたが、ファイルを分けるなら処理も分けようという気になります。（気持ちの問題ですね。）
Button.tsx の他に style.module.scss や useButton.hooks.ts など書き分けることに抵抗が無くなり、テストもスッキリ書けるようになりました。

また、ファイルの行き来に関しては大きいモニターを買うことによって左に構造、右にスタイルでファイルの行き来をしないで済むようにして、デメリット？を無くしました。
![1ファイル1コンポーネントを克服した図](/images/20230119-advantages-of-emotion-and-css-modules/02.png)

## バンドルサイズが小さい

実際に測って比較したわけではありませんが、単純に考えて Web ページは Emotion のライブラリサイズ分多くダウンロードする必要があるため、シンプルな CSS を吐き出してくれる CSS Modules の方がバンドルサイズは小さくなります。

## 素の CSS が書ける

Emotion のメリットと同じです。

## 16 進数形式で書ける

これは CSS Modules の機能ではありませんのでおまけ程度に見ていただけたらと思います。

SCSS を使うことによって `background-color: rgba(#000, 0.7)` のように書くことが出来ます。
これはデザインシステムのカラーを 16 進数形式で管理しているときにとても便利に思いました。
`opacity` だけで解決できないときに `rgba(#000, 0.7)` を使えることを知り、めっちゃ助かったので無理やりここで紹介しました。

# まとめ

Emotion ユーザーの自分が CSS Modules を触る事になり、それぞれ触っていて良いと思ったところを振り返ってみました。

これまで Emotion を使用していたので違和感を感じながらコーディングすることになるかと思っていたのですが、そこまで気になりませんでした。

今はユーザーに届ける際のバンドルサイズが小さくなり開発速度も変わらないのであれば、今後自分でなにか作るときにも CSS Modules を採用しようと思っています。

最後にはなりますが、本記事を最後まで読んで頂き、ありがとうございました。「**こんな記事を書いてほしい！**」などありましたらコメントいただけると幸いです。

# 関連記事

https://zenn.dev/offers/articles/20220804-css_design_with_css_modules
https://zenn.dev/offers/articles/20220818-css-custom-highlight-api
https://zenn.dev/offers/articles/20220804-css_design_with_css_modules
https://zenn.dev/offers/articles/20220627-css-anchored-positioning
