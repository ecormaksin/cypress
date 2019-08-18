[Cypressダッシュボード](https://on.cypress.io/dashboard)は記録されたテストにアクセスするサービス。

# 機能

## プロジェクトの編成

- ダッシュボードで記録するためにプロジェクトをセットアップする
- レコードキーをリセットするか、さらに追加する
- Cypressプロジェクトのアクセス権を変更する
- プロジェクトの所有権を移管する
- プロジェクトを削除する

## テスト実行結果を確認する

- 失敗、通過、保留、スキップされたテストの数を確認する
- 失敗したテストの全体のスタックトレースを取得する
- テストが失敗した時または`cy.screenshot()`を使った時のスクリーンショットを確認する
- テスト実行の全体のビデオまたはテストが失敗した時のビデオクリップを確認する
- 並行で実行されたかどうかも含めて、CI内でspecファイルがどのくらいの時間（速度）で実行されたかを確認する
- テストの関連したグルーピングを確認する。

## 組織を管理する

- 組織を作成、編集、削除する。
- それぞれの組織の使い方の詳細を確認する。
- 選択された請求プランに支払う。

## ユーザーを管理する

- ユーザーを組織に招待してユーザーのロールを編集する
- 組織への参加を承認または拒否する

## Githubに統合する

- [ステータスチェック](https://docs.cypress.io/guides/dashboard/github-integration.html#Enabling-GitHub-integration-for-a-project)のコミットを経由したGithubワークフローとCypressのテストを統合する
- [プルリクエスト](https://docs.cypress.io/guides/dashboard/github-integration.html#Pull-request-comments)を通じてCypressをGithubへ統合する

### テストランナーでテストの実行を確認する

「Runs」タブからすべてのプロジェクトのテストの実行を確認できる。

他の疑問点は[FAQ](https://docs.cypress.io/faq/questions/dashboard-faq.html)を参照。

# サンプルプロジェクト

[ダッシュボード サービス](https://on.cypress.io/dashboard)にログインすると[パブリックなプロジェクト](https://docs.cypress.io/guides/dashboard/projects.html#Public-vs-Private)であれば何でも見ることができる。

- [cypress-example-recipes](https://dashboard.cypress.io/#/projects/6p53jw)
- [cypress-example-kitchensink](https://dashboard.cypress.io/#/projects/4b7344)
- [cypress-example-todomvc](https://dashboard.cypress.io/#/projects/245obj)
- [cypress-example-piechopper](https://dashboard.cypress.io/#/projects/fuduzp)
