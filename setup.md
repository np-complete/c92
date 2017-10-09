## 開発の準備

マストドンは基本的に普通のRailsと同じように開発できると考えてよいです。
開発には(むしろ本番も) **Ubuntu** を使うことをお勧めします。

Githubからコードをチェックアウトします。

```
$ git clone https://github.com/tootsuite/mastodon
$ cd mastodon
$ git checkout -b v2.0.0 v2.0.0
```

現在のは最新バージョンである `v2.0.0` を基準に解説します。
過去のバージョンを見るには `git checkout -b v1.6.1 v1.6.1` でチェックアウトできます。

マストドン開発には `ruby`, `node`, `yarn` のソフトウェアが必要です。
それぞれ好きな方法でインストールしてください。

### ライブラリのインストール

`Aptfile` に書かれているライブラリをインストールします。

```
$ sudo apt-get install `cat Aptfile`
```

でさくっとインストールできるはずです。
Debian系OS以外の場合は必要なライブラリをがんばって入れましょう[^1]。

ライブラリがインストールできたら

```
$ bundle install
$ yarn install
```

でgemとnpmをインストールします。

[^1]: ライブラリをがんばって入れるよりUbuntuをインストールするほうが簡単です

### ミドルウェアを立ち上げる

PostgreSQLとRedisはdockerで雑に立ち上げましょう。

```
$ docker run -d -p 5432:5432 --rm --name postgres postgres:alpine
$ docker run -d -p 6379:6379 --rm --name redis redis:latest
```

立ち上げたコンテナに繋がるように `.env` ファイルを作ります。

```
DB_HOST=localhost
DB_USER=postgres
```

この状態で

```
$ rake db:create
$ rake db:schema:load
```

を叩いてみましょう。
データベースが作られると思います。

`.env` ファイルは環境変数を追加できます。
開発用には `.env` を、本番用には `.env.production` 使います。
どちらもコミットしてはいけません[^2]。

[^2]: 本番には何らかの方法で `.env.production` を配置する必要があるということです。

### アプリケーションを立ち上げる

あとはアプリケーションを立ち上げたら開発が始まります。
とは言えRails、stream、sidekiqを個別に立ち上げるのは面倒ですので、**foreman** を使います。

```
$ gem install foreman
$ foreman start
```

`Procfile.dev` の内容に従って、必要なプロセスが起動します。
しばらくすると `webpack` の処理が終わり、 http://localhost:3000 にマストドンが立ち上がります。
これで開発の準備は完了です。
