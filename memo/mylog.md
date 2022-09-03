# GitHub Pagesにpushするファイル一式の作成

　HTTPSにアップロードすることで動作するようにした。詳細は以下ソースコード参照。

* src/js/app/github/site-maker.js

　ざっくりいうと以下の通り。

* mpurse APIを使い投げモナボタンを実装する
* sql.js（WASM版）でSQLファイルmylog.dbの内容を読み取る
* 必要な各ファイルをコピーなり一部書き換えなりパス変更なりをする
* つぶやきの日付表示バグを修正した

　Node.jsでなく、ブラウザ上で動作するコードにする必要があった。基本的には以下をベースにしている。

* [つぶやきを保存するツール6][]

[つぶやきを保存するツール6]:https://monaledge.com/article/482






# git pushのアップロードに失敗してる謎

　Electronでshellコマンドを実行しgit pushしたが、なぜか一部ファイルのアップロードがされなかった。ローカルファイルのコピーは成功しているが、GitHubのソースコード側では一部アップされてない。原因不明。

<!-- more -->

# 症状

　投げモナボタンの画像ファイルがアップロードされていなかった。原因不明。そのせいで投げモナボタンが表示されない。

# ブツ

* [リポジトリ][]

[リポジトリ]:https://github.com/ytyaru/Electron.MyLog.20220831120615

## インストール＆実行

```sh
NAME='Electron.MyLog.20220831120615'
git clone https://github.com/ytyaru/$NAME
cd $NAME
npm install
npm start
```

### 準備

1. [GitHubアカウントを作成する](https://github.com/join)
1. `repo`スコープ権限をもった[アクセストークンを作成する](https://github.com/settings/tokens)
1. `npm start`でアプリ起動し終了する（`db/setting.json`ファイルが自動作成される）
1. `db/setting.json`に以下をセットしファイル保存する
	1. `address`：任意のモナコイン用アドレス
	1. `username`：任意のGitHubユーザ名
	1. `email`：任意のGitHubメールアドレス
	1. `token`：`repo`スコープ権限を持ったトークン
	1. `repo`：任意リポジトリ名（`mytestrepo`等）
1. `dst/mytestrepo/.git`が存在しないことを確認する（あれば`dst`ごと削除する）
1. GitHub上に同名リモートリポジトリが存在しないことを確認する（あれば削除する）

### 実行

1. `npm start`で起動またはアプリで<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>R</kbd>キーを押す（リロードする）
1. `git init`コマンドが実行される
	* `repo/リポジトリ名`ディレクトリが作成され、その配下に`.git`ディレクトリが作成される
1. [createRepo][]実行後、リモートリポジトリが作成される

### GitHub Pages デプロイ

　アップロードされたファイルからサイトを作成する。

1. アップロードしたユーザのリポジトリページにアクセスする（`https://github.com/ユーザ名/リポジトリ名`）
1. 設定ページにアクセスする（`https://github.com/ユーザ名/リポジトリ名/settings`）
1. `Pages`ページにアクセスする（`https://github.com/ユーザ名/リポジトリ名/settings/pages`）
    1. `Source`のコンボボックスが`Deploy from a branch`になっていることを確認する
    1. `Branch`を`master`にし、ディレクトリを`/(root)`にし、<kbd>Save</kbd>ボタンを押す
    1. <kbd>F5</kbd>キーでリロードし、そのページに`https://ytyaru.github.io/リポジトリ名/`のリンクが表示されるまでくりかえす（***数分かかる***）
    1. `https://ytyaru.github.io/リポジトリ名/`のリンクを参照する（デプロイ完了してないと404エラー）

### 結果

* [自動作成されたリポジトリを手動でデプロイしたサイト][]
* [自動作成されたリポジトリ][]

[自動作成されたリポジトリ]:https://github.com/ytyaru/Electron.MyLog.Upload.Test.20220831120615
[自動作成されたリポジトリを手動でデプロイしたサイト]:https://ytyaru.github.io/Electron.MyLog.Upload.Test.20220831120615/

　投げモナボタンが表示されなかった。ボタンの画像ファイルがアップロードされなかったせい。`asset/`ディレクトリまるごと。なぜアップされなかったかは不明。ちゃんと`site-maker.js`で以下のようなコードを書いてあるのに。

```javascript
this.#cp(`asset/`),
```

　ローカルリポジトリ内にはファイルコピーされていた。なのにGitHubにアップロードされてない。

　非同期がうまくいってない感じかな？　でもちゃんと`main.js`で`await`しているし。謎。

```javascript
await maker.make()
await git.push()
```

