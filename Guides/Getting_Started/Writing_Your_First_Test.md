# 最初のテストを書く

https://docs.cypress.io/guides/getting-started/writing-your-first-test.html#Add-a-test-file

## テストファイルを追加する

- Cypressをインストールして開いていることが前提。

1. `sample_spec.js`ファイルを作成する。
2. Cypressが仕様の一覧を更新するのを確認する。
3. Cypressをインタラクティブ モードで起動する。

生成された`cypress/integration`ディレクトリの下に新しいファイルを作成する。
ファイルを作成すると、Cypressテストランナーにより統合テストの一覧にファイルが追加される。
Cypressは仕様のファイルを監視して、変更があれば自動的に表示する。

## 簡単なテストを書く

1. 最初の通過テストを書く。
2. 最初の失敗テストを書く。
3. Cypressが即時に再読み込みするのを確認する。

- `sample_spec.js`に好きなIDEで以下のコードを追加する。

<details><summary>サンプルコード</summary><div>

```
describe('My First Test', function() {
    it('Does not do much!', function() {
        expect(true).to.equal(true)
    })
})
```
</div></details>  

---

- テストを書き換えて失敗させる。

<details><summary>サンプルコード</summary><div>

```
describe('My First Test', function() {
    it('Does not do much!', function() {
        expect(true).to.equal(false)
    })
})
```
</div></details>  

## 実際のテストを書く

**堅牢なテストは通常3つのフェイズをカバーする:**

1. アプリケーションの状態をセットアップする。
2. 動作させる。
3. 結果を確認する。

上記のフェイズは次のようにも表現される。「Given(〜の前提条件で), When(〜する時), Then(〜になる)」「Arrange(整える), Act(操作する), Assert(検証する ※日本語だと主張・断言の意味だが適切な対応語がわからない)」

以下では、概念の範囲を狭めてCypressのコマンドに対応したステップを例示する。

1. Webページへアクセスする。
2. 要素を探す。
3. その要素とやりとりする。
4. ページの内容について確認する。

### 1. Webページへアクセスする。

Cypressが用意してくれている[Kitchen Sink](https://docs.cypress.io/examples/examples/applications.html#Kitchen-Sink) ページへアクセスする。

`cy.visit()`を使って、簡単に指定したURLへアクセスできる。

最初のテストを次のように書き換える。

<details><summary>サンプルコード</summary><div>

```
describe('My First Test', function() {
    it('Visits the Kitchen Sink', function() {
        cy.visit('https://example.cypress.io')
    })
})
```
</div></details>  

---

Cypressテストランナーが更新され、次のことが行われている。

1. コマンドログが新しい`VISIT`アクションを表示している。
2. Kitchen Sinkアプリケーションがアプリプレビュー画面に読み込まれている。
3. テストはグリーン(OK)、ただしアサーションはない。
4. `VISIT`はページ読み込みが終わるまで、青い保留状態になる。

#### 制御可能なアプリだけをテストすること

制御できないアプリをテストすべきではない理由

- テストが壊れたらすぐに修正する責任がある。
- 一貫した結果を得ることができないABテストをしているかもしれない。
- （悪意のある）スクリプトと検知してアクセスをブロックされるかもしれない。（Googleはこれをやる）
- Cypressの動作を阻害するセキュリティ機能があるかもしれない。

Cypressが目指しているのは、**みなさん自身のアプリケーション**の構築とテストを行うために日々使うツールになること。  
CypressはWebの自動化ツールという目的はない。

### 2. 要素を探す。

要素を探すには`cy.contains()`を使う。

テストに追加して、何が起こるかを見てみる。

<details><summary>サンプルコード</summary><div>

```
describe('My First Test', function() {
    it('finds the content "type"', function() {
        cy.visit('https://example.cypress.io')

        cy.contains('type')
    })
})
```
</div></details>  

上記コードで`type`をページ上に存在しない`hype`に間違えるとすぐエラーになる。

### 3. 要素をクリックする

前述のコマンドの末尾に`.click()`を追加するだけでよい。

<details><summary>サンプルコード</summary><div>

```
describe('My First Test', function() {
    it('clicks the link "type"', function() {
        cy.visit('https://example.cypress.io')

        cy.contains('type').click()
    })
})
```
</div></details>  

#### おまけ
- コード入力時のIntelliSenseを有効にする方法（以下のいずれか。Visual Stduio CodeではRemote-SSHを使ってリモートアクセスした場合でも有効になったが、Atomではrmate＋SSHでも、リモート上で直接AtomをインストールしてTypeScriptサポートを有効にしても機能しなかった。）
    - 各テストファイル（*_spec.js）の1行目に以下を設定する。  
      ```
      /// <reference types="Cypress" />
      ```

    - `cypress/support/index.d.ts`に以下を設定する。  
      ```
      /// <reference types="Cypress" />
      ```
    
      - [リファレンス](https://docs.cypress.io/guides/tooling/intelligent-code-completion.html#Set-up-in-your-Dev-Environment) では、カスタムコマンドを定義している場合も含めて以下のように記述されている。  
        ```
        // type definitions for Cypress object "cy"
        /// <reference types="cypress" />

        // type definitions for custom commands like "createDefaultTodos"
        /// <reference types="../support" />
        ```
    
    - `tsconfig`で参照タイプを定義する。
        - `cypress`フォルダー直下に`tsconfig.json`を追加する。  
          ```
          {
              "compilerOptions": {
                  "allowJs": true,
                  "baseUrl": "../node_modules",
                  "types": [
                      "cypress"
                  ]
              },
              "include": [
                  "**/*.*"
              ]
          }
          ```

### 4. 検証する

- 新しいURLが期待したものかどうかを検証してみる。`.should()`を使う。

<details><summary>サンプルコード</summary><div>

```
descrive('My First Test', function() {
    it('clicking "type" navigates to a new url', function() {
        cy.visit('https://example.cypress.io')

        cy.contains('type').click()

        // Should be on a new URL which includes '/commands/actions'
        cy.url().should('include', '/commands/actions')
    })
}) 
```
</div></details>

#### コマンドとアサーションをさらに追加する

- 1つのテストで複数の操作とアサーションが可能。
- `cy.get()`
    - CSSクラスに基づき要素を選択する。
- `.type()`
    - 指定した要素にテキストを入力する。
- `.should()`
    - テキストが入力されているか検証する。

<details><summary>サンプルコード</summary><div>

```
descrive('My First Test', function() {
    it('Gets, types and asserts', function() {
        cy.visit('https://example.cypress.io')

        cy.contains('type').click()

        // 新しいURLが'/commands/actions'を含んでいること
        cy.url().should('include', '/commands/actions')

        // 入力要素を取得し、それに値を入力し、値が更新されることを検証する
        cy.get('.action-email')
          .type('fake@email.com')
          .should('have.value', 'fake@email.com')
    })
}) 
```
</div></details>  

#### ページ遷移

サンプルコードでは2つの方法で2つのページへ遷移した。

1. `cy.visit()`を使って最初のページへアクセスした。
2. `.click()`を使ってリンクからページ遷移した。

Cypressは`ページ遷移イベント`のようなものを自動的に検出して、次のページ読み込みが完了するまで自動的にコマンドの実行を停止する。

CypressはDOMの要素を検索するまでのタイムアウトが4秒だが、`ページ遷移イベント`が検出されると`PAGE LOAD`イベントのためにタイムアウトが60秒に増える。

様々なタイムアウトが[設定](https://docs.cypress.io/guides/references/configuration.html#Timeouts) のドキュメントに定義されている。

## デバッグ

Cypressはテストの理解を助けるデバッグツールのホストになる。

Cypressでできること
- それぞれのコマンドのスナップショットへ戻れる。
- 発生した`ページイベント`を確認できる。
- それぞれのコマンドに関する追加の出力を受け取れる。
- 複数のコマンド実行のスナップショットを行ったり来たりできる。
- コマンドを一時停止したりスキップできる。
- 非表示要素や複数要素が見つかった時に見える化できる。

### スナップショット

- 時系列上のコマンドをクリックすると、その行がピン留めされて背景色が紫に変わる。

上記のような状態になることの利点

#### 1. ピン留めされたスナップショット

- そのスナップショットが固定され、タイムシフト(原語だと`Time travel`)が機能しなくなり、そのタイミングのDOMを精査できる。

#### 2. イベント発生箇所

- `.click()`はアクションを起こすコマンドなので、イベントが発生した箇所を赤い丸で確認できる。

#### 3. スナップショットのメニューパネル

- コマンドによっては実行前後のスナップショットを確認できる。`before`と`after`を切り替えて、それぞれの状態を確認できる。
- サンプルコードの`CLICK`コマンドはリンクのクリックによりページ遷移するので、`after`のスナップショットはページ読み込みの速度によって、遷移前のままか、遷移直後の空白のページかが実行の度に異なる可能性がある。

### コンソール出力

- ディベロッパー ツールのConsoleタブにログが出力される。
- ディベロッパー ツールのElementsタブで要素を精査することもできる。

#### Cypressがコンソールに出力する情報(`GET`コマンドの場合)

- Command(`get`)
- Yielded(コマンド実行による結果)
- Elements(見つかった要素の数)
- Selector(要素の指定に使ったCSSクラス)

### 特殊コマンド

例
- `cy.pause()`
    - 「▶」(Resume)をクリックするまで処理を停止する。
- `cy.debug()`
    - テストをデバッグモードで実行する。
