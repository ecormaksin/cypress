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

`cy.get()`によって生成されたオブジェクトは、ディベロッパー ツール上で変数`subject`としてアクセスできる。

## ディベロッパー ツールを使う

Cypressにはアプリケーションとテストで起こっていることの理解に役立つ優れたテスト ランナーがあるが、ブラウザーのディベロッパー ツールに取って代わることはできない。

コンソール ログには様々な情報が出力されているので、それを活用する。

# Cypressのトラブルシューティング

## 問題の切り分け

- 録画された動画やスクリーンショットを確認する。
- 大きなスペック ファイルを小さく分割する。
- 長いテストを小さく分割する。
- 同じテストを`--browser chrome`オプションで実行する。Electronブラウザ特有の問題に切り分けられるかもしれない。
- Electronブラウザの問題であると切り分けできたら、ElectronとChromeの両方で同じテストを実行し、スクリーンショットと動画を比較する。コマンド ログにおける違いを探して切り分ける。

### Chromeの特定のバージョンをダウンロードする

Chromeは自動更新されるので、そのことにより自動化テストが失敗することがある。`chromium.cypress.io`(https://chromium.cypress.io/)で特定のバージョンを提供している。

## Cypressのキャッシュを削除する

Cypressのインストール中に問題が発生したら、Cypressのキャッシュの中身を削除してみる。
マシンにキャッシュされているCypressのインストールされたすべてのバージョンがクリアされる。

```shell
$ cypress cache clear
```

上のコマンドを実行した後は、Cypressを再実行する前に`cypress install`を実行する必要がある。

```shell
$ npm install cypress --save-dev
```

## アプリ データのクリア

### アプリ データのクリア方法
1. Cypressを`cypress open`で開く
1. `File` -> `View App Data`を選択する
1. `cy`という名前のフォルダーが開かれる。
1. `cy`フォルダーの中のすべてを削除する。
1. Cypressを閉じて、もう1度開く。

## テストのコマンドを1つ1つ実行する

`.pause()`コマンドを使って、コマンドごとにテストを実行できる。

```javascript
it('項目を追加する', function() {
  cy.pause()
  cy.get('.new-todo')
  // 追加のコマンド
})
```

## デバッグ ログを出力する

Cypressは[debug](https://github.com/visionmedia/debug)を使って構築されている。
このモジュールをONにして実行することで、有益なデバッグ出力を受け取れる。
**注意: **`DEBUG=...`を指定して実行すると、メッセージが大量に出力される。

さらに学ぶには[コマンドライン](./Command_Line.md)と[よいロギング](https://glebbahmutov.com/blog/good-logging/)を参照。

### ブラウザ上のデバッグ ログ

`cypress open`の実行中に問題が見られるようなら、ブラウザーにデバッグ ログを出力できる。ブラウザーのディベロッパー ツールを開いて、`localStorage`プロパティをセットする。

```javascript
localStorage.debug = 'cypress*'

// デバッグ メッセージを無効にする
delete localStorage.debug
```

ブラウザーをリロードすると、ディベロッパー ツールのコンソールでデバッグ メッセージが見えるようになる。

## Cypressのイベントをログに記録する

