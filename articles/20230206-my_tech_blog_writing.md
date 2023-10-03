---
title: "私のテックブログの書き方 | Offers Tech Blog"
emoji: "📑"
type: "idea"
topics:
  - "テックブログ"
  - "技術文書"
published: true
published_at: "2023-02-06 09:00"
publication_name: "overflow_offers"
---

<!-- textlint-disable overflow-techblog/prh-rules -->

# はじめに

こんにちは！

プロダクト開発人材の副業転職プラットフォーム [Offers](https://offers.jp/) を運営する株式会社 [overflow](https://overflow.co.jp/) で Offers のフロントエンドを開発している [fumiya](https://twitter.com/fumiya_itsc) です。

前回たくさん記事を書くことを宣言したものの、会社のテックブログに載るという重圧からなかなかテック記事を書けずにいました。

今回は、会社のテックブログに載ることを前提にして、自分なりにどういった内容であれば納得して公開できるかということを考えていきたいと思います。

本文へ入る前になぜ私は重圧を感じているのかを整理しておきます。
同じような悩みを持っている方の役立つ内容になっていれば嬉しいです。

<!-- 本来はテック記事なのですが、タイトルと合わせるためブログとしています。タイトルもテック記事よりもテックブログの方がキャッチーなのでテックブログとしています。 -->

# 私はなぜテックブログを書けないのか

先程重圧と書きましたが一体何が重圧なのでしょうか？
私は自分がテック記事を書く時に、公式ドキュメントやネットに公開されているテック記事と似たような内容を似たようなレベルで書きたくないと考えています。
書こうと思ったら他の方がまとまった内容を書いていたり、自分が書く内容の上位互換が既にあって書くことをやめてしまいます。

この記事ではそんな私が上記の気持ちとどう折り合いをつけてテック記事を書いたら良いか must(絶対にやる)と should(出来ればやる)まとめたものになります。
今後自分が書く記事はこの記事の方針に則って書いていこうと思います。

# must ルール

must ルールはテック記事を書く上で必ず満たすべき条件になります。

1. 自分の頭で考えられている

2. 間違ったことを書かない[^1]

3. この記事は誰の役にも立たない、とは考えないようにする[^2]

# should ルール

should ルールはテック記事を書く上でなるべく満たすべき条件になります。
しかし、そこまで肩肘張らず気楽に考えても良いんじゃないかとも思うもので、
今の自分が少し意識してみようかなというものを上げてみました。

- 文章を構造的に書く
  書く前に一度どういう流れで書くか考える。[^3]

- 技術系の単語を正しく書く

  例えば"javascript" ではなく "JavaScript" と書く。

- 正しい日本語を使う
  デスマス調、である調など、書き方はある程度揃える。

# 記事内容について

ここからは must ルールを前提にどのような記事であれば良さそうかを考えました。[^4]

- 基礎系

  - 普段技術[^5]をどう使っているか・そうしている理由
  - 難しいコトを分かりやすい文章で説明する[^6]
  - これまでにまとまっていないものをまとめる

- 実践系

  - 複数の技術を使って組み合わせについて書かれている
  - 想定された使い方ではないが有用な手段
  - アンチパターン
  - ハンズオン

- tips 系

  - 個別のエラーに関して
  - 個別の設定に関して

# まとめ

私はなぜテックブログを書けないのかを考えつつ、どうしたらその泥沼から抜け出せるかを考えてみました。
must ルールや should ルールは一見縛りに見えますが、指針として役に立ってくれるのではと思っています。（ルールが全く無いというのも何していいか分かりづらいですよね）
今後、私が書いていく記事はすべて（たぶん）must ルールを守り、should ルールを守ろうとしながら、記事内容についてで上げた系統の記事を書いていくと思います。

これまでなかなか記事がかけず、悩んでいる方に少しでも参考になったなら幸いです。

# 最後に

私は決して世の中に出回るすべての記事が上記で書いた must ルールや should ルールが適用されるべきだとは考えていません。
単純に現状の自分が何故テック記事書けないのだろうかと考え、そこから脱するためには何が必要かを考えて自身で使おうと思ったモノで、それを共有したものになります。


# 関連記事

https://zenn.dev/offers/articles/20220512-awesome-texlint-correction
https://zenn.dev/offers/articles/20221226-tech-blog-rewards-and-operations

[^1]:
    勘違いにより間違ったことを書いてしまうこともありますが、裏取りはしっかりとするべきと考えます。
    多少重圧がかかりますが、ここは許容しなければいけないと思います。
    それでも間違ってしまうこともありますがその時は追記して内容を修正するか、記事を取り下げると良いと思います。

[^2]: ルールとは少しズレますがこの気持ちを持っていた方が良い記事を書ける気がします！
[^3]: アウトプットすることが重要なため、少し理解し辛い文章だと思い留まらずに公開する。
[^4]: 基礎系にまとめたものは must や should と役割が似ていますが、より記事に近いと思ったのでこちらにまとめました。
[^5]:
    特定技術に限らず設計の考えや普段の習慣なども含む。
    既に記事化されていても、もう少しうまく説明できそうだと思ったら取り組むと良いと思います。

[^6]:
    公式ドキュメントは英語から翻訳されているため読みづらく、意図も伝わりづらいモノが多いです。
    なので、分かりやすい文章はそれだけで価値になると思います。
    冗長かな？と思ってもあえて多くを書くことで理解出来る文章が出来ると思います。