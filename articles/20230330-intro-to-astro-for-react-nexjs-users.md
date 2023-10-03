---
title: "React/Nexjs ユーザーに向けた Astro 入門｜Offers Tech Blog"
emoji: "🅰️"
type: "tech"
topics:
  - "jamstack"
  - "astro"
published: true
published_at: "2023-03-30 09:00"
publication_name: "overflow_offers"
---

こんにちは！
[Offers](https://offers.jp/), [Offers MGR](https://offers-mgr.com/lp) を運営している株式会社 [overflow](https://overflow.co.jp/) で Offers のフロントエンドを開発している [ふみー](https://twitter.com/fumiya_itsc) です。

最近 Astro を個人で使う機会があり、そこで調べたことの棚卸しをしようと思います。
ただ、今回は Astro の特徴的な機能である Markdown & MDX は使用しなかったため、この記事では取り扱いません。

普段 React や NextJS を使用している方向けに書きました。
この記事を押さえておけば Astro をある程度書けるレベルまでになれると思います。

# Astro の必要性

最近の Web フロントエンド界隈では React/NextJS がデファクトスタンダードになり、多くのサイトが React/NextJS を使って実装され、多くのエンジニアが使用する技術になりました。
そこまでユーザーの操作を必要としないサイトまでこれらの技術が使われており、必要以上の通信をユーザーに強いている状況になっていると思います。

今後は表示速度の速いサイトが優位に立てる状況になるのではと思います。
そこで今回はユーザーの操作を必要としない、コンテンツがメインのサービスを作る際に力を発揮する Astro を紹介いたします。

# Astro とは

Astro は Web サイトをユーザーへ早く届けるために Web サイトから JavaScript のコードを極力削った、早さに重きを置いた ALL IN ONE の Web Framework です。
Static Site Generator（SSG）が出来て、操作をメインとしないサイトやコンテンツ重視のサイトを生成することが出来ます。

直近のアップデート（v2.0, v2.1）では扱うのが大変な Markdown/MDX ファイルを Type Safe で扱えるようにしたり、自動で画像を最適化する機能を Astro Core へ取り入れたりしています。[^1]　[^2]

また、Astro Component 内で React や Svelte などの UI ライブラリを使用できる点も大きな特徴です。

# Astro Islands

Astro Islands (別名 Component Islands) は Astro が開発したアーキテクチャパターンです。
先程 Astro は JavaScript のコードを極力削った SSG だと説明しました。
それを実現するのが Astro Islands の技術です。

Astro Islands はサイト内で静的な箇所と動的（インタラクティブアイランド）な箇所を分類して、JavaScript が必要かそうでないかを区別しています。

![アストロアイランドの図](/images/20230330-intro-to-astro-for-react-nexjs-users/a.png)

Astro は何も設定しなければ全ての JavaScript を削除したサイトを生成します。
結果としてユーザーには HTML、CSS とコンテンツのみが配信されるので素早くサイトを届けることが出来ます。
もしユーザーの操作が必要な場合、アイランド毎に JavaScript を使用することを明示します。
また、その操作がどのくらい早くアクティブになって欲しいかの優先度もアイランド毎に設定することが出来ます。

# Astro 基礎

実際にプロジェクトを作成して、Astro と React/NextJS の違いを見ていきます。
`npm create Astro@latest` を Terminal で実行するとプロジェクトが作成出来ます。
Template は `Use blog template` を選択しました。

## プロジェクトのフォルダ構造

React と比べて Astro の特有な箇所をハイライトしました。
NextJS を使ったことがある方は馴染みのある構成だと思います。

```diff jsx
❯ tree .
 .
 ├── README.md
+├── Astro.config.mjs
 ├── package-lock.json
 ├── package.json
 ├── public
 │   ├── favicon.svg
 │   ├── placeholder-about.jpg
 │   ├── placeholder-hero.jpg
 │   └── placeholder-social.jpg
 ├── src
 │   ├── Components
 │   │   ├── BaseHead.Astro
 │   │   ├── Footer.Astro
 │   │   ├── FormattedDate.Astro
 │   │   ├── Header.Astro
 │   │   └── HeaderLink.Astro
 │   ├── consts.ts
+│   ├── content
+│   │   ├── blog
+│   │   │   ├── first-post.md
+│   │   │   ├── markdown-style-guide.md
+│   │   │   ├── second-post.md
+│   │   │   ├── third-post.md
+│   │   │   └── using-mdx.mdx
+│   │   └── config.ts
 │   ├── env.d.ts
+│   ├── layouts
+│   │   └── BlogPost.Astro
+│   ├── pages
+│   │   ├── about.Astro
+│   │   ├── blog
+│   │   │   ├── [...slug].Astro
+│   │   │   └── index.Astro
+│   │   ├── index.Astro
+│   │   └── rss.xml.js
 │   └── styles
 │       └── global.css
 └── tsconfig.json

9 directories, 29 files
```

## Pages と Routing

Astro は NextJS と同じようにファイルベースルーティングを採用しています。

### ページ遷移

ページ遷移には HTML スタンダードの a タグを使用します。

### ExtraFile

src/pages には実際に表示されるページとして Astro Component だけではなく HTML ファイルと Markdown/MDX ファイルも置くことが出来ます。

また、API として js/ts ファイルを置くことが出来ます。
API を使うときは以下のように api フォルダを分けた方が見通し良くなると思います。

```diff jsx
 ├── pages
 │   ├── about.Astro
+│   ├── api
+│   │   └── rss.xml.js
 │   ├── blog
 │   │   ├── [...slug].Astro
 │   │   └── index.Astro
 │   └── index.Astro
```

### 404 Not Found Page

src/pages フォルダ内に `404.Astro` または `404.md` を配置すると意図しない URL アクセス時に独自の 404 ページを出すことが出来ます。

実装しない場合は以下のように、 Astro が用意したページが表示されます。

![デフォルトの404エラーページ](/images/20230330-intro-to-astro-for-react-nexjs-users/b.png)

## Layouts

Astro の Layouts は NextJS の [Layouts](https://nextjs.org/docs/basic-features/layouts) と同じように再利用可能な構造を定義します。

しかし、Astro の Layouts には `<html>` , `<head>` , `<body>` を使用することが出来ます。
NextJS の Layouts よりも `_document.page.tsx` に役割が近いことには注意が必要です。

### slot

React でいう children を Astro では slot と呼びます。

## Integrations を追加する

Integrations を追加することによって簡単にサイトに対して機能を追加することが出来ます。
これは公式やコミュニティが作成したものを使うことも出来ますし、自身でインテグレーションを作成することも出来ます。
UI フレームワークを追加する他に、Tailwind、サイトマップの自動生成、ビルドプロセスや開発サーバーを hook するなど様々なものが公開されています。

Integration を追加することは簡単です。
以下のコマンドを使うことによって必要な依存関係、自動で必要な設定をしてくれます。

```jsx
npx Astro add react
```

## UI Component

React の Integration を導入したので、Astro Component 以外の UI Component をどのように使えるかを説明していきます。

### UI Component の配置

追加した React のコードは src/pages 以外ならどこでも保存できますが、src/pages に置いた場合は NotFound ページが表示されます。

基本的には src/Components 内のどこかに保存するのが良いと思います。

### UI Component を使用する

追加した UI Component は Astro Component 内で使用することが出来ます。

```jsx
---
import MyReactComponent from '../Components/MyReactComponent.jsx';
---
<html>
  <body>
    <h1>Use React Components directly in Astro!</h1>
    <MyReactComponent />
  </body>
</html>
```

UI Component 内に Astro Component は書くことが出来ませんが、Astro Component 内に UI Component の children に対して Astro Component を渡すことが出来ます。

文字だと説明しづらいのでコードを確認ください。

```jsx
---
import MyReactSidebar from '../Components/MyReactSidebar.jsx';
---
<MyReactSidebar>
  <p>Here is a sidebar with some text and a button.</p>
</MyReactSidebar>
```

## インタラクティブアイランドを作成する

JavaScript を有効にするためには UI Component に対して client ディレクティブに値を指定します。

- load
  ページが読み込まれたと同時に JavaScript をダウンロードして、hydrate します。

- visible
  Component が ViewPort へ入ったときに JavaScript をダウンロードして、hydrate します。

```jsx
---
import InteractiveButton from '../Components/InteractiveButton.jsx';
import InteractiveCounter from '../Components/InteractiveCounter.jsx';
---
<InteractiveButton client:load />

<InteractiveCounter client:visible />
```

他にも [多くのディレクティブ](https://docs.Astro.build/en/reference/directives-reference/#client-directives) があります。

### 複数の UI Component を使用する

あまり利用することは無いと思うのですが、Astro のユニークな機能なので紹介します。
UI Component を使用する時は Integrations を追加することを忘れないようにしましょう。

```jsx
---
import MyReactComponent from '../Components/MyReactComponent.jsx';
import MySvelteComponent from '../Components/MySvelteComponent.svelte';
import MyVueComponent from '../Components/MyVueComponent.vue';
---
<div>
  <MySvelteComponent />
  <MyReactComponent />
  <MyVueComponent />
</div>
```

## Astro Component

今まで特に説明せずに Astro Component を見てきましたが、改めて説明します。

```jsx
---
const { hello, world  } = Astro.props;
const name = "Astro";
const items = ["Dog", "Cat", "Fish"];
const visible = true;
const htmlString = '<p>Raw HTML content</p>';
function handleClick () {
    console.log("はたして俺はコンソールログに映るかな？");
}
---
<!-- HTMLにコメントが残る -->
{/* HTMLにはコメントが残らない */}
<h1>{hello}, {world}!</h1>
<div>
  <h2>Hello {name}!</h2>
</div>
<ul>
  {items.map((item) => (
    <li>{item}</li>
  ))}
</ul>
{visible && <p>表示される</p>}
{visible ? <p>表示される</p> : <p>表示されない！残念！</p>}
<Fragment set:html={htmlString} />
<button onClick={handleClick}>このボタンは押しても何も起こらないｗ！</button>
```

基本的には JSX をリスペクトして作られているので何が行われているのか想像がつくと思います。

ここからは React/NextJS では見かけない特徴を紹介します。

### コードフェンス

コードフェンス `---` で囲むと中に JavaScript を書くことが出来るようになります。
ここで Props から値を受け取ったり変数を登録して HTML 内に反映させることが出来ます。

```jsx
---
const { hello, world  } = Astro.props;
---
<h1>{hello}, {world}!</h1>
```

### <></>がトップレベルに必要ない

Astro Component はこれで動きます。
しかし、下記の場合だと<></>が必要になります。。。

```jsx
---
const items = ["Dog", "Cat", "Fish"];
---
<ul>
  {items.map((item) => (
    <>
      <li>Red {item}</li>
      <li>Blue {item}</li>
      <li>Green {item}</li>
    </>
  ))}
</ul>
```

### <Fragment set:html={htmlString} />

受け取った string の値を HTML として表示させます

```jsx
---
const htmlString = '<p>Raw HTML content</p>';
---
<Fragment set:html={htmlString} />
```

### <button onClick={handleClick}>ボタンを押しても何も起こらない</button>

Build 時に HTML の属性は全て文字列に変換されてしまうので関数を渡すことが出来ません。
この問題は下記のようにすればうまく動作します。

```jsx
---
---
<button id="button">Click Me</button>
<script>
  function handleClick () {
    console.log("button clicked!");
  }
  document.getElementById("button").addEventListener("click", handleClick);
</script>
```

これは正常に動作しますが、`<script>ほにゃらら</script>` は Global Scope での実行となるので Astro Component 内でイベント登録することはおすすめしません。
ユーザー操作を実装したい場合は UI Component を使用して、client ディレクティブを設定しましょう。

## Astro Component Props

Astro Component の props の定義のやり方は以下になります。
Astro.props を使用することで値を取り出すことが出来ます。
VS Code で開発されている方はハイライトや補完が効く様になるので [Astro VSCode Extension](https://docs.Astro.build/en/editor-setup/) をインストールしておくと DX が向上します。

```jsx
---
interface Props {
  hello: string;
  world?: string;
}
const { hello, world = "Astro" } = Astro.props;
---
<h2>{hello}, {world}!</h2>
```

```jsx
---
import { HelloWorld } from "../Components/HelloWorld";
---
<HelloWorld hello="hElLo" world="Worldn" />
```

## Routing

### SSG

`src/pages/[post].Astro` のように[]で囲むと動的にページを作成することが出来ます。

ここでは固定値で説明しますが、API を実行して、その結果を使うことも出来ます。

```jsx
---
export function getStaticPaths() {
  return [
    {params: {post: 'about me'}},
    {params: {post: 'about my a hobby'}},
    {params: {post: 'about my favorite movies'}},
  ];
}

const { post } = Astro.params;
---
<section>
  <h1>{post}</h1>
  <p>このブログは…</p>
</section>
```

上記の場合、URL は以下が出来上がります。

- /post/about-me
- /post/about-my-a-hobby
- /post/about-my-favorite-movies

### Pagination

Astro は大規模なデータコレクションのために Pagination 機能が組み込まれてます。
Pagination に必要な機能は一通り揃っている印象ですが検索のような小洒落たものは厳しい印象でした。

```jsx
---
import Layout from '../../layouts/BlogPost.Astro';
export async function getStaticPaths({ paginate }) {
  const readingbooksPages = [{
    readingbook: '炎炎ノ消防隊',
  }, {
    readingbook: 'ノラガミ',
  }, {
    readingbook: '無能なナナ',
  }, {
    readingbook: 'ワールドトリガー',
  },{
    readingbook: '推しの子',
  }, {
    readingbook: 'かげきしょうじょ!!',
  }, {
    readingbook: 'チェーンソーマン',
  }, {
    readingbook: 'ファイヤーパンチ',
  }];
  return paginate(readingbooksPages, { pageSize: 5 });
}
const { page } = Astro.props;
---

<Layout
	title="reading book"
	description="reading book"
	pubDate={new Date('August 08 2021')}
	updatedDate={new Date('August 08 2022')}
	heroImage="/placeholder-about.jpg"
>
<ul>
  {page.data.map(({ readingbook }) => <li>{readingbook}</li>)}
</ul>
{page.url.prev ? <a href={page.url.prev}>前へ</a> : <span>前へ</span>}
<span>Page {page.currentPage}</span>
{page.url.next ? <a href={page.url.next}>次へ</a> : <span>次へ</span>}
</Layout>
```

ページネーションにしたい時は `src/pages/[post].Astro` のところが `[post]` ではなく、[]の部分をページ番号にします。
`src/pages/readingBook/1` `src/pages/readingBook/2` `src/pages/readingBook/3` のような要領です。
これを実現するためには `src/pages/readingBook/[page].Astro` というファイルを用意します。
さらに、`paginate` 関数を使い、pageSize を指定すればページネーションのページを生成できます。
ついでに `page.url.prev`、 `page.url.next` と `page.url.next` を使えばいい感じに出来上がります。

![ページネーションが実装されたページ](/images/20230330-intro-to-astro-for-react-nexjs-users/c.png)

## Styles

### Astro Component にスタイリングする

Astro Component 内で<style>タグを使い、スタイリングをすると自動で Scoped Styles となり他 Component への影響を抑えられます。

```jsx
<style>
  .content { font-size: 16px; }
</style>
```

あえてグローバルにスタイリングしたい場合は以下のようにします。

```jsx
<style is:global>
  h1 { margin: 0; }
  h2 { margin: 0; }
  h3 { margin: 0; }
  h4 { margin: 0; }
  h5 { margin: 0; }
</style>
```

### React/NextJS にスタイリングする

Integration で React を入れた時点で CSS Modules が使えます。

### sass や scss を導入する

Astro はライブラリをインストールするだけでプリプロセッサを導入することが出来ます。

```terminal
npm install sass
```

後は `styles.module.scss` を作成して import したらそのまま使うことが出来ます。

## Assets (Experimental)

Assets (Experimental)は Astro@2.1.0 で追加されました。

この機能はまだ実験段階なので、破壊的変更があるかもしれません。
ここでは現在、公式ドキュメントで書かれている事実を書くに留めます。
有効化する方法は各自で調べてください。

今まで画像の最適化には Integration の追加が必要でした。
しかし、今回のアップデートで built-in image optimization が Astro Core に取り込まれたため画像を import するだけで最適化されます。

一応 src/assets/を作ることが推奨されていますが、絶対ではないみたいです。
最適化したい画像を作成したフォルダに保存しておくことによって Astro Component、md、mdx や UI Component で使用することが出来ます。

以下に例を提示します。

```jsx
---
import rocket from '../assets/images/rocket.svg'
---
<img src={rocket.src} width={rocket.width} height={rocket.height} alt="A rocketship in space." />
```

さらに今回のアップデートで、import した画像は以下のインターフェースを備えるようになったようです。

```jsx
interface ImageMetadata {
  src: string;
  width: number;
  height: number;
  format: string;
}
```

また、画像の最適化のために Astro は [squoosh](https://github.com/GoogleChromeLabs/squoosh) を使用しているようです。
カスタムで [sharp](https://github.com/lovell/sharp) に変更が可能で、Node の環境下であればこちらの方が高速に動作するようです。

# まとめ

ここまでの知識があれば Astro らしいコードを書くことが出来ます。

Astro は何度も説明しているようにユーザーの操作が少ないサイトやブログなどのコンテンツをメインとしたサイトに向いています。
しかし、ダッシュボードや SNS などのようなユーザー操作が多いサービスには向いていません。

作る前にこのサービスはどういった特性を持っているのかを考えて、最適な技術選定が出来ると良いと思います。

# 関連記事

https://zenn.dev/offers/articles/20220704-offers-hr-lab-tech-explainer

https://zenn.dev/offers/articles/20220711-develop-issues-part2

[^1]: 以前まで画像を最適化するために Integration を追加をしていました。今回のアップデートで Astro Core に画像最適化の機能が組み込まれたので、これからはデフォルトで使うことが出来ます。
[^2]: SSR（ServerSideRendering）も可能ですが Astro は SSG がコアなフレームワークだと思うので、今回は SSG に絞りました。
