### ブランチ戦略

ガチ運営勢になろうと思っている人のために、
実際のガチ運営勢がどのようなブランチ戦略をとっているか解説します。

前提として、企業に所属してマストドンインスタンスを運用すると想定します。
また同時に、マストドン本体への貢献とも切り替えやすいことも前提とします。

#### 方針

AGPLのため、コードの公開義務があります。**何も考えずに最初からGithubを使いましょう**。
会社のOrganizationを作り、そこで公開します。
もちろん、Githubを使うからには開発フローはプルリクエストベースです。

本家への追従は、 **リリースタグ** の単位で行います。
**リリースタグですらぶっ壊れてる**ことが多いので、
更にぶっ壊れてる可能性の高いmasterへの追従は**基本的にお勧めしません**。

メインブラインチは `master` ではなくサービス名などを付けます。
仮に、Organization名を **dwango**、 サービス名を **friends.nico** とします[^1]。

[^1]: この物語はフィクションです。 実在の人物、組織、サービスとは一切関係ありません。

#### チェックアウト

dwango に `tootsuite/mastodon` をフォークします。
コードを取得し、最新のタグからブランチを作ります[^2]。

```
$ ghq get -p dwango/mastodon
$ cd dwango/mastodon
$ git checkout -b friends.nico v2.0.0
$ git push origin friends.nico
```

Githubのリポジトリ設定でメインブランチを `friends.nico` にします。
次に、 **dwango/mastodonを自分のアカウントにフォーク** し、リモートリポジトリを登録します。

```
$ git remote add masarakki git@github.com:masarakki/mastodon
```

**さらに**、本家をリモートブランチとして登録します。

```
$ git remote add super https://github.com/tootsuite/mastodon
```

この本家のリモートブランチは、主に**バージョンアップのため**に使います。
これで、`super` が `tootsuite/mastodon`、 `origin` が `dwango/mastodon` 、`masarakki` が `masarakki/mastodon` を向くようになりました。
ここで `hub pull-request` のコマンド[^3]を叩くと `dwango/mastodon` に対するプルリクエストが作れます。


本体にも貢献する場合は、別のディレクトリに `clone` したほうが圧倒的に便利です。

```
$ ghq get tootsuite/mastodon
$ cd tootsuite/mastodon
$ git remote add masarakki git@github.com:masarakki/mastodon
```

こちらは `origin` が `tootsuite/mastodon` を向き、 `masarakki` が `masarakki/mastodon` を向く、普通のオープンソース開発と変わらない状態に出来ます。
個人のリモートリポジトリは `dwango/mastodon` からのフォークですが、
この状態で `hub pull-request` を叩くと、きちんと `tootsuite/mastodon` に対してのプルリクエストが生成され、通常と同じ感覚で開発できます。

本家も同時に開発する場合、同じデータベースの同じテーブルを参照していると、
スキーマの定義のマイグレーションがめんどくさいことになります。
その場合、 それぞれのディレクトリの `.env` ファイルに `DB_NAME=friends_nico_development` のようにデータベース名を指定してあげれば、プロジェクトごとにデータベースを分離できて便利です。

[^2]: `ghq` コマンドは便利だから使いましょう
[^3]: `hub` コマンドも(ry

### 開発ブランチ

開発ブランチは、当然 `friends.nico` から切ります。

```
$ git checkout -b new-feature friends.nico
$ # 開発
$ git push masarakki new-feature
$ hub pull-request
```

### バージョンアップ

現在、 `friends.nico` ブランチは、本家の `v2.0.0` から派生して、独自機能がいくつかコミットされている、という状態になっています。
さて、本家の開発が進み無事に `v2.0.1` がリリースされたとします。
こちらのリポジトリも新しいバージョンに追従しましょう。

```
$ git tag                        # v2.0.0 までしかない
$ git fetch super
$ git tag                        # v2.0.1 がある
$ git checkout -b merge-v2.0.1 friends.nico
$ git merge v2.0.1
$ # コンフリクトいっぱいなおす
$ git push masarakki merge-v2.0.1
$ hub pull-request
```

### デプロイ

デプロイには本家同様 `capistrano` を使いたいところですが、様々な秘匿情報を使う都合上、公開リポジトリには入れられないことが多いでしょう。
そのため、社内のGithub:Enterpriseに専用のリポジトリを作り、
その中から `https://github.com/dwango/mastodon` の `friends.nico` タグをデプロイするような設定をしています。
また、プルリクエストを簡単にテストできるように、 `masarakki/merge-v2.0.1` のようにブランチを指定して開発環境にデプロイできる仕組みもあったほうが良いでしょう。

### もっとヤバイ場合

某P社は複数のマストドンインスタンスを運用しており、
`某p社/mastodon` に `pなんとか` と `pなんとか-music` という2つのブランチを抱えています。

聞くところによると某P社は、プルリクエストベースではなく、
**社内のリポジトリで開発されたものを公開リポジトリにpushするだけ** という方針をとっているらしいのですが、
もしこれがプルリクエストベースの開発フローに移行した場合、間違いを起こさず運用できるのか・・・。
自信がないので今後の課題かもしれません。
