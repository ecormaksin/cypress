# フォルダ構造

## フィクスチャ

- デフォルトの場所は`cypress/fixtures`
- 他のディレクトリへ設定で変更できる。

## テスト ファイル

- テスト ファイルは以下のものが認識される。
    - `.js`
    - `.jsx`
    - `.coffee`
    - `.cjsx`

- ES2015やCommonJSのモジュールを使える。

# テストの記述

- CypressはMochaやChaiの上に成り立っている。
- ChaiのBDDおよびTDDのアサーション スタイルをサポートしている。

## テストの構造

- Mochaから借りてきたテスト インターフェース
    - `describe()`
    - `context()`: `describe()`と同じ
    - `it()`
    - `specify()`: `it()`と同じ

## フック

- Mochaから借りてきたフックも提供している。
- 一連のテストの前後、あるいは各テストの前後に実行する処理を記述する。  

```
describe('フック', function() {
    before(function() {
        // ブロック内のすべてのテストの前に一度だけ実行する
    })

    after(function() {
        // ブロック内のすべてのテストの後に一度だけ実行する
    })

    beforeEach(function() {
        // ブロック内の各テストの前に実行する
    })

    afterEach(function() {
        // ブロック内の各テストの後に実行する
    })
})
```

- テストの実行順序
    - `before()`
    - `beforeEach()`
    - テスト コード
    - `afterEach()`
    - `after()`

---

`after()`や`afterEach()`のフックを書く前に、[`after()`や`afterEach()`で状態をクリアするというアンチ パターンに対する考察](https://docs.cypress.io/guides/references/best-practices.html#Using-after-or-afterEach-hooks)を参照のこと。

---

## テストの除外と対象化

- 特定のテストだけを実行する時は`.only`を付加する。`it`でネストされたブロックのテストのみが実行される。テスト スイートを書く時に推奨される方法。

- 特定のテストだけを除外する時は`.skip`を付加する。`it`でネストされたブロックのテストのみがスキップされる。

