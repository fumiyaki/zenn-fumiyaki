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
---

こんにちは、[Offers](https://offers.jp/)と[Offers MGR](https://offers-mgr.com/lp/)  を運営している株式会社  [overflow](https://overflow.co.jp/)  のフロントエンジニアの fumiyaki です。

今回は、半年前に OffersMGR に`Playwright`を使った`E2E`  テストを導入した話ができればと思います。

過去に  `Datadog Synthetics Test`  と `Playwirght`周りの記事を書いていたので合わせてご覧いただけると幸いです。

- [Playwright を使ってコードベースな E2E テストに移行した話 | Offers Tech Blog](https://zenn.dev/overflow_offers/articles/20230116-e2e-test-with-playwright)
- [Datadog Synthetics Test 導入フロー | Offers Tech Blog](https://zenn.dev/offers/articles/20220822-datadog-synthetics-test-introduction)
- [Datadog Synthetics の Subtest でブラウザテストを Component 化する｜ Offers Tech Blog](https://zenn.dev/offers/articles/20220915-datadog-synthetics-introduce-subtest)

## 導入の背景と目的

半年前の OffersMGR はフェーズ的にテストケースを厚くして守りを固めるよりも、最低限の守りで新機能のリリースを優先する攻めの姿勢を取っていました。

今回の E2E テストの導入は最低限の守りを固めていくと同時に今後のテスト拡充の布石にしたいと考えていました。

テーマと外れるため詳しくは説明しませんが、既に導入されて実施していたテストに軽く触れます。

| Static Test             | Lint と Build を実行して正常に動作するか検証              |
| ----------------------- | --------------------------------------------------------- |
| UI Test                 | StoryBook で VRT、Snapshot を実行してコンポーネントを検証 |
| A11y Test               | StoryBook で A11y を実行してコンポーネントを検証          |
| Datadog Synthetics Test | ブラウザでのシナリオを検証                                |

実は OffersMGR でも`Datadog Synthetic Test`  を用いてテストを行っていました。

しかし、シナリオが 2 つしか無くカバー範囲も狭く、デグレが起こっても検知出来ない状況でした。

今回の Playwright を使った E2E テストの導入では既存シナリオの移行と OffersMGR の全ページが健全に動作しているかのチェックを目的としました。

## 今後を見据えた設計と運用

BtoC ももちろんですが、BtoB のプロダクトは特に成長していくとともに安定したサービス提供が求められるのでテストの重要度は高くなります。

そのため、E2E テストや結合テストの拡充は必然的に実施していくことになります。

一応、特定の Component に対象を絞る事によって E2E テストでも結合テストが出来ます。

しかし、E2E テストは結合テストと比べ実行速度が遅く、最終的なテスト量は E2E テスト < 結合テストが理想で、E2E テストで結合テストを書いてしまうことを避けたいと考えました。

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
- API との繋ぎ込みが正しく行われ、画面に正しそうな値が表示されているかテスト（正しそうなというあやふやな表現は後の**変更に強い E2E テストの実装を目指す**で説明します）
- 全てのページが正常に画面表示出来ているかを確認するテスト
  OffersMGR には既に多くのページがあり、それぞれのページでチェックしたい項目も違ったため、`workingAllPage`配下に同ページ名のテストを作成して項目をチェックしていきました。
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

OffersMGR の想定ユーザーがプロダクトを使う一連の操作が正しく動作するかをテストします。

- サインアップから担当者の招待まで
- オンボーディング
- サービス熟練者の使い方

### scene

全体に関わるものでもなく、scenario ほど大きくないものは scene に入れていきます。

ここは結合テストで出来る範囲のものが多く、増やしすぎないように意識したいです。

Storybook のインタラクションテストを追加してここを無くすことも出来るのですが、現在のフェーズとリソースだと管理するべきテストを増やすよりも一旦ここに置いておく方が良いと判断しました。

- サインインの後にホームに遷移するか
- プロフィールを更新後、反映されているか

## 変更に強い E2E テストの実装を目指す

設計と運用は型を作ったので迷うこと無くテストを書き始めることが出来ると思います。

しかし、実装にも気を配らなければテストは簡単に壊れ、運用が面倒になってしまいます。

何度も壊れるテストはやがて更新されずにひっそりと姿を消していくものです。

そこで、どうしたら壊れにくい実装となるかを考えました。

大事な点は以下になります。

- セレクタの直接指定は極力避ける
- data-testid 属性は極力使用しない
- 極力[page.getByRole()](https://playwright.dev/docs/locators#locate-by-role)を使用する
- page.getByText を使う時は正規表現を使用する

### セレクタの直接指定は極力避ける

ユーザーに見える画面は日々更新されていくものです。

もしセレクタでテストを書いていた場合、そのページが更新されると多くのテストが一斉に火を吹きます。

HTML の構造に依存したテストは極力避けるべきです。

### data-testid 属性は極力使用しない

これは反対意見も多くあるかとは思いますが、E2E テストの目的とギャップが生じてしまうため、data-testid 属性は極力使用しない方が良いと考えています。

私の考える E2E テストの目的は、実際にユーザーが使用する UI を通してプロダクトの動作をテストすることだと思います。

しかし、data-testid 属性を使用すると、その属性が存在することを前提としたテストを作成することになってしまい、UI やロジックの変更に対して影響を受けにくくなることがあると思います。

結果、UI が変わった場合でもテストが通ってしまうような状況が生じてしまうので、極力 data-testid 属性は極力使用しない方が良いと考えます。

後、名前を考えるのもプロダクションで削除するのも面倒です。

### 極力 page.getByRole()を使用する

**`getByRole()`**は役割に基づいて要素を取得します。

これはスクリーンリーダー等のアクセシビリティツールがウェブページの要素を解釈する方法と同じことを実現します。

**`getByRole()`**を使うことでアクセシビリティの問題を見つける事ができるようになります。

他にも要素の役割は UI の構造が変わっても変わりにくいこともあります。

**`getByRole()`**を使用すると壊れにくいテストが書けます。

また、他人の書いたテストでも役割ベースで書かれているため、テストの意図や動作が読み取りやすいというメリットもあります。

### page.getByText を使う時は正規表現を使用する

page.getByText と正規表現を合わせて使うことで、テストに柔軟性を持たせる事ができます。

もしテキストが微妙に変更される場合や、テキストの一部が動的に変わる場合でも、正規表現を使用すれば適切に要素を取得することが出来ます。

これにより、テストが壊れにくくなりテストの修正する頻度が減ります。

## まとめ

ここまで E2E テストを導入した時に考えた設計と実装と運用についてまとめました。

この記事が実装や運用のプラスになれば嬉しく思います。

## \***\*関連記事\*\***

[Playwright を使ってコードベースな E2E テストに移行した話 | Offers Tech Blog](https://zenn.dev/overflow_offers/articles/20230116-e2e-test-with-playwright)

[Datadog Synthetics Test 導入フロー | Offers Tech Blog](https://zenn.dev/overflow_offers/articles/20220822-datadog-synthetics-test-introduction?redirected=1)

[Datadog Synthetics の Subtest でブラウザテストを Component 化する｜ Offers Tech Blog](https://zenn.dev/overflow_offers/articles/20220915-datadog-synthetics-introduce-subtest?redirected=1)
