## Rails

マストドンのRailsは基本的には結構素直な[^1]Railsです。
この規模のアプリケーションにしては、非情に綺麗にまとまった[^1]コードだと思います。
常に最新のRailsに追従できており、`webpacker` も導入済みです。

[^1]: 異論は認める

### Gemfileを見てみよう

どんなgemが使われているか、 `Gemfile` を見てみましょう。

おなじみの `devise`, `doorkeeper`, `paperclip`、 `simple_form` や、
もちろんワーカーには `sidekiq`、hamlの実装には `hamlit` を使い、権限管理には `pundit` が使われています。
テストにもおなじみ `rspec`, `capybara`,  `rubocop`, `brakeman`, `faker`, `webmock` などが使われています。 `factory-girl` ではなく `fabrication` というのを使っているのは少し珍しいでしょうか。

現状でベストチョイスと言えるgemがふんだんに使われているように見ます。
このあたりは普通にRailsアプリを作る時の参考にもなるでしょう。

珍しいところで言うと、以前はJSON出力に `rabl` というライブラリを使っていました。
どうやら `v1.5.0` で大部分が `active_model_serializers` に置き換えられているようです。
この `rabl` というやつ、**控えめに言ってもクソ** だなぁと思っていたので排除されて大変嬉しく思いました。
とはいえ `active_model_serializers` が良いものだとは思わないので、なんで `jbuilder` じゃないのかなぁと思う次第です。

### 変わってる部分

普通のRailsっぽくないところは、**サービス層** が導入されており、
データベースの更新を伴うような複雑なコントローラはほとんど、その処理をサービス層に分離しています。
サービスの中では基本的にデータベースを操作が行われ、サービスからさらにサービスが呼ばれたり、特に **ワーカーにジョブを追加する** ことが重要なミッションになっているようです。

読んでみた感じ、通常これは `after_save` などの `ActiveRecord::Callbacks` を利用して実装する類のものが多いように思います。
コードを分離するなら `ActiveSupport::Concern` を利用するべきでしょう。

例えば、あるユーザが別のユーザブロックするという処理が、 `BlockService` として定義されています。
`BlockService` の中では、フォローを解除する `UnfollowService` を呼び、
そのあと `Block` を新規作成し、 `BlockWorker` と `NotificationWorker` をジョブに登録します。

もし、何かしらの理由で直接 `Block.create` をしてしまったら[^2]どうなるでしょう。
ブロックはされているのにフォローは切れていないという、通常では想定していない状況ができてしまいます。
サービス層を導入した場合、データベース更新に通常のRailsの流儀である `ActiveRecord` を使わず、サービス層を必ず使うというのを徹底させなければなりません。
これはレールから大きく外れていると断言でき、よい設計だとは言えないと思います。

通常のRailsの流儀では、`Block` モデルに `before_create` でフォロー解除するコードを書き、 `after_create` で2つのワーカーを登録するコードを書きます。
そうすると `Block.create` でそのコールバックが呼ばれるので、
既存のRailsの流儀で雑にデータベース操作しても同じことが実現できます。

ファイルを分割したり再利用したいなら、 `ActiveSupport::Concern` という仕組みが用意されているので、このようなコードにできます。
(この例だと分割しないほうが良いと思います。)

```
# app/models/concerns/unfollow_helper
module UnfollowHelper
  extend ActiveSupport::Concern

  included do
    before_create :unfollow_target
  end

  def unfollow_target
  end
end

# app/models/block.rb
class Block
  include UnfollowHelper
end
```

[^2]: Rails開発をしていると本番サーバで `rails console` してデバッグする、ということが稀によくある

### カスタマイズのポイント

カスタマイズする際は、Rubyのオープンクラスをうまく活用し、既存のコードの変更点を最小になるようにします。
マストドンは本体の更新スピードが早いので、なるべく元のコードに変更を入れないほうが管理が楽になります。

`app/lib/customs` などのディレクトリを切り、変更点はすべて `ActiveSupport::Concern` の流儀でモジュール書きます。
既存のファイルの変更点は、末尾で作ったモジュールを `include/prepend` するコードを追加するだけにできます。

具体的に、[ProcessHashtagsServiceを拡張するモジュール](https://github.com/dwango/mastodon/blob/friends.nico/app/lib/friends/process_hashtags_service.rb) と [読込する部分](https://github.com/dwango/mastodon/blob/friends.nico/app/services/process_hashtags_service.rb) のコードを例として挙げておきます。
