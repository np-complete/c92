# マストドンとは

マストドンは、Twitterライクな**分散型**SNSです。
**OStatus / ActivityPub プロトコル**によりGNU Socialと互換性を持ち、**AGPL**のライセンスで[Github](https://github.com/tootsuite/mastodon/)に公開されています。


## 分散型

マストドンは、誰でもサーバ(マストドン用語で**インスタンス**という)を建てることができます。
インスタンスは完全に孤立しているわけではなく、世界中のインスタンス同士が通信しあい、
**リモートフォロー**により、どのインスタンスの人でもフォローし発言(**トゥート**)を見ることができます。
このような仕組みにより、マストドンは**分散型**と呼ばれます。
分散型という特徴により、ユーザは自由にインスタンスを選択することができ、
 どこかの一企業によりマストドン全体が支配されることはありません。

## OStatus

OStatusは、分散SNSを実現するためのプロトコルで、いくつかの既存技術を組み合わせて作られたプロトコルです。

- [WebFinger](https://tools.ietf.org/html/rfc7033) ユーザIDのデータ形式を定義する
- [Atom](https://tools.ietf.org/html/rfc4287) 投稿を配信するプロトコル
- [Salmon](http://www.salmon-protocol.org/) 投稿にコメントをつけるプロトコル
- [PubSubHubbub](https://pubsubhubbub.appspot.com/) データの変更を外部にリアルタイム通知する

これらの技術を組み合わせてOStatusが実現されています。

## ActivityPub

[ActivityPub](https://www.w3.org/TR/activitypub/) は、OStatusに変わる新しい分散SNSプロトコルです。
いろいろな技術の詰め合わせであるOStatusとは違い、
分散SNSに必要なメッセージのやり取りに関わる様々な機能を整理し、たった2つのエンドポイントとJSONフォーマットに押し込めています。

マストドンではバージョン1.6からActivityPubが搭載され、サーバ間の通信がActivityPubでもできるようになりました。
他のOStatus互換システムのActivityPub移行次第ですが、OStatusはいずれサポートされなくなる予定です。

ユーザ検索などのメッセージのやり取りと直接関係ない部分は仕様に規定されていないように見えます。
マストドンでは従来通りWebFingerを使うようです。

## AGPL

マストドンは、**AGPL**のライセンスで公開されています。
AGPLの大きな特徴は、**サーバで動かす場合でもソースコードの公開義務がある**ことです。
このため、企業が運営するインスタンスであってもソースコードが強制的に公開され、
インスタンスの信頼性を担保する効果があります。

カスタマイズしている場合の多くは、サービス名でブランチを作り、それをデフォルトブランチにしているようです。
例えば [friends.nico](https://friends.nico) の場合、 リポジトリは [github.com/dwnago/mastodon](https://github.com/dwango/mastodon) で、デフォルトブランチは `friends.nico` です。
