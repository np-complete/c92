## フロントエンド

フロントエンドは既に `webpacker` 化されていますが、Rails 5.1.1 の流儀からも外れ、
完全にJavaScript世界の流儀で構成されています。
ファイルは `app/javascript` 以下にすべて格納されています。

### css

cssは `scss` が使われいて、`app/javascript/styles` 以下に格納されています。
完璧ではないものの命名規則に[**BEM**](http://getbem.com/)を採用しているようです。

webpackの設定で、通常は `app/javascript/styles/application.scss` をエントリーファイルにしますが、
`app/javascript/styles/custom.scss` がある場合はそちらをエントリーファイルにします。
デザインをカスタマイズする場合、 `custom.scss` から `application.scss` と独自定義のファイルを読み込むようにします。

### JavaScript

`app/javascript/mastodon` というディレクトリが存在し、おそらくこれがマストドンのフロントエンドの重要なファイル群だというのはひと目でわかります。
が、とりあえず一旦webpackがどんな動作をするのか確認しておきましょう。
webpackの設定を見ると、どうやら `app/javascript/packs` の中にあるファイルをエントリーファイルにして、
何種類かのJavaScriptファイルを生成しているようです。
いくつかあるうち、`application.js` 以外は管理画面ようなどの小さなファイルです。
webpackの設定では、 `custom.js` を除外するような記述が書かれていますが、
どうやらcssの時のようにカスタマイズはここに書けというわけでは無さそうで、**これが何を意味するのかわかりません**。
`application.js` は案の定 `app/javascript/mastodon/main` を読み込んでおり、やはり `app/javascript/mastodon` が本命であることが確認できました。

さてコードの中身を見ると、完全に **React** と **Redux** で作られたシングルページアプリケーションだということがわかります。
TypeScriptやFlowなどの型支援をつかうような **トチ狂った選択** はしていません。

`actions/` があり、 `reducers/` があり、おそらく教科書の通りのReduxだと言えるのではないでしょうか。
実際のコンポーネントは、 `features/` の下にそれぞれの要素ごとに実装されています。

カスタマイズしたい場合は、 *まぁ・・・頑張れや・・・* としか言えません。
Rubyのような便利なクラス拡張などはできません。
既存のファイルをガリガリいじっていくことになるでしょう。当然アップデートの追従は大変です。
JavaScriptいじるときは**覚悟を持って**やる必要があるでしょう。
