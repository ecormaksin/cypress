# 自分たちのアプリをテストする

---
- 学習すること
    - Cypressと自分たちで開発するバックエンドとの関係
    - Cypressを自分たちのアプリに合わせて設定する方法
    - 認証の仕組みを使って（あるいは使わずに）処理する
    - テストデータを効率的に準備する
---

## 1. Webサーバーを開始する

- アプリケーションを動かすローカルの開発サーバーを開始する。

---
- アンチパターン
    - Cypressのスクリプト内でWebサーバーを起動してはいけない。ここの[ベストプラクティス](https://docs.cypress.io/guides/references/best-practices.html#Web-Servers) について読むこと。
---

---
- なぜローカルの開発サーバーを開始するのか？  
    - すでにデプロイされたプロダクションをテストすることもできるが、Cypressが主に対象としているところではない。  
    - Cypressは日々のローカルでの開発のために作られ、最適化されている。  
    - テストと開発を同時に行えるようになるだけでなく、より速くアプリをビルドできる。
    - **スタブ ネットワーク リクエスト**のようなことができるので、サーバーなしで開発できる。
    - すでに開発されたアプリケーションへ何とかしてテストを適用しようとすることは、テストを書く時に開発するよりも難しい。当初からテストを書くことを避けると、その分後で大変な目に遭う。
    - 最も重要な理由はおそらく制御可能であること。プロダクション環境で実行されているアプリケーションは制御できない。  
      開発環境で実行している場合、次のことが簡単にできる:  
        - ショートカットできる。
        - 実行可能なスクリプトを走らせることで、データを埋め込む。
        - テスト環境の特定のルートを明らかにできる。
        - 自動化を難しくするセキュリティ機能を無効にできる。
        - サーバーやデータベースの状態をリセットできる。
    - とはいうものの、プロダクション環境でテストすることもできる。  
      ユーザーの多くは統合テストの大部分をローカルサーバーで実行し、残りはデプロイしたプロダクション環境でスモークテストの一部のみを実行している。
---

## 2. サーバーへアクセスする

`sample_spec.js`を削除して、`home_page_spec.js`のようなファイルを作成する。（新しくプロジェクトを作成してもよいのでは？）  
以下のようなコードを書く。

```
describe('The Home Page', function() {
    it('successfully loads', function() {
        cy.visit('http://localhost:8080') // 開発対象のULRに合わせて変える
    })
})
```

## 3. Cypressを設定する

Webアプリケーションのトップページはアクセスする回数が多く、毎回そのURLを指定するのは手間なので、Cypressを設定することでベースのURLは指定が不要になる。

`cypress.json`へ以下のように設定する。

```
{
    "baseUrl": "http://localhost:8080"
}
```

上記のように設定することで、`cy.visit()`や`cy.request()`を実行するとベースURLが指定された状態になる。

---

`cypress.json`を編集すると、Cypressは自動的に再起動して開いているブラウザーを閉じる。

---

以下のように相対パスでアクセスできるようになる。

```
describe('The Home Page', function() {
    it('successfully loads', function() {
        cy.visit('/')
    })
})
```

---

- 設定オプション
    - Cypressはテストの実行場所、デフォルトのタイムアウト期間、環境変数、報告者のアカウントなどのような項目をカスタマイズできる。  
      [設定ページ](https://docs.cypress.io/guides/references/configuration.html) を参照のこと。

---

## テスト戦略

- **何をテストするべきか、境界や層がどこにあるか、リグレッションテストの範囲をどこにするかは各アプリの開発チームが考えなければならない。**

- 上記の通りではあるものの、現代のWebテスティングにはすべての開発チームが経験することがある。なので、そのような共通の状況における簡単なコツを解説する。

### データの準備

- 構築しているアプリケーションの構造による
    - フロントエンド
        - 現代的にjson経由なのか
        - 従来のようにサーバーでHTMLをレンダリングしているのか

- 従来のようにSeleniumを使ってE2Eテストを書く場合は、ブラウザーを自動化する前にサーバー上で`set up`(テストの前準備)や`tear down`(テストの後処理)を行う。

- データの準備には多くの戦略があるが、Cypressで行う場合は一般的に次の3つの方法がある。
    - `cy.exec()` - システムのコマンドを実行する
    - `cy.task()` - [pluginsFile](https://docs.cypress.io/guides/references/configuration.html#Folders-Files)を使ってNode上でコードを実行する
    - `cy.request()` - HTTPリクエストを行う

- サーバーで`node.js`を実行している場合、`npm`のタスクを実行する`before`や`beforeEach`のフック処理を追加できる。

```
describe('The Home Page', function() {
    beforeEach(function () {
        // 各テストの前にデータベースをリセットしてデータを準備する
        cy.exec('npm run db:reset && npm run db:seed')
    })

    it('successfully loads', function() {
        cy.visit('/')
    })
})
```

- いくつかのリクエストを構成して、サーバーに期待する状態を設定することもできる。

```
describe('The Home Page', function() {
    beforeEach(function() {
        // 各テストの前にデータベースをリセットしてデータを準備する
        cy.exec('npm run db:reset && npm run db:seed')

        // テストから制御する1件の投稿をDBに準備する
        cy.request('POST', '/test/seed/post', {
            title: 'First Post',
            authorId: 1,
            body: '...'
        })

        // テストから制御するユーザーをDBに準備する
        cy.request('POST', '/test/seed/user', { name: 'Jane' }).its('body').as('currentUser')
    })

    it('successfully loads', function() {
        // this.currentUser は cy.request() のレスポンス・ボディを参照しており
        // ログインやそのほかの操作で使うことができる

        cy.visit('/')
    })
})
```

- 上記のアプローチが悪いわけではないけれども、多くの複雑性をもたらしている。サーバーとブラウザーの状態を同期するのに苦労するだろう。テストの前に、毎回その状態をセットアップし破棄する必要がある。(そして、それは遅い。)

**Cypressでは、間違いなく従来のテスト方法より良くて速いアプローチがある。**

### サーバーをスタブする

---
#### メリット
- データベースをリセットしたり、データを準備したりする必要がない。
- テストコードから状態を変えなくてよい。
- 他のテストに影響を与えるような状態変化がない。
- サーバーがなくてもテストできる。

#### デメリット
- スタブが返すレスポンスが、実際にサーバーが返すものと同じという保証がない。
---

- よりバランスの取れたアプローチは、スタブを使わないE2Eテストを1つ書いて、残りはスタブを使うこと。

### ログイン

- アプリケーションのログインをテストするのは、最も難しい内容の1つ。

- 推奨方法は、UIを使って実際のユーザーが行うのと同様にログインすること（サーバーとDBも使う）。ただし、各テストの前にUIを使ってログインすべきではない。

#### UIを使ったログインテストの例

```
describe('ログインページ', function() {
    beforeEach(function() {
        // 各テストの前にデータベースをリセットしてデータを準備する
        cy.exec('npm run db:reset && npm run db:seed')

        // テストから制御するユーザーをDBに準備する
        // ランダムなパスワードが生成されるという前提
        cy.request('POST', '/test/seed/user', { username: 'jane.lane' })
          .its('body')
          .as('currentUser')
    })

    it('フォームのサブミットを介してログインした時に認証クッキーをセットする', function() {
        // this.currentUser の割当を解除する
        const { username, passowrd } = this.currentUser

        cy.visit('/login')

        cy.get('input[name=username]').type(username)

        // {enter}によりフォームがサブミットされる
        cy.get('input[name=password]').type(`${password}{enter}`)

        // /dashboard へリダイレクトする
        cy.url().should('include', '/dashboard')

        // 認証クッキーが存在すること
        cy.getCookie('your-session-cookie').should('exist')

        // ユーザーがログイン済みであることをUIが示していること
        cy.get('h1').should('contain', 'jane.lane')
    })
})
```

- UIを使って上記のように正常系のログインをテストすると、次のようなこともテストしたくなるかもしれない。
    - 不正なユーザー名 / パスワード
    - 取得されたユーザー名
    - パスワードの複雑さの要件
    - ロックされている / 削除されたアカウント などの境界テスト

#### UIをバイパスする

- 特定の機能をテストする時はUIを使ってテストすべき。他の機能をテストするために、そこに至る状態を作るためにはUIを使うべきではない。
    - 例
        - ショッピングカートの機能をテストしているとする。
        - 商品をカートに追加できる機能が必要。
        - UIを使って管理者機能にログインし、商品説明も含む商品、カテゴリー、画像すべてを登録し、それぞれの商品ページへアクセスしてショッピングカートへ1個1個追加するのは大変。

- CypressはSeleniumではないので、`cy.request()`を使ってUIの使用を大幅にショートカットできる。

##### `cy.request()`を使ったログインの例

```
describe('ログインページ', function() {
    beforeEach(function() {
        // 各テストの前にデータベースをリセットしてデータを準備する
        cy.exec('npm run db:reset && npm run db:seed')

        // テストから制御するユーザーをDBに準備する
        // ランダムなパスワードが生成されるという前提
        cy.request('POST', '/test/seed/user', { username: 'jane.lane' })
          .its('body')
          .as('currentUser')
    })

    it('フォームのサブミットを介してログインした時に認証クッキーをセットする', function() {
        // this.currentUser の割当を解除する
        const { username, passowrd } = this.currentUser

        // UIなしでログインする
        cy.request('POST', '/login', {
            username,
            password
        })

        // ログインしているので、制限つきのパスにもアクセスできる
        cy.vist('/dashboard')

        // 認証クッキーが存在すること
        cy.getCookie('your-session-cookie').should('exist')

        // ユーザーがログイン済みであることをUIが示していること
        cy.get('h1').should('contain', 'jane.lane')
    })
})
```

---
#### 認証のコツ

認証はこの例以外にも複雑な例がある。

CSRFトークンを扱ったり、XHR(XMLHttpRequest)に基づくログインフォームをテストしたりするコツが用意されている。

詳しくは[リファレンス](https://docs.cypress.io/examples/examples/recipes.html)を参照。

---