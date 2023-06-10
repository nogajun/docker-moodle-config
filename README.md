# dockerを使ってmoodleをセットアップする

[bitnami](https://bitnami.com/)が作っているmoodleのdockerコンテナを使って、moodleをセットアップしたのでメモです。

## 基本方針

* Docker(docker compose)を使う
    * dockerは、すでにインストールされている前提
* データはホストのディレクトリに永続化する
    * ボリュームコンテナは使わない

## 設定

### docker-compose.ymlの設定

moodleは、docker composeを使って起動するので、`docker-compose.yml`を作成します。[bitmaniのリポジトリ](https://github.com/bitnami/containers/)に[docker-compose.ymlのサンプルがある](https://github.com/bitnami/containers/blob/main/bitnami/moodle/docker-compose.yml)ので、これをベースに編集しました。

主な変更点は以下の通り。データベースのユーザー名やデータベース名、パスワードは、`docker-compose.yml`に直書きしていますが、moodle,mariadb両方に記述するので`.env`ファイルに書いて読み込むようにしたほうが間違いがなくていいと思います。

* mariadb
    * rootパスワードを設定
    * ユーザーbn_moodleのパスワードを設定
    * 空のパスワードを無効
    * collateをutf8mb4_general_ciに変更
    * dbを保存するボリュームに実ディレクトリを指定
* moodle
    * mariadbに合わせてユーザーbn_moodleのパスワードを指定
    * 空のパスワードを無効
    * moodleのホスト名、サイト名、言語を指定
    * dbを保存するボリュームに実ディレクトリを指定

`environment`の変数については次のリンクを参照。

* containers/README.md at main · bitnami/containers · GitHub: https://github.com/bitnami/containers/blob/main/bitnami/mariadb/README.md#configuration
* containers/README.md at main · bitnami/containers · GitHub: https://github.com/bitnami/containers/blob/main/bitnami/moodle/README.md#configuration

### データを保存するディレクトリを作成

`docker-compose.yml`の設定に合わせてデータを保存するディレクトリを`docker-compose.yml`直下に作成します。

`mariadb`と`moodle`でディレクトリの所有者が違うので気をつけてください。あと、パーミッションも忘れないように設定します。

```console
$ mkdir mariadb moodle moodledata
$ sudo chown 1001:root mariadb/
$ sudo chown daemon:root moodle/ moodledata/
$ sudo chmod 775 mariadb/ moodle/ moodledata/
```

## moodleを起動する

起動については、ほかのdocker composeと同じで、`docker compose up -d`で起動します。

## moodleにログインする

起動したあと、何でログインできるか探しまくる人がいると思うので初期ユーザー名とパスワードを書いておきます。

* ユーザー名: `user`
* パスワード: `bitnami`

上のデフォルト設定以外にしたい場合は、bitnamiドキュメントの[サイト設定](https://github.com/bitnami/containers/blob/main/bitnami/moodle/README.md#user-and-site-configuration)にも書いていますが、`MOODLE_USERNAME`と`MOODLE_PASSWORD`をそれぞれ設定してください。

## LICENSE

* [Creative Commons CC0](https://creativecommons.org/publicdomain/zero/1.0/deed.ja)
