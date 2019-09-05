# `debugger`を使う

## いつものようにデバッグする

次のコードは意図したように機能しない。

```javascript
it('鬼のようにデバッグさせろ', function() {
  cy.visit('/my/page/path')

  cy.get('.selector-in-question')

  debugger // 機能しない
})
```

上の`debugger`は`cy.visit()`および`cy.get()`の前に実行される。
（`cy`コマンドはキューに格納され、後から実行されるため。）

Cypressのコマンド実行中にデバッグする時は`.then()`を使う。

```javascript
it('コマンド実行後にデッバッグさせろ', function() {
  cy.visit('/my/page/path')

  cy.get('.selector-in-question')
    .then(($selectedElement) => {
      // デバッガーは cy.visit と cy.get コマンドが完了した後に機能する
      debugger
    })
})
```

## `.debug()`を使う

Cypressはコマンドをデバッグするためのショートカットである`.debug()`も提供している。前述のテストをこのヘルパー メソッドを使って書き換えると以下のようになる。

```javascript
it('鬼のようにデバッグさせろ', function() {
  cy.visit('/my/page/path')

  cy.get('.selector-in-question')
    .debug()
})
```
