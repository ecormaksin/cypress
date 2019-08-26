# セットアップ

## 記録するプロジェクトをセットアップする

1. テストランナー内でプロジェクトのRunsタブをクリックする。
1. [Set up Project to Record]ボタンをクリックする。
1. テストを記録するためにはログインが必要なので、Cypressダッシュボードへログインする。
1. プロジェクト名を入力する（表示目的のみで後から変更できる）。
1. プロジェクトの所有者を選択する。個人、あるいは作成した組織を選択できる。まだ組織を作成していなければ、「Create organization」をクリックする。組織はGitHub上の組織と同じように機能する。個人と作業プロジェクトを分けることができる。詳細は[Organizations](https://docs.cypress.io/guides/dashboard/organizations.html)を参照。
1. プロジェクトがPublicかPrivateかを選択する。
    - Publicプロジェクトは誰でも記録や実行の様子を確認できる。典型的にそれらはオープンソースであることが多い。
    - Privateプロジェクトは招待したユーザーのみにアクセスを制限する。
1. [Setup Project]をクリックする。
1. テストの実行を初めて記録する時の方法が表示される。
1. プロジェクトのセットアップが完了した後、Cypressは`cypress.json`にユニークな[プロジェクトID](https://docs.cypress.io/guides/dashboard/projects.html#Identification)を挿入している。ソースコード管理システムを使っている場合は、プロジェクトIDを含む`cypress.json`をコミットすることが推奨される。
1. [CI](https://docs.cypress.io/guides/guides/continuous-integration.html)またはターミナルから実行する時は、`cypress run`で実行する時に、[レコードキー](https://docs.cypress.io/guides/dashboard/projects.html#Identification)を指定する。
  - 直接指定する場合:
    ```shell
    $ cypress run --record --key <record key>
    ```
  - あるいは環境変数としてレコードキーをセットする
    ```shell
    $ export CYPRESS_RECORD_KEY=<record key>
    ```
    ```shell
    $ cypress run --record 
    ```
  - **上記の`cypress run`コマンド実行時でハマったところ**
    - `cypress run`だけでは`command not found: cypress`でエラーになった。[Command Line](https://docs.cypress.io/guides/guides/command-line.html#How-to-run-commands)に記載のあるように、以下のいずれかで実行する必要があった。コマンドを実行する際のルートパスはプロジェクトのルートパス。
      - ```shell
        $(npm bin)/cypress run --record --key <record key>
        ```
        **※先頭の`$`はターミナルのプロンプトではなく、コマンドの一部**
      - ```shell
        ./node_modules/.bin/cypress run --key <record key>
        ```
      - ```shell
        npx cypress run --key <record key>
        ```

# プロジェクトの識別

## プロジェクトID

プロジェクトのセットアップにより`cypress.json`に自動的に追加される6桁の文字列

手動で変更すると、**Cypressがプロジェクトを識別できなくなり、またテストの記録も検索できなくなる。**

プロジェクトIDが追加されている`cypress.json`をバージョン管理システムにコミットするのが推奨だが、プロジェクトIDを公開したくない場合は環境変数に指定する。

```shell
export CYPRESS_PROJECT_ID={projectId}
```

## レコード キー

レコード キーは、ダッシュボード サービスにテストを記録することが許可されているプロジェクトであることを認証するために使われる。プロジェクトIDがあってもレコードキーが非公開だと、テスト実行を記録できない。

公開プロジェクトではレコード キーを秘密にしておかないと、レコード キーとプロジェクトIDを知っている第三者が実行したテスト記録が請求対象に含まれてしまう。　

もしレコード キーが漏えいした場合は、[削除](https://docs.cypress.io/guides/dashboard/projects.html#Delete-record-key)して[作り直す](https://docs.cypress.io/guides/dashboard/projects.html#Create-new-record-key)必要がある。

[ダッシュボード](https://on.cypress.io/dashboard)で複数のレコード キーを作成したり、既存のものを削除したりできる。

テスト ランナーのSettingsタブ内にもレコード キーが記載されている。

# 並列化の設定

## 実行完了の遅延

テストの実行が完了状態へ移行するまでに待機する秒数を編集できる。詳細は[並列化ガイド](https://docs.cypress.io/guides/guides/parallelization.html#Run-completion-delay)を参照。

## Githubとの統合

プロジェクトをGithubと統合でき、設定をプロジェクトの設定ページから編集できる。

詳細は[Github統合ガイド](https://docs.cypress.io/guides/dashboard/github-integration.html)を参照。

# 実行記録へのアクセス

プロジェクトの設定ページを開くと、誰に実行権限が付与されているか分かる。

## パブリック vs プライベート

- **パブリック**は、プロジェクトIDを知っている人なら誰でも、テストの実行記録を閲覧できることを意味する。
- **プライベート**は、[組織](https://docs.cypress.io/guides/dashboard/organizations.html)に招待された人だけがテストの実行記録を閲覧できることを意味する。たとえプロジェクトIDを知っていても、招待されていなければ実行記録を閲覧できない。

# 所有権の譲渡

## プロジェクトを他のユーザーや組織へ譲渡する

自分が所属している他の組織、または組織内のユーザーへプロジェクトを譲渡できる。プロジェクトはダッシュボード サービスからのみ譲渡できる。

## プロジェクト譲渡のキャンセル

移譲に際して、組織のプロジェクトへアクセスして[Cancel Transfer]ボタンをクリックすることで、いつでも譲渡をキャンセルできる。

## 移譲されたプロジェクトの受諾または拒否

プロジェクトが自分に譲渡された時は、通知メールが届く。自分の所属する組織のプロジェクト一覧へアクセスし、[Accept]または[Reject]をクリックすることで、受諾または拒否できる。

# プロジェクトの削除

所有しているプロジェクトを削除できる。プロジェクトを削除すると記録されたテストの実行結果もすべて削除される。プロジェクトはダッシュボード サービスからのみ削除できる。