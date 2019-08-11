---
title: 再試行
---

動的なWebアプリケーションのテストを支えるCypressの主な機能の1つが再試行。  

# コマンド vs アサーション

Cypressのテストで扱えるメソッドには2種類ある: コマンドとアサーション  

例  

```javascript
cy.get('.todo-list li')     // コマンド
  .should('have.length', 2) // アサーション
```

現代のWebアプリケーションは非同期のため、指定したクラス`todo-list`を持つすべてのDOM要素を検索することはできない。また、そのエレメントが2つしかないことを検証することもできない。前述のことがうまくいかないケースはたくさんある。  

- コマンドを実行するまでにアプリケーションがDOMを更新していない。
- DOMの要素を生成する前にバックエンドのレスポンスを待機している。
- アプリケーションがDOMに結果を表示する前に負荷の高い計算をしている。

Cypressは`cy.get()`が失敗したら、タイムアウトが発生するまで再試行する。

# 複数のアサーション

単一のコマンドの後に実行する複数のアサーションは、順番に再試行される。1番目のアサーションがパスした時は、1番目→2番目のアサーションが再試行される。1番目と2番目のアサーションがパスした時は、1番目〜3番目のアサーションが再試行される。以降は同じ要領。

# すべてのコマンドが再試行されるわけではない

CypressはDOMを検索するコマンドのみ再試行する: `cy.get()`, `.find()`, `.contains()`など。

## なぜ再試行 _されない_ コマンドがあるのか

テストでアプリケーションの状態を変える可能性がある場合、コマンドは再試行されない。（`.click()`など）

# ビルトイン アサーション

Cypressのコマンドには、再試行するコマンドを生み出すビルトイン アサーションを持っているものが多い。

例

```javascript
cy.get('.todo-list li')     // コマンド
  .should('have.length', 2) // アサーション
  .eq(3)                    // コマンド
```

`.eq()`コマンドは紐付けられたアサーションがなくても、前の処理で生成されたリストに該当のインデックスを持つ要素が見つかるまで再試行する。

再試行できないコマンドでも、ビルトインの待機処理を持っているものがある。`.click()`コマンドは要素が操作可能になるまで待機する。

Cypressはブラウザを使う人間のユーザーのように振る舞おうとする。

- 要素をクリックできるか？
- 要素が見えているか？
- 要素が他の要素に隠れていないか？
- 要素が使用不可か？

`.click()`コマンドは上記のようなビルトインのアサーションのチェックがパスするまで自動的に待機して、1度だけクリックしようとする。

# タイムアウト

デフォルトでは、再試行するコマンドは[`defaultCommandTimeout`](https://docs.cypress.io/guides/references/configuration.html#Timeouts)により4秒まで待機する。このタイムアウトは、設定ファイル、CLIパラメーター、環境変数、実装で変更できる。

例: コマンドラインによりタイムアウトを10秒へ変更する場合

```shell
$ cypress run --config defaultCommandTimeout=10000
```

このオプションを上書きする他の例については、[Configuration: Overriding Options](https://docs.cypress.io/guides/references/configuration.html#Overriding-Options)を参照。全体のタイムアウトを変更することは非推奨。各コマンドの`{ timeout: ms }`オプションを指定する方がよい。

# 最後のコマンドだけ再試行される

```javascript
cy.get('.new-todo').type('todo B{enter}')
cy.get('.todo-list li')         // 検索してすぐに<li>が1つ見つかる
  .find('label')                // 1個目の<li>で再試行する
  .should('contain', 'todo B')  // 1個目の<li>では絶対に成功しない
```

## クエリーのマージ

推奨される1つ目の解決策は、要素を検索するコマンドの不必要な分割を避けること。

```javascript
cy.get('.todo-list li label')
  .should('contain', 'todo B')
```

### 別の例

```javascript
// 非推奨
// 最後の'its'だけ再試行される
cy.window()
  .its('app')     // 1回だけ実行される
  .its('model')   // 1回だけ実行される
  .its('todos')   // 再試行される
  .should('have.length', 2)

// 推奨
cy.windows()
  .its('app.model.todos') // 再試行される
  .should('have.length', 2)
```

## コマンドとアサーションを交互に行う

不安定なテストを修正するもう1つの方法。長いテストを書く時はいつでも、コマンドとアサーションを交互に行うことが推奨される。

```javascript
it('adds two items', function() {
  cy.visit('/')

  cy.get('.new-todo').type('todo A{enter}')
  cy.get('.todo-list li')         // コマンド
    .should('have.length', 1)     // アサーション
    .find('label')                // コマンド
    .should('contain', 'todo A')  // アサーション

  cy.get('.new-todo').type('todo B{enter}')
  cy.get('.todo-list li')         // コマンド
    .should('have.length', 1)     // アサーション
    .find('label')                // コマンド
    .should('contain', 'todo B')  // アサーション
})
```

# 関連情報

- 自作の[カスタムコマンド](https://docs.cypress.io/api/cypress-api/custom-commands.html)に再試行する機能を追加できる。[cypress-xpathへのこのプルリクエスト](https://github.com/cypress-io/cypress-xpath/pull/12/files)を参照。

- サードパーティのプラグイン[cypress-pipe](https://github.com/NicholasBoll/cypress-pipe)を使ってアサーションに紐付けられた関数を何でも再試行できる。

- [Cypressはコールバックすべき](https://glebbahmutov.com/blog/cypress-should-callback/)というブログの投稿の中の再試行の例を参照。
