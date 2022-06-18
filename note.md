# github action

## ワークフロー
* リポジトリ内のソフトウェア開発におけるプロセスを自動化したもの
* ソフトウェアのビルド、テスト、パッケージ作成、デプロイ自動化等のCI/CD
* ジョブAが成功したらジョブBを実行するというように依存関係を持たせることもできる


## アクション
* Github Actionが提供する何かしらの処理の塊を表す単位
* 自作することも可能

## ワークフローファイルの保存場所
* .github/workflowsディレクトリ配下
* 拡張子は .yml か .yaml
* 異なるジョブ間では別のVMが使われるため、ジョブ間で情報をやり取りするためにはアーティファクトなどの方法が必要
* githubが提供する環境のOSはLinux, windows, macOSの三種類
* Linuxではパスワードなしでsudoコマンドを実行できる


## ワークフロー構文

### name
* ワークフローの名前を表す。ブラウザ上でリポジトリのactionタブを開いた時に表示される

### on 
* ワークフローを実行するイベントを表す。
* 指定必須
* 例: push時に実行する場合
  * on: push
* 例: push, プルリクエスト時に実行する場合
  * on: [push, pull_request]

### jobs
* ジョブの定義

### jobs.<jon_id>
* ジョブのIDを表す。値を自由だが重複してはいけない
* アルファベットか_で始まる文字列で使える文字列は[アルファベット、数字、-、_ ]のみ

### jobs.<jon_id>.name
* ジョブの名前を定義する。ブラウザー上でactionタブを開いた時に表示される
* 省略するとジョブIDが表示される。

### jobs.<jon_id>.runs-on
* ジョブが実行されるマシンを定義する。(必須)
* マシンの種類
  * ubuntu-latest
    * ubuntuの最新バージョン
  * ubuntu-18.04
  * windows-latest
    * windows server 2019
  * macos-latest
    * macos catalina 10.15

### jobs.<job_id>.steps
* ジョブを実行するステップの定義
* それぞれのステップはそれぞれ異なるプロセスで実行される。そのためステップ内で環境変数を変更しても後のステップには変更が反映されない
* 後のステップに環境変数の変更を反映さえる方法はset-envを使用する

### jobs.<job_id>.steps.name
* ステップの名前を定義

### jobs.<job_id>.steps.uses
* アクションの実行を定義する
* 「アクション@バージョン」の形で定義する
* バージョン指定
    * uses: actions/checkout@v2.0.0
* メジャーバージョン指定
    * uses: actions/checkout@v2
* コミットSHA指定
    * uses: actions/checkout@722adc63f1aa60a57ec378
* おすすめの方法はSHAで記述してコメントでバージョン情報を記述する
# v2.0.0
- uses: actions/checkout@722adc63f1aa60a57ec378

* docker hub上のイメージをアクションとして実行
  * uses: docker://alpine:3.11

### jobs.<job_id>.steps.with
* アクションのパラメータを定義する
* 渡せるパラメータはアクション側にメタデータとして定義されている

### jobs.<job_id>.steps.with．entrypoint
* Dockerコンテナアクションを実行する場合 entrypointとargsという2つのパラメータを渡すことができる
* entrypointはDockerfileのENTRYPOINTを上書きする

### jobs.<job_id>.steps.with．args
* コンテナアクション実行時ENTRYPOINTに渡される引数を文字列で定義する

### jobs.<job_id>.steps．run
* コマンドを実行

### jobs.<job_id>.steps.working-directory
* run実行時にworking-directoryを指定することでコマンド実行のワーキングディレクトリを変更できる
* ワーキングディレクトリを /tmpに変更してコマンドを実行する例
* - run: |
      pwd
      ls
    working-directory: /tmp   

### jobs.<job_id>.steps.shell
* run実行時にシェルを指定することでコマンド実行時に使用されるshellを変更できる
* 種類
  * shell
  * bash
  * pwsh
  * python
  * sh
  * cmd
  + powershell
* pythonスクリプトを実行
* - run: |
      import sys
      print(sys.version)
    shell: python

### jobs.<job_id>.countinue-on-error
* ジョブのcontinue-on-errorをtrueに指定することでそのジョブが失敗してもワークフローが失敗にならなくなる

### jobs.<job_id>.steps.id
* ステップにIDを設定することで ${{ steps.<step_id>.outputs.<output_name> }}のようにコンテキスト経由でステップのアウトプットにアクセスできる

### defaults
* ワークフローのトップレベルでdefaultsを指定すると、ワークフロー内の全てのジョブにデフォルトで適用される設定を定義できる

### defaults.run
* ワークフロー内の全てのrunステップにデフォルトで適応される設定を定義できる
* shellとworking-directoryを指定することができる

### env
* ワークフローのトップレベルでenvを指定するとワークフロー内すべてのジョブ、ステップに適応される環境変数を定義できる

### jobs.<job_id>.env
* 特定のjobだけで適用される環境変数を定義

### デフォルトの環境変数
* CI
  * 常にtrue
* HOME
  * ホームディレクトリのパス
* GITHUB_WORKFLOW
  * ワークフロー名
* GITHUB_RUN_ID
  * ワークフローの実行ごとに設定されるリポジトリー全体でユニークなID (ワークフローを再実行しても変更されない)
* GITHUB_RUN_NUMBER
  * ワークフローの実行ごとに設定されるそれぞれのワークフローごとにユニークな1から始める番号 (ワークフローを再実行しても変更されない)
* GITHUB_ACTION
  * 現在実行中のアクションのユニークなID (同じアクションでも毎回異なる値になる)
* GITHUB_ACTOR
   * ワークフローを実行した人又はアプリの名前
* GITHUB_REPOSITORY
   * <owner>/<repository>の形式
* GITHUB_EVENT_NAME
    * ワークフローを実行したイベント名
* GITHUB_EVENT_PATH
    * ワークフローを実行したイベントのペイロードが保存されたファイルパス

### 秘密情報の扱い
* secretsに登録
* github actions上で以下の方法でアクセス可能
    * ${{ secrets.SECRET_TOKEN_NAME }}

### jobs.<job_id>.needs
* そのジョブの実行前に成功していないとけないジョブを定義できる
```
  practice: 
    name: practice
    runs-on: ubuntu-latest
    needs: [unit-test, lint]
```