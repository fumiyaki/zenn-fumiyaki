---
title: "2024 年 12 月版 Next.js Self Hosting"
emoji: "🔍"
type: "tech"
topics:
  - "nextjs"
  - "self-hosting"
  - "vercel"
  - "render"
  - "opennext"
published: false
published_at: "2024-12-29 23:59"
publication_name: "tacoms"
---

## はじめに

### 背景

Next.js を本番運用する際、**Vercel を使うのが最も手軽かつ王道**とされています。  
しかし、コスト面・インフラポリシー・企業セキュリティ要件など様々な理由で、  
**Self Hosting** を使いたいこともあるでしょう。

実際、Next.js のすべての機能が Vercel でしか動かないわけではありません。
**2024 年のアップデート（[Next.js 14.1](https://nextjs.org/blog/next-14-1#improved-self-hosting) / [15 RC2](https://nextjs.org/blog/next-15-rc2#improvements-for-self-hosting)）** では、Self Hosting で運用する際の **[ドキュメント](https://nextjs.org/docs/app/building-your-application/deploying)** や機能サポートがかなり充実してきました。

### この記事のゴール

- **Next.js 14 / 15 の登場で変わった Self Hosting 事情を整理**する
- **Vercel／[Cloudflare Workers(with OpenNext)](https://opennext.js.org/cloudflare)／[AWS Lambda(with OpenNext)](https://opennext.js.org/aws)／Amazon ECS／Render** などの選択肢を比較検討する
- 特に **App Router で追加された ４種の Cache、Server Actions** などの最新機能に焦点を当てる

本記事は、業務で Next.js を使うフロントエンドエンジニア向けに書かれており、  
「Vercel が便利なのは知っているけど、社内事情やコストの問題で Vercel 以外のインフラを選択せざるを得ない」という方々へのヒントとなれば幸いです。

## Next.js とホスティングの概観

Next.js は以下のような**主要機能**を提供し、フロントエンドの DX を大幅に向上させるフレームワークです。

- **SSR / SSG / ISR**
- **Middleware**
- **画像最適化**
- **Server Actions**
- file-system based router, API Routes, etc. (今回の話では特に扱いません)

### Self Hosting のメリット・デメリット

| メリット                                                | デメリット                                                                                 |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| - 自由度が高い (Docker, K8s, ECS,オンプレ等何でも OK)   | - インフラやスケーリングの知識が必要                                                       |
| - コストコントロールがしやすい (VPS, 社内サーバ等)      | - 運用負荷が高い (モニタリング、デプロイフロー、障害対応を自前で行う必要)                  |
| - 特殊要件 (厳格なセキュリティ/オンプレ環境) にも対応可 | - Vercel のようなグローバル CDN や自動スケールはないため、必要なら自力で構築する必要がある |

### OpenNext, コンテナ, PaaS などの代表的選択肢

- **OpenNext(Cloudflare Workers / AWS Lambda)**

  - AWS 上で **CloudFront + Lambda@Edge** を組み合わせ、**Vercel ライクなエッジ配信**を再現する OSS
  - Cloudflare Workers でも同様にエッジロケーションでコードを実行できる。

- **コンテナ(Amazon ECS)**

  - Docker コンテナをデプロイする方式
  - 全て自前でリバースプロキシ、スケーリング、キャッシュなどを管理・設定

- **PaaS(Render)**

  - Node.js アプリとして Next.js をデプロイできる PaaS
  - Dockerized も可能。Redis や RDB 等も同じ管理画面でセットアップしやすい

「**自社でどこまで手動管理するか**」と「**クラウドが肩代わりしてくれるか**」のバランス次第で、これらの選択肢を検討することになります。

## Node Runtime と Edge Runtime

Next.js アプリケーションを実行する際、大きく分けて **Node ランタイム** と **Edge ランタイム** という 2 つの実行形態があります。
Edge ランタイムは Node ランタイムより**軽量かつ制限された** 環境（Vercel Edge Functions や Cloudflare Workers など）です。

### Node ランタイム

- **従来の Node.js 上で Next.js サーバーを起動**し、`next start` や `node server.js` 等で実行する
- `fs` や `crypto` など **標準の Node.js API** をフルに使える
- Middleware など一部機能は Edge Runtime を想定しているが、Node ランタイム上でも**Edge Runtime 互換**として動作する
- Self Hosting は多くの場合、この Node ランタイムで運用するパターンが主流

| メリット                                                              | デメリット                                                    |
| --------------------------------------------------------------------- | ------------------------------------------------------------- |
| ほぼ全ての npm パッケージが動作 (`fs`, `path`, `child_process`, etc.) | ランタイムが大きく、起動/メモリ消費が Edge Runtime より大きい |

### Edge ランタイム

- **Vercel Edge Functions** や **Cloudflare Workers** など、グローバルに配置されたエッジサーバレス環境
- **「Node.js のサブセット」** とされる制限付きのランタイム
  - `fs` や `child_process` などは使えない
- **高速なコールドスタート** でグローバルな分散デプロイが可能
- Next.js では **Middleware** がこの Edge Runtime を想定。さらに Server Components のストリーミング等で低レイテンシの恩恵を受けやすい

| メリット                                                              | デメリット                                               |
| --------------------------------------------------------------------- | -------------------------------------------------------- |
| 世界中のエッジロケーションで超高速レスポンス (コールドスタートは極小) | 一部 Node API 非対応 (`fs`, `net`, `child_process` など) |

### Node Runtime と Edge Runtime のまとめ

- **Node ランタイム** は互換性や自由度が高く、Self Hosting でも一番わかりやすい選択肢。
- **Edge ランタイム** は超高速起動と世界中への分散配置で、特に `Middleware` と組み合わせると強力だが、API 制限に注意。
- **一部の Next.js 機能 (Middleware, Server Actions など)** で Edge Runtime が推奨される場合もあるが、Node ランタイムでも“エミュレート”して動くパターンがほとんど。

## Next.js 14 / 15 で強化された Self Hosting 機能

### 1. App Router まわりのキャッシュ戦略・改善

App Router では **Request Memoization / Data Cache / Full Route Cache / Router Cache** を組み合わせたキャッシュ戦略が可能になりました。  
しかし複数インスタンス構成の場合、**デフォルトのファイルキャッシュやインメモリキャッシュ**だけでは整合性が保てず、Vercel のようにスケールアウトしにくいのが課題です。

そこで Next.js 14.1 では、**`cacheHandler`** オプションが正式安定化しました。  
たとえば以下の設定をすると、Next.js のデフォルトキャッシュを無効化し、**外部ストレージ (Redis 等) を通じてキャッシュを永続化**できます。

```js
// next.config.js
module.exports = {
  cacheHandler: require.resolve("./cache-handler.js"),
  cacheMaxMemorySize: 0, // in-memory cacheを無効化
};
```

:::message
cache-handler.js に、Redis や Memcached などを使って **同じキャッシュキー** に読み書きするロジックを実装する必要があります。
:::

この設定を組み合わせると、複数サーバーでも同じキャッシュを共有でき、Vercel に近いスケーリングを自前で構成できます。

### 2. 画像最適化の改善 (Sharp / WebAssembly)

Next.js 12.3 リリース前後あたりから、WebAssembly フォールバック (`@squoosh/lib`の独自拡張) が導入され、Sharp がインストールされていなくても画像最適化が可能となりました。[^1]
しかしパフォーマンス面から Sharp が推奨されていたのですが、**Next.js 15 以降では Sharp が自動インストールされるようになり**、
特に何もしなくても Sharp が使われる形に進化しています。

これによって、Self Hosting する場合でもセットアップが不要になり、ビルド手順がシンプルになりました。

また、WASM フォールバックも引き続き残っているので、環境によっては Sharp が使えない場合でも自動的に WASM にフォールバックしてくれます。

### 3. Middleware (Edge Runtime) の自前ホスト対応

Next.js 14.1 のリリースで Edge Runtime を想定していた Middleware が Node.js 環境でも動作するようになりました。

Self Hosting をして複数リージョンでエッジ配信したい場合は、Nginx や CloudFront + Lambda@Edge 等の追加構築が必要になります。

### 4. Runtime environment variables

Next.js 14 以降、self-hosting 時の環境変数管理に関するドキュメントも強化されました。
process.env.\* で読み込み、Docker や ECS の環境変数にマッピングする形が一般的です。
Vercel のように専用 UI は提供されないため、.env ファイル管理や CI/CD パイプラインでの注入などを自力で設計しましょう。

(この先は「Vercel / Self Hosting / Render / OpenNext の比較」や「おさらい」「まとめ」などへ繋げていただき、
本稿を完成させてください。上記の点を踏まえれば、Next.js を Self Hosting する際に必要となる知見や注意点を
ひととおり把握できる内容になるはずです。)

### その他 Self Hosting で動作するか気になる機能

#### 5. Streaming (Server Components + Suspense)

Next.js の App Router では、Server Components + Suspense を使ってレスポンスをチャンク単位で送るストリーミングが可能です。  
セルフホスティングでも、**Node.js 上で `next start`** している限り特別な設定は不要で、段階的にコンテンツを返せます。

ただし、**Nginx** などリバースプロキシ経由の場合、レスポンスをまとめて返す設定があるとストリーミングのメリットが得られません。  
必要に応じてバッファリングを無効化すると Vercel 同様に段階表示による高速なユーザー体験を提供できます。

#### 6. Server Actions

**Server Actions** は、フォーム送信や簡単な API 呼び出しをサーバーコンポーネント内で扱える機能です。  
**Vercel 固有**ではなく **Next.js 本体の機能** なので、Self Hosting (Node.js)でも特別な設定なしに動作します。  
ただし、**Edge 環境** で使う場合は、注意が必要です。
現状は OpenNext のアダプタが最新の仕様に対応しているか要確認です。[^2]

## Vercel / Self Hosting / Render / OpenNext の比較

### 共通点と相違点

表形式で「キャッシュ機能」「スケーリング」「Edge 配信」「デプロイの簡単さ」「コスト」などを比較

- Vercel
  - Next.js を最も簡単にデプロイできる
  - グローバル CDN, Edge Functions, ISR キャッシュを自動管理
  - カスタムキャッシュはやや冗長か
- Self Hosting
  - 完全自由。Docker/K8s/ECS などインフラ構成はお好み
  - Custom Cache Handler + Redis で大規模スケール対応可能
  - 運用・学習コスト高
- Render
  - “Node.js として Next.js をホスティング” …PaaS 的サービス
  - Redis や DB を同じ管理画面でセットアップ可能
  - Edge Functions のようなグローバル分散はなし
- OpenNext
  - AWS 上で CloudFront + Lambda@Edge を組み合わせて “Vercel ライクな” エッジ配信を目指す OSS
  - ただし Next.js の最新機能への追随や運用面での継続サポートに留意
- ECS
  - 画像最適化
  - Middleware
  - Server Actions
  - Runtime environment variables

## おさらい

- 「Next.js 15 から Vercel と同じ機能を全部自前で再現できる？」
  - 大部分の機能は自前で再現しやすくなったが、Edge ネットワークを含むグローバル配信などは別物
- 「Custom Next.js Cache Handler を使わないといけないの？」
  - 複数サーバー・複数インスタンス構成でキャッシュを共有する場合はほぼ必須
- 「Middleware / Edge Runtime は Node API 使えない？」
  - Edge Runtime は本質的にサーバレス的 Web APIs なので一部制限がある
- 「Server Actions などの新しい機能 は Vercel じゃないとダメ？」
  - Next.js の機能かが重要
  - 例えば Server Actions は Next.js の機能で Node.js プロセスで動作するので Self Hosting でも問題なく動く

## まとめ

Next.js 14/15 で Self Hosting が一気にやりやすくなったが、Vercel が提供するグローバルエッジ配信・自動スケールなどは依然として強み
プロジェクトの要件 (コスト, 運用負荷, グローバル展開, リクエスト量) に応じて Vercel / Self-hosting / Render / OpenNext などの選択肢を検討する
今後さらに Self-hosting 向け機能が強化される見込み。キャッシュ戦略や運用設計をしっかり組み立てることで、Vercel に頼らずとも高度な Next.js サイトを構築可能になってきている

## 参考リンク / 参考リポジトリ

- Next.js Official Docs（Self-Hosting）
- OpenNext GitHub
- Neshca (カスタムキャッシュハンドラの例)
- 公式例: Next.js Custom Cache Handler Example
- Render ドキュメント (Next.js Deploy)

## PR

現在株式会社 tacoms では全ポジションで絶賛採用募集中です！！
是非気軽にカジュアル面談からお話ししましょう 👏

https://www.tacoms-inc.com/recruit

## PR2

React Tokyo というコミュニティが出来ました 🙌
詳しくは[こちらの記事](https://zenn.dev/dai_shi/articles/9f2760086fb31a)に書かれていますが，もし React に興味あるよ，好きだよという方は是非ご参加ください 🙌
1 月はオフラインイベントもありますよ＼(^o^)／

以上！

[^1]: 残念ながらどこで導入されたか見つけることが出来ませんでした 🙏ChatGPT に聞いて 12.3 と強く言い張るのでここでは 12.3 としています
[^2]: 実際に試した方が居たら教えてください 🙏
