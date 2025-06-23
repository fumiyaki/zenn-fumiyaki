---
title: "React Tokyo ミートアップ #6 イベントレポート"
emoji: "🔄"
type: "idea"
topics:
  - "react"
  - "イベントレポート"
  - "reacttokyo"
published: false
publication_name: "react_tokyo"
---

![](/images/20250623-react-tokyo-event-report/00.JPG)

お久しぶりです、東京を中心に活動する React 開発者のコミュニティ、React Tokyo でお疲れ様会の幹事をしているふみやです！
[x.com](https://x.com/fumiyaki_)の DM とかで新宿・恵比寿・渋谷で飲み会で使えるおすすめのお店を教えて貰えるととても喜びます 🤩

今回は 2025 年 6 月 20 日(金)に開催された「React Tokyo ミートアップ #5」にスタッフとして参加してきましたので、そのレポートをお届けします！

https://react-tokyo.connpass.com/event/354906/

## イベントの概要

React Tokyo ミートアップは、React Tokyo コミュニティにご参加いただいている方々の親交を深め、Discord（オンライン）上でも活発な交流ができるような土壌作りを目的としたオフラインイベントです。

メンバー同士の交流がメインのイベントとなっており、毎回決まるテーマに沿って行うグループワークや、交流会を行います。
React に関する LT も行われます！

今回は 6 度目の開催で、参加者は約 36 名でした。
![](/images/20250623-react-tokyo-event-report/01.JPG)

## 会場スポンサー LT

![](/images/20250623-react-tokyo-event-report/02.JPG)
オフラインイベントを応援していて、コミュニティを大切にしたいと話していました！
会場も貸してますよー！とのことです（今回はありがとうございます！）

## 飲食スポンサー LT

![](/images/20250623-react-tokyo-event-report/03.JPG)

サンドイッチとお酒を提供していただきました 🥪🍻
エンジニアのキャリアの壁打ちとか募集しているらしいので、AI 時代にキャリアをどう考えていくのか壁打ちさせてもらいたいですね 🙌

## メイントーク

https://speakerdeck.com/calloc134/reactnonei-bu-gou-zao-wozhi-tuteoku-react-tokyo-number-6-at-calloc134

かろっくさんの発表で React の内部構造について Deep Dive する内容でした 🤿
Discord や x でアイコンと名前は知っていたのですが、まさか学生さんだったとは驚きでした笑

スライドにかわいい図がたくさんあってかわいいスライドでした。
準備に凄く時間をかけている事が伝わる Deep Dive でした！

「React のドキュメントでは仮想 DOM はあまり使われていなくて、Fiber ノードと呼ばれている」「alternate、FiberRootNode」など終始感嘆しっぱなしでした 🫠(脳が溶けている様子)
皆も React のソースコード読もうぜ！

発表の最後に「Meta の React 開発チームの皆様に間違いなどあれば教えてね〜」って言ってたけど会場に Meta の人はいたのかな 🤔

## グループワーク

![](/images/20250623-react-tokyo-event-report/04.JPG)

今回のグループワークのテーマは【フォーム実装どうしている？】でした！

自分の参加したグループでは React Hook Form をメインで使っている方が多かったです。
バリデーションをエクセルで管理しつつ、そのデータからコード生成する仕組みが整っているなどの話があり興味深かったです！
次点で Conform で、TanstackForm は様子見という印象でした 👀

フォームの話のセットでバリデーションの話も上がり、「valibot 使いたいけど zod 使ってる」「テストめんどくさいよね・あまり書いていない・・・」などと悲しみ・苦しみの共有もされました 😭

## LT1

![](/images/20250623-react-tokyo-event-report/05LT1.JPG)
晴れ井戸さんの発表で JSX（インターフェース）について熱いぽえむが聞けました 🙌
XHP という記法が JSX のふるさと？という過去の話から JSX の未来について学びが多かったです 🖌️

## LT2

![](/images/20250623-react-tokyo-event-report/06LT2.JPG)
akfm さんの発表で DataLoader のすゝめ。
DataLoader はいいぞぉ、バッチングで待って N+１っぽいやつを解決するという話。
x(1)
x(2)
と書いて x([1,2])で送るのと同等の挙動になるらしいです（詳しくは[akfm さんの zenn の記事](https://zenn.dev/akfm/articles/desing-of-data-fetching)を確認してください 🙏）

## LT3

![](/images/20250623-react-tokyo-event-report/07LT3.JPG)
林原さんの React における MRAH 設計のススメ。
MRAH は Modular, Rendering, Adaptive, Hydration の頭文字らしいです！
特にハイドレーションについて詳しく解説されていました 🚿

## 交流会

交流会では、ドリンクとサンドイッチをお供にさまざまな話題で盛り上がっていました 🌼

![](/images/20250623-react-tokyo-event-report/08.JPG)
![](/images/20250623-react-tokyo-event-report/09.JPG)

## まとめ

久々に React Tokyo のイベントに参加して React にまつわる話がたくさん出来て楽しかったです。
オンライン・オフラインどちらも継続して盛り上げていきたいので皆さんも是非たくさんご参加ください！

イベント開催にあたり、会場、飲食のご支援を頂きました株式会社タイミー様、ロスカ株式会社様、そして日頃よりコミュニティをご支援いただいているコミュニティスポンサーの株式会社キッカケクリエイション様、Dress Code 株式会社様、株式会社 Rebase 様、 株式会社カケハシ様、誠にありがとうございました。
そして運営をサポートしてくださった、React Tokyo サポートチームの皆様、本当にありがとうございました。

React Tokyo では、今後もオンライン・オフラインでのイベントを企画していきます。
興味を持っていただけた方は、ぜひ React Tokyo の Discord にご参加ください！

それではまた次回のイベントでお会いしましょう！

![](/images/20250623-react-tokyo-event-report/10.JPG)
