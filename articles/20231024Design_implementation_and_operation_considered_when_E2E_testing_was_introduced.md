---
title: "E2Eテストを導入した時に考えた設計と実装と運用について | Offers Tech Blog"
emoji: "✅"
type: "tech"
topics:
  - "テスト"
  - "e2e"
  - "playwright"
published: true
published_at: "2023-10-24 09:00"
publication_name: "overflow_offers"
---

こんにちは、[Offers](https://offers.jp/)と[Offers MGR](https://offers-mgr.com/lp/) を運営している株式会社 [overflow](https://overflow.co.jp/) のフロントエンジニアの fumiyaki です。

今回は、半年前にOffersMGRに`Playwright`を使った`E2E` テストを導入した話ができればと思います。

過去に `Datadog Synthetics Test` と `Playwirght`周りの記事を書いていたので合わせてご覧いただけると幸いです。

- [Playwrightを使ってコードベースなE2Eテストに移行した話 | Offers Tech Blog](https://zenn.dev/overflow_offers/articles/20230116-e2e-test-with-playwright)
- [Datadog Synthetics Test 導入フロー | Offers Tech Blog](https://zenn.dev/offers/articles/20220822-datadog-synthetics-test-introduction)
- [Datadog SyntheticsのSubtestでブラウザテストをComponent化する｜Offers Tech Blog](https://zenn.dev/offers/articles/20220915-datadog-synthetics-introduce-subtest)

## 導入の背景と目的

半年前のOffersMGRはフェーズ的にテストケースを厚くして守りを固めるよりも、最低限の守りで新機能のリリースを優先する攻めの姿勢を取っていました。

今回のE2Eテストの導入は最低限の守りを固めていくと同時に今後のテスト拡充の布石にしたいと考えていました。

テーマと外れるため詳しくは説明しませんが、既に導入されて実施していたテストに軽く触れます。

| Static Test | LintとBuildを実行して正常に動作するか検証 |
| --- | --- |
| UI Test | StoryBookでVRT、Snapshotを実行してコンポーネントを検証 |
| A11y Test | StoryBookでA11yを実行してコンポーネントを検証 |
| Datadog Synthetics Test | ブラウザでのシナリオを検証 |

実はOffersMGRでも`Datadog Synthetic Test` を用いてテストを行っていました。

しかし、シナリオが2つしか無くカバー範囲も狭く、デグレが起こっても検知出来ない状況でした。

今回のPlaywrightを使ったE2Eテストの導入では既存シナリオの移行とOffersMGRの全ページが健全に動作しているかのチェックを目的としました。

## 今後を見据えた設計と運用

BtoCももちろんですが、BtoBのプロダクトは特に成長していくとともに安定したサービス提供が求められるのでテストの重要度は高くなります。

そのため、E2Eテストや結合テストの拡充は必然的に実施していくことになります。

一応、特定のComponentに対象を絞る事によってE2Eテストでも結合テストが出来ます。

しかし、E2Eテストは結合テストと比べ実行速度が遅く、最終的なテスト量はE2Eテスト < 結合テストが理想で、E2Eテストで結合テストを書いてしまうことを避けたいと考えました。

そこで、以下のようなフォルダ構成を考えました。

```jsx
.
├── entire   // 全体に関わるテスト
│   ├── authorityCheck // 認証認可が正常に動作している
│   ├── fetchedData    // 外部との連携が正常に動作している
│   └── workingAllPage // 全てのページが正常に動作している
├── scenario // ユーザーが使用する所を想定したシナリオ集
│   ├── from_sign_up_to_contact_invitation // サインアップから担当者の招待まで
│   ├── onboarding                         // オンボーディング
│   └── service_skilled_person_how_to_use  // サービス熟練者の使い方
└── scene    // 個別のケース集
    ├── when_sign_in_go_to_home_page                // サインインの後にホームに遷移するか
    └── is_it_reflected_after_updating_your_profile // プロフィールを更新後、反映されているか
```

上記は今後を想定したテストケース集も含んでいます。

今回の追加すべきテストは`workingAllPage`のテストで全てのページが正常に動作しているかを確認します。

### entire

全体で実施したいテストをここにフォルダを区切って置いていきます。

今回想定したのは

- 認証認可が正しく動作して、未ログインユーザーとログインユーザーでページが見れる見れないのテスト
- APIとの繋ぎ込みが正しく行われ、画面に正しそうな値が表示されているかテスト（正しそうなというあやふやな表現は後の**変更に強いE2Eテストの実装を目指す**で説明します）
- 全てのページが正常に画面表示出来ているかを確認するテスト
    
    OffersMGRには既に多くのページがあり、それぞれのページでチェックしたい項目も違ったため、`workingAllPage`配下に同ページ名のテストを作成して項目をチェックしていきました。
    
    ```jsx
    .
    └── workingAllPage
        ├── 404.spec.ts
        ├── alert.spec.ts
        ├── company_edit.spec.ts
        ├── employee.spec.ts
        ├── employee_edit.spec.ts
        ├── logout.spec.ts
        ├── lp.spec.ts
        ├── project.spec.ts
        └── terms.spec.ts
    ```
    

### scenario

OffersMGRの想定ユーザーがプロダクトを使う一連の操作が正しく動作するかをテストします。

- サインアップから担当者の招待まで
- オンボーディング
- サービス熟練者の使い方

### scene

全体に関わるものでもなく、scenarioほど大きくないものはsceneに入れていきます。

ここは結合テストで出来る範囲のものが多く、増やしすぎないように意識したいです。

Storybookのインタラクションテストを追加してここを無くすことも出来るのですが、現在のフェーズとリソースだと管理するべきテストを増やすよりも一旦ここに置いておく方が良いと判断しました。

- サインインの後にホームに遷移するか
- プロフィールを更新後、反映されているか

## 変更に強いE2Eテストの実装を目指す

設計と運用は型を作ったので迷うこと無くテストを書き始めることが出来ると思います。

しかし、実装にも気を配らなければテストは簡単に壊れ、運用が面倒になってしまいます。

何度も壊れるテストはやがて更新されずにひっそりと姿を消していくものです。

そこで、どうしたら壊れにくい実装となるかを考えました。

大事な点は以下になります。

- セレクタの直接指定は極力避ける
- data-testid属性は極力使用しない
- 極力[page.getByRole()](https://playwright.dev/docs/locators#locate-by-role)を使用する
- page.getByTextを使う時は正規表現を使用する

### セレクタの直接指定は極力避ける

ユーザーに見える画面は日々更新されていくものです。

もしセレクタでテストを書いていた場合、そのページが更新されると多くのテストが一斉に火を吹きます。

HTMLの構造に依存したテストは極力避けるべきです。

### data-testid属性は極力使用しない

これは反対意見も多くあるかとは思いますが、E2Eテストの目的とギャップが生じてしまうため、data-testid属性は極力使用しない方が良いと考えています。

私の考えるE2Eテストの目的は、実際にユーザーが使用するUIを通してプロダクトの動作をテストすることだと思います。

しかし、data-testid属性を使用すると、その属性が存在することを前提としたテストを作成することになってしまい、UIやロジックの変更に対して影響を受けにくくなることがあると思います。

結果、UIが変わった場合でもテストが通ってしまうような状況が生じてしまうので、極力data-testid属性は極力使用しない方が良いと考えます。

後、名前を考えるのもプロダクションで削除するのも面倒です。

### 極力page.getByRole()を使用する

**`getByRole()`**は役割に基づいて要素を取得します。

これはスクリーンリーダー等のアクセシビリティツールがウェブページの要素を解釈する方法と同じことを実現します。

**`getByRole()`**を使うことでアクセシビリティの問題を見つける事ができるようになります。

他にも要素の役割はUIの構造が変わっても変わりにくいこともあります。

**`getByRole()`**を使用すると壊れにくいテストが書けます。

また、他人の書いたテストでも役割ベースで書かれているため、テストの意図や動作が読み取りやすいというメリットもあります。

### page.getByTextを使う時は正規表現を使用する

page.getByTextと正規表現を合わせて使うことで、テストに柔軟性を持たせる事ができます。

もしテキストが微妙に変更される場合や、テキストの一部が動的に変わる場合でも、正規表現を使用すれば適切に要素を取得することが出来ます。

これにより、テストが壊れにくくなりテストの修正する頻度が減ります。

## まとめ

ここまでE2Eテストを導入した時に考えた設計と実装と運用についてまとめました。

この記事が実装や運用のプラスになれば嬉しく思います。

## ****関連記事****

[Playwrightを使ってコードベースなE2Eテストに移行した話 | Offers Tech Blog](https://zenn.dev/overflow_offers/articles/20230116-e2e-test-with-playwright)

[Datadog Synthetics Test 導入フロー | Offers Tech Blog](https://zenn.dev/overflow_offers/articles/20220822-datadog-synthetics-test-introduction?redirected=1)

[Datadog SyntheticsのSubtestでブラウザテストをComponent化する｜Offers Tech Blog](https://zenn.dev/overflow_offers/articles/20220915-datadog-synthetics-introduce-subtest?redirected=1)
