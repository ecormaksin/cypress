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

