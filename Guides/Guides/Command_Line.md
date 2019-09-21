# インストール

[Cypressをインストール](https://docs.cypress.io/guides/getting-started/installing-cypress.html#System-requirements)していることが前提。**プロジェクトルート**からこのドキュメントのすべてのコマンドを実行できる。

# コマンドの実行方法

---

実行する代わりに[モジュールAPI](https://docs.cypress.io/guides/guides/module-api.html#cypress-run)を使ってノードモジュールとしてCypressを実行することもできる。

---

それぞれのドキュメントでは、cypressを実行するパスが省略されているので、実際に実行する際には以下のようにプリフィクスが必要。

```shell
$ $(npm bin)/cypress run
```

```shell
$ ./node_modules/.bin/cypress run
```

```shell
$ npx cypress run
```

`package.json`の`script`オブジェクトにCypressのコマンドを追加し、[`npm run`](https://docs.npmjs.com/cli/run-script.html)[スクリプト](https://docs.npmjs.com/cli/run-script.html)から実行することもできる。

```JavaScript
{
  "scripts": {
    "cy:run": "cypress run"
  }
}
```

単一のスペックファイルのテストを実行して、ダッシュボードに結果を記録するには、次のように実行する。

```shell
$ npm run cy:run -- --record --spec "cypress/integration/my-spec.js"
```

[npx](https://github.com/zkat/npx)ツールを使っている場合は、ローカルにインストールしたCypressのツールを直接呼び出せる。

```shell
$ npx cypress run --record --spec "cypress/integration/my-spec.js"
```

# コマンド

## `cypress run`

Cypressのテストを実行する。デフォルトでは、`Electron`ブラウザでヘッドレスにすべてのテストを実行する。

```shell
$ cypress run [options]
```

### オプション

| オプション | 説明 |
| --- | --- |
| `--browser`, `-b` | テストを実行するブラウザを指定する |
| `--ci-build-id` | グループ化や並列化を可能にするために実行単位に識別子を指定する |
| `--config`, `-c` | 設定を指定する |
| `--env`, `-e` | 環境変数を指定する |
| `--group` | 記録された複数のテストを単一の実行単位にグループ化する |
| `--headed` | ヘッドレスに実行する代わりにElectronブラウザーを表示する |
| `--help`, `-h` | 使用方法を表示する |
| `--key`, `-k` | 秘密のレコード キーを指定する |
| `--no-exit` | 1つのスペック ファイルの実行が終わった後もテスト ランナーを開いたままにする |
| `--parallel` | 記録されたスペックを複数のマシンにまたがって並列に実行する |
| `--port`, `-p` | デフォルト ポートをオーバーライドする |
| `--project`, `-P` | 特定のプロジェクトへのパスを渡す |
| `--record` | テストの実行をどこに記録するか指定する |
| `--reporter`, `-r` | Mochaレポーターを指定する |
| `--reporter-options`, `-o` | Mochaレポーターのオプションを指定する |
| `--spec`, `-s` | 実行するスペック ファイルを指定する |

# コマンドのデバッグ

Cypressは[debug](https://github.com/visionmedia/debug)を使って構築されている。
`cypress open`や`cypress run`の前にこのモジュールをONにして実行することで、有益なデバッグ出力を受け取れる。

### MacやLinux:

```shell
$ DEBUG=cypress:* cypress open
```

### Windows

```shell
$ set DEBUG=cypress:*
```

```shell
$ cypress open
```

Cypressは多くのサブ モジュールを含む巨大で複雑なプロジェクトなので、デフォルトの出力だとその量に圧倒されることになってしまう。

### デバッグ出力を特定のモジュールにフィルターするには

```shell
$ DEBUG=cypress:cli cypress run
```

```shell
$ DEBUG=cypress:launcher cypress run
```

あるいは3段階の階層のサブモジュールでもフィルターできる

```shell
$ DEBUG=cypress:server:project cypress run
```
