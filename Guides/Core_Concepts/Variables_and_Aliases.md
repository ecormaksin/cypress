# 戻り値

Cypressの新規ユーザーは、非同期APIに最初苦労するかもしれない。

JavaScriptでも非同期APIはある。現代のコードでも非同期処理はあらゆる所で見られる。ほとんどの新しいブラウザーAPIは非同期で、多くの主要なNodeモジュールも非同期である。

以下で探求するパターンはCypress以外でも有用である。

**Cypressのコマンドでは戻り値の割り当てや操作はできない。** コマンドはキューに格納され、非同期に実行される。

```javascript
// 以下は機能しない

// ダメ
const button = cy.get('button')

// ダメ
const form = cy.get('form')

// ダメ
button.click()
```

## クロージャー

Cypressの各コマンドが生成するオブジェクトへアクセスするためには`.then()`を使う。

```javascript
cy.get('button').then(($btn) => {
  // $btn は前のコマンドが生成したオブジェクト
})
```

[素のJavaScriptのプロミス](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)に馴染みがあるのであれば、Cypressの`.then()`は同じように機能する。`.then()`の内側でコマンドをさらにネストできる。

```javascript
cy.get('button').then(($btn) => {

  // ボタンのテキストを保存する
  const txt = $btn.text()

  // フォームをサブミットする
  cy.get('form').submit()

  // 2つのボタンのテキストを比較し
  // 違うことを確認する
  cy.get('button').should(($btn2) => {
    expect($btn2.text()).not.to.eq(txt)
  })
})

// 以下のコマンドは上記のすべてのコマンドが終わった後に実行される
cy.get(...).find(...).should(...)
```

---

コールバック関数を使うことで[クロージャー](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)を生成している。クロージャーは前に実行されたコマンドによって生成されたオブジェクトの参照を保持する。

---

## デバッグ

`.then()`を使うことは、`debugger`を使うのにうってつけ。コマンドが実行される順序を理解するのに役立つ。それぞれのコマンドが生成したオブジェクトを精査することもできる。

```javascript
cy.get('button').then(($btn) => {
  // $btn オブジェクトを精査する
  debugger

  cy.get('#countries').select('USA').then(($select) => {
    // $select オブジェクトを精査する
    debugger

    cy.url().should((url) => {
      // url 文字列を精査する
      debugger

      $btn    // はまだ使用可能
      $select // もまだ使用可能
    })
  })
})
```

## 変数

Cypressでは`const`, `let`, `var`を使う必要すらない。クロージャーを使っている時は、コマンドにより生成されたオブジェクトへ、変数に割り当てることなくアクセスできる。

例外は、変更可能なオブジェクトを扱う時。オブジェクトの変更前後の値を比較したい時。

このような時は`const`を使う。（割り当て以降は変更できないので、値の比較に適している。）

# エイリアス

`.then()`は前のコマンドにより生成されたオブジェクトへアクセスする素晴らしい方法だが、`before`や`beforeEach`のようなフックの中で実行したコマンドにより生成されたオブジェクトへ参照できない。


```javascript
beforeEach(function() {
  cy.button().then(($btn) => {
    const text = $btn.text()
  })
})

it('textへアクセスできない', function() {
  // どうやって text へアクセスする？
})
```

エイリアスを使う。  
エイリアスは様々な使い方がある強力な機能。  

## コンテキストを共有する

コンテキストの共有はエイリアスを使用する最もシンプルな方法。

共有したいものにエイリアスをつけるためには`.as()`コマンドを使う。

```javascript
beforeEach(function() {
  // $btn.text()に`text`というエイリアスを付与する
  cy.get('button').invoke('text').as('text')
})

it('textへアクセスできる', function() {
  this.text // はアクセス可能
})
```

内部では、基本的なオブジェクトやプリミティブ型にエイリアスを付与する際にMochaで共有されている[`context`](https://github.com/mochajs/mocha/wiki/Shared-Behaviours)オブジェクトを利用している。エイリアスは`this.*`のような形で使用可能となる。

### フィクスチャーへのアクセス

コンテキストの共有で最もよく使われるのは、`cy.fixture()`を使う時。

`beforeEach`フックでフィクスチャーを読み込むけれども、テストでその値を使いたいということがよくある。

```javascript
beforeEach(function() {
  // usersフィクスチャにエイリアスを付与する
  cy.fixture('users.json').as('users')
})

it('何かしらusersを使う', function() {
  // usersのプロパティにアクセスする
  const user = this.users[0]

  // ヘッダーに1番目のユーザーの名前が含まれていることを確認する
  cy.get('header').should('contain', user.name)
})
```

**Cypressのコマンドは非同期なので、`this.*`は`.as()`コマンドが実行されるまで使えない。**

```javascript
it('エイリアスを正しく使っていない', function() {
  cy.fixture('users.json').as('users')

  // これは動作しない
  //
  // this.users は定義されていない
  // 'as'コマンドはキューに格納されただけで、まだ実行されていないため
  const user = this.users[0]
})
```

コマンドで生成されたオブジェクトにアクセスしたい場合は、`.then()`を使ったクロージャーの中で処理する必要がある。

```javascript
// すべてうまくいく
cy.fixture('users.json').then((users) => {
  // エイリアスを避けてコールバック関数を使える
  const user = users[0]

  // 通る
  cy.get('header').should('contain', user.name)
})
```
### `this`の使用を避ける

---

テストやフックで[アロー関数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)を使っている場合、`this.*`のエイリアスでプロパティへアクセスできない。

`function () {}`であれば動くが、`() => {}`では動かない。

---

`this.*`を使う代わりに、エイリアスへアクセスする他の方法がある。

`cy.get()`コマンドは`@`を使った特殊な構文でエイリアスにアクセスできる。

```javascript
beforeEach(function () {
  // usersフィクスチャにエイリアスを付与する
  cy.fixture('users.json').as('users')
})

it('何かしらusersを使う', function () {
  // エイリアスへアクセスするために特殊な'@'構文を使う
  // そのことにより'this'を使わなくて済む
  cy.get('@users').then((users) => {
    // 引数のusersへアクセスする
    const user = users[0]

    // ヘッダーに1番目のユーザーの名前が含まれていることを確認する
    cy.get('header').should('contain', user.name)
  })
})
```

`this.users`を使う時は同期的にアクセスする。`cy.get('@users')`を使う時は非同期にアクセスする。

`cy.get('@users')`は[`cy.wrap(this.users)`](https://docs.cypress.io/api/commands/wrap.html)と同じと考えることができる。

## 要素

DOMの要素と一緒に使用される時、エイリアスは他の特殊な特徴を持つ。

DOM要素にエイリアスを付与した後、後でアクセスする時に再利用できる。

```javascript
// テーブル内で見つかったすべてのtrに'rows'というエイリアスを付与する
cy.get('table').find('tr').as('rows')
```

内部的に、Cypressは<tr>の集合に対して"rows"というエイリアスとして返される参照を生成している。同じ"rows"に後で参照するためには、`cy.get()`コマンドを使う。

```javascript
// Cypressは<tr>の集合への参照を返却する
// そしてコマンドをつなげて1番目の行を探すことができる
cy.get('@rows').first().click()
```

