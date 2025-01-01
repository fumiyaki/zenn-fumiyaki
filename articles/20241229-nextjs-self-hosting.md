---
title: "2025年 Next.js Self Hosting"
emoji: "🌅"
type: "tech"
topics:
  - "nextjs"
  - "ecs"
  - "vercel"
  - "render"
  - "opennext"
published: true
publication_name: "tacoms"
---

この zenn の記事 は、tacoms Advent Calendar 2024 の 6 日目です!
他メンバーの Advent Calendar はこちらからご覧ください！👇

https://qiita.com/advent-calendar/2024/tacoms

それでは tacoms Advent Calendar 2024 の 6 日目の記事スタートです 🫣

## はじめに

こんにちは！fumiyaki です！
今回は所属しているコミュニティの [React Tokyo](https://discord.gg/5B9jYpABUy) で『今 React，Next.js をデプロイするならどこがおすすめですか？』というスレッドがあり、そこで盛り上がったため、自分なりに深堀りしてまとめてみました。
24 年中に公開するつもりでしたが年が明けたので最新っぽいタイトルをつけることが出来ました 🫠

### 背景

Next.js を本番運用する際、**Vercel を使うのが最も手軽**とされています。
しかし、コスト面・インフラポリシー・企業セキュリティ要件など様々な理由で、
**Self Hosting** をする場合もあります。

**2024 年のアップデート（[Next.js 14.1](https://nextjs.org/blog/next-14-1#improved-self-hosting) / [15 RC2](https://nextjs.org/blog/next-15-rc2#improvements-for-self-hosting)）** では、Self Hosting で運用する際の **[ドキュメント](https://nextjs.org/docs/app/building-your-application/deploying)** や機能サポートがかなり充実してきました。
そこで、Next.js を Self Hosting する際の選択肢を整理してみました。

### この記事のゴール

- **Next.js 14 / 15 の登場で変わった Self Hosting 事情を整理**する
- **Vercel／[Cloudflare Workers with OpenNext](https://opennext.js.org/cloudflare)／[AWS Lambda with OpenNext](https://opennext.js.org/aws)／Amazon ECS／Render** を比較検討する
- 特に **App Router で追加された ４種の Caching、Server Actions** などの最新機能に焦点を当てる

本記事は、業務で Next.js を使うフロントエンドエンジニア向けに書かれており、
「Vercel が便利なのは分かっているけど、社内の事情やコストの問題で Vercel 以外のインフラを選択せざるを得ない」という方々へのヒントとなれば幸いです。

## Next.js と Hosting

Next.js は以下のような**主要機能**を提供し、フロントエンドの DX を大幅に向上させるフレームワークです。

- **SSR / SSG / ISR**
- **Middleware**
- **画像最適化**
- **Server Actions**
- file-system based router, API Routes, etc. (今回の話では特に扱いません)

### Vercel で Hosting をするメリット

- Middleware、 Server Actions や `export const runtime = "edge"` と書いた箇所は Edge ランタイム上で動作する
- スケーリング、CI/CD、モニタリング、障害対応がオールインワンで提供される
- 複数インスタンスで自動スケーリングしながらキャッシュを共有

### Self Hosting をするメリット

- PaaS, ECS, K8s, オンプレ等何でも OK
- 好きな機能やサービスをいくらでも組み合わせられる
- 特殊要件 (厳格なセキュリティ/オンプレ環境) にも対応可能
- コストコントロールがしやすいことがある

### Self Hosting をするツラミ

- 複数インスタンスでの Cache ハンドリング、CI/CD、モニタリング、障害対応など全て自前で行う必要がある
- Edge での配信は all or nothing で、一部(例えば Middleware だけ)を Edge に配置することは出来ない

### OpenNext, コンテナ, PaaS の代表的な選択肢

- **OpenNext(AWS Lambda / Cloudflare Workers)**

  - Lambda@Edge では Node ランタイム、Cloudflare Workers では Edge ランタイム上で Next.js を実行する
  - AWS 上で **CloudFront + Lambda@Edge** を組み合わせ、**Vercel ライクなエッジ配信**を再現する OSS
  - Cloudflare Workers でも同様に Edge ロケーションでコードを実行できる。

- **コンテナ(Amazon ECS)**

  - Docker コンテナの Node ランタイム上で Next.js を実行する
  - 全て自前でリバースプロキシ、スケーリング、キャッシュなども管理・設定可能

- **PaaS(Render)**

  - PaaS の Node ランタイム上で Next.js を実行する
  - 負荷増時のスケーリングは手動・自動を選択可能
  - RDB や Redis 等も管理画面でセットアップしやすい

「**自前でどこまで管理するか**」と「**サービスにどこまで肩代わりしてもらうか**」のバランスを検討することになります。

## Node ランタイムと Edge ランタイム

少し横道に逸れますが、Next.js アプリケーションを実行する際、**Node ランタイム** と **Edge ランタイム** という 2 つの実行形態があります。
Edge ランタイムは Node ランタイムより**軽量かつ制限された** 環境になります。

### Node ランタイム

- Node ランタイム上で `next start` や `node server.js` 等で Next.js サーバーを起動する
- `fs`, `path`, `child_process`, `crypto` など **標準の Node API** をフルに使える
- ほぼ全ての npm パッケージが動作
- Middleware など一部機能は Edge ランタイムを想定しているが、Node ランタイム上でも**Edge ランタイム互換**としてちゃんと動作する
- Self Hosting の多くは Node ランタイムで運用するパターンが主流
- デメリットとしてはランタイムが大きく、起動/メモリ消費が Edge ランタイムより大きい

### Edge ランタイム

- **Vercel の Edge** や **Cloudflare Workers** などはグローバルに配置された Edge サーバレス環境に分散デプロイが可能で、レスポンスが高速
- **高速なコールドスタート** が可能
- Next.js では **Middleware** がこの Edge ランタイムを想定。さらに **Server Components のストリーミング** では低レイテンシの恩恵を受けやすい
- デメリットとしては Node.js のサブセットとされる制限付きのランタイムなので`fs` や `child_process` などは使えない
- それらを使った npm パッケージも動作しない

### Node ランタイムと Edge ランタイムのまとめ

- **Node ランタイム** は互換性や自由度が高く、Self Hosting でも一番分かりやすい選択肢。
- **Edge ランタイム** は高速起動と世界中へ分散デプロイされるので、特に `Middleware` や `Streaming` と相性が良いが、API の制限に注意。
- **一部の Next.js 機能 (Middleware, Streaming, 他にも Server Actions など)** で Edge ランタイムが推奨される場合もあるが、Node ランタイムでも“エミュレート”して動くパターンがほとんど。

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
cache-handler.js に、Redis や Memcached などを使って **同じキャッシュキー** に読み書きするロジックを実装する必要があります。[^1]
:::

この設定を組み合わせると、複数サーバーでも同じキャッシュを共有でき、Vercel に近いスケーリングを自前で構成できます。

### 2. 画像最適化の改善 (Sharp / WebAssembly)

Next.js 12.3 リリース前後あたりから、WebAssembly フォールバック (`@squoosh/lib`の独自拡張) が導入され、Sharp がインストールされていなくても画像最適化が可能となりました。[^2]
しかしパフォーマンス面から Sharp が推奨されていたのですが、**Next.js 15 以降では Sharp が自動インストールされるようになり**、特に何もしなくても Sharp が使われる形に進化しました。

これによって、Self Hosting する場合でも画像最適化のためのセットアップが不要になり、ビルド手順がシンプルになりました。

また、WASM フォールバックも引き続き残っているので、環境の問題によって Sharp が使えない場合でも自動的に WASM にフォールバックしてくれます。

### 3. Middleware の改善

Next.js 14.1 のリリースで Edge ランタイムを想定していた Middleware が **Node.js 環境でも動作する** ようになりました。

Vercel のように複数リージョンでエッジ配信したい場合は、Nginx や CloudFront + Lambda@Edge 等の追加構築が必要になります。

### 4. Runtime Environment Variables

Next.js は **ビルド時** と **ランタイム時** の両方で環境変数を扱えます。
クライアント側に埋め込む場合は **`NEXT_PUBLIC_`** が必須ですが、これはビルド後の JavaScript に直接書き込まれる点に注意が必要です。
一方、サーバーコンポーネントが `process.env.*` を参照する場合は、Docker や ECS の起動時に設定した値を実行中に読み込めます。[^3]
Self Hosting では `.env` ファイル管理や CI/CD の注入などを自力で行い、Vercel のような専用 UI はありませんが、同じコンテナでも環境変数を差し替えて動かせるメリットがあります。

### その他 Self Hosting で動作するか気になる機能

#### 5. Streaming (Server Components + Suspense)

Next.js の App Router では、Server Components + Suspense を使ってレスポンスをチャンク単位で送るストリーミングが可能です。
セルフホスティングでも、**Node.js 上で `next start`** している限り特別な設定は不要で、段階的にコンテンツを返せます。

ただし、**Nginx などのリバースプロキシ経由の場合** はレスポンスをまとめて返す設定があるとストリーミングはされません。
必要に応じてバッファリングを無効化すると、段階的な表示による高速な UX を提供できます。

#### 6. Server Actions

Server Actions は、フォーム送信や簡単な API 呼び出しをサーバーコンポーネント内で扱える機能です。
**Vercel 固有**ではなく **Next.js 本体の機能** なので、Self Hosting (Node ランタイム)でも特別な設定なしに動作します。

## Vercel / Self Hosting / Render / OpenNext の比較

ここでは Next.js を本番運用する際の代表的な選択肢を、いくつかの観点で比較してみます。
| | **Vercel** | **OpenNext** | **Amazon ECS** | **Render** |
|-----------------------|------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------|
| **デプロイの簡単さ** | - Git 連携だけでデプロイ可能<br>- CI/CD が標準搭載<br>- Preview も標準搭載| - デフォルト設定で AWS/Cloudflare のリソースを自動配置してくれるので比較的簡単<br>- ただし **AWS のデプロイに時間がかかる** | - プロダクションレベルだと AWS アーキ設計や詳細な設定が必要<br>- 自前で CI/CD を組む必要がある | - Git 連携だけでデプロイ可能<br>- CI/CD が標準搭載<br>- Preview も標準搭載 |
| **スケーリング** | 自動スケール | 自動スケール | - **ECS / Fargate + ALB** などで柔軟にスケール<br>- SRE 力 💪 | - 手動/動的スケールが選択可能 |
| **Edge 配信** | - Middleware が Edge で動作<br>- ServerActions や Streaming も Edge で動作 | - **CloudFront + Lambda@Edge** で Web アプリをエッジで配信可能<br>- 但し**Streaming と Edge 配置の併用は現状出来なかった** | - ECS を Edge には置けないため考慮外<br>- 将来 Middleware が Next.js 本体から分離出来れば Middleware だけ CloudFront+Lambda@Edge におけるかも(願望) | 単一リージョンでの運用 |
| **キャッシュ機能** | Full 機能標準搭載 | デフォルトで S3 や DynamoDB など**AWS リソースへの Cache 連携**が簡単 | - 自前で Redis 等を用意して CacheHandler の実装が必要<br>- CloudFront や ALB レイヤー含めて全体の Cache の調整が必要 | 自前で Redis 等を用意して CacheHandler の実装が必要 |
| **デプロイの柔軟性** | - **Vercel 独自インフラ**に縛られる<br>- ほぼ設定不要でラク | - AWS / Cloudflare の サービス連携が前提<br>- **OSS** なので自由度は高いがバージョン追随が必要 | - **VPC, SG, ALB** 等を自由に組み合わせ可能<br>- AWS 全般の豊富なオプションがある反面、**設定と保守**はユーザ負担 | PaaS なので AWS などよりはシンプルだが，自由度も少なくなる |
| **運用負荷** | - Vercel がオールインワン管理<br>- モニタリング等も専用 UI | - サーバレス × エッジで大規模に強い<br>- 周辺サービス (S3, DynamoDB etc.) の理解が必要<br>- OpenNext のアップデート追随が必要 | - インフラ周りを自力で構築<br>- モニタリング, ログ, デプロイフローも作ればある | CI/CD は自動 |
| **コスト/趣味用 (小〜中規模)** | - **Hobby プラン** (無料枠あり)<br>- ビルド回数や帯域などに制限<br>- 個人ブログや社内ツールなら十分 | - AWS / Cloudflare の無料枠を超えると課金<br>- トラフィックが少なければ月数ドル程度 | - ECS / EC2 の無料枠は基本的に無い<br>- **ライトな構成**にしても$10〜30/月はかかる<br>- 運用規模が小さいならそもそも ECS 選択のメリットが薄い | - Hobby プラン: $0/月あり<br> - 小規模なら無料インスタンスで始められる<br> - Starter レベルのビルド/帯域が含まれるが超過すると課金が発生 |
| **コスト/実運用 (中〜大規模)** | - **Pro/Enterprise プラン**<br>- リクエスト数や帯域に応じて**従量課金**が発生<br>- トラフィックの多いサイトだと数百〜数千ドル/月になる | - AWS / Cloudflare の利用に応じた課金<br>- 大量アクセス時は従量課金が急増しやすいので注意 | - **EC2 / Fargate / ALB**等の AWS 課金<br>- 大規模なら数百〜数千ドル/月も珍しくない | - Professional: $19/人/月 + 従量課金<br> - 組織向け: $29/人/月 + 従量課金 |

## おさらい

- 「Next.js 15 から Vercel と同じ機能を全部自前で再現できるの？」
  - **Next.js の主要機能は自前で再現しやすくなった**が、Edge 配信などの最適化を自前で構築することは不可能
- 「Custom Next.js Cache Handler を使わないといけないの？」
  - **複数サーバー・複数インスタンス構成**でキャッシュを共有する場合はほぼ必須。単一インスタンスの場合はデフォルトのファイル/インメモリキャッシュで十分戦える。
- 「Middleware では Node API は使えないの？」
  - Middleware は Node ランタイムでも動作可能になりはしたが、依然として Edge ランタイムを想定していて build 時に Edge ランタイム想定のコードに変換されるため Node API は使えない。
- 「Server Actions は Vercel じゃないと動かないの？」
  - Server Actions は**Next.js 本体の機能**です。**Self Hosting (Node.js)** でも問題なく動きます。

## まとめ

Next.js 14 / 15 で Self Hosting しやすくなりました。しかし、Vercel が提供する**グローバルへの Edge 配信**や４種のキャッシュハンドリングなどは依然として強みがあります。
プロジェクトの要件 (コスト, 運用負荷, グローバル展開, リクエスト量) に応じて **Vercel / OpenNext / Container / PaaS** などの技術を選択しましょう。

今後のアップデートでも Self Hosting 向けの機能に力を入れていくと予想され、**Custom Cache Handler**をはじめ、これからも大きく改善されていくと思います。
キャッシュ戦略やインフラ設計を組み立てれば、**Vercel に頼らず**とも大規模な Next.js サイトを運用できるようになってきていると言えるでしょう。

## PR

現在株式会社 tacoms では全ポジションで絶賛採用募集中です！！
是非気軽にカジュアル面談からお話ししましょう 👏

https://www.tacoms-inc.com/recruit

## PR2

[React Tokyo](https://discord.gg/5B9jYpABUy) というコミュニティが出来ました 🙌
詳しくは[こちらの記事](https://zenn.dev/dai_shi/articles/9f2760086fb31a)に書かれていますが，もし React に興味あるよ，好きだよという方は是非ご参加ください 🙌
1 月はオフラインイベントもありますよ＼(^o^)／

以上！

[^1]: [akfm_sato](https://x.com/akfm_sato) さんの書かれた『[Custom Next.js Cache Handler - Vercel 以外での Next.js キャッシュ活用](https://zenn.dev/akfm/articles/nextjs-cache-handler-redis)』を参考にしてください
[^2]: 残念ながらどこで導入されたか見つけることが出来ませんでした 🙏ChatGPT に聞いて 12.3 と強く言い張るのでここでは 12.3 としています
[^3]: 実行中に環境変数を読む方法は[ドキュメント](https://nextjs.org/docs/app/building-your-application/deploying#environment-variables)を参考にしてください
