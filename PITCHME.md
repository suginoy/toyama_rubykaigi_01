## Crowler on Rails

---
## 自己紹介

- @suginoy
  - 「すぎの」
- フリーランスプログラマー
  - 2011 ~
  - 川崎在住
- 富山県小矢部市出身
  - 某Rubyコミッタも同じ中学校出身

---
## 地域Ruby会議の開催  
## おめでとうございます！

- 第1回参加メンバー
  - 富山出身のメンバーがいる某女性ボーカルユニットKalafinaのネタをスキあらばぶっこむ係

---
## 今日のお題 

### Ruby on Rails を使って、"個人で" クローラーを作ってみた話

---
## あなたに質問
- Rubyをそれなりに使っている人？
- Ruby on Railsについて少し知っている人？
- Ruby on Railsを仕事等で使っている人？
- クローラーを作ったことがある人？

---
## 動機

### ある日思い立つ。
### 「メタサーチエンジンを  
###  作ってみたい。」

---
## メタサーチエンジン  
## (メタ検索エンジン)
## とは？

> メタ検索エンジン（メタけんさくエンジン）は、入力されたキーワードを複数の検索エンジンに送信し、得られた結果を表示するタイプの検索エンジン。メタサーチエンジン、横断検索エンジンとも呼ぶ。

[Wikipedia「メタ検索エンジン」](https://ja.wikipedia.org/wiki/%E3%83%A1%E3%82%BF%E6%A4%9C%E7%B4%A2%E3%82%A8%E3%83%B3%E3%82%B8%E3%83%B3)

---
## 横断検索
- ホテルとか
- 映画館とか
- ECサイトとか

--- 
## 特定ジャンルの  
## 横断検索がしたい

- データが必要
- 往々にして、サイトがライバルどうしだったりする
- API があるのは稀

---
## 自分で取ってくるしかない

---
## クローラーを
## 作ってみよう

---
## クローラーの効能
- HTTP、HTML、SEO、文字コードに詳しくなる
- データモデリングの練習になる
- 各サイトの実装方針に詳しくなる
  - SPA
  - APIの分け方、バージョニング
- 中の人より仕様やバグに詳しくなる
- 一意なURLとは何かといった哲学的な問いを考えだす

---
## 今日の話の定義(1)
- クロール/クリーリング/クローラー
  - 外部WebサイトからHTML/JSON/XML/画像等を取得すること
- スクレイピング/スクレイパー
  - クローラーが取得したHTML/JSON/XML等からデータを取り出すこと(+保存)

---
## 今日の話の定義(2)
- ドメインモデル
  - クロール対象のサイトのデータモデル
  - ActiveRecord 使う関係でDBのレコードとごっちゃに話す

---
## 制約
###  一人でコツコツ作る
- 時間は有限
- コストは有限
- データの修正等の手作業は最小限
- あまりにも複雑な仕掛けは無理

---
## 使う技術の検討

- Ruby
- Puppetter(JavaScript)
- BeautifulSoup(Python)

---
## Railsで作れるのでは？

- RailsはHTTPリクエストを受けてレスポンスを返す
- クローラーはHTTPリクエストしてレスポンスを解析する

### 逆のことをやればよさそう？

---
## Rails を使うメリット
- ActiveRecord による容易なDBアクセス
- ActionDispatch/Rack::Utils によるHTTP周りの便利ライブラリ
- ActiveSupprt を始めとした便利ライブラリ

---
## 他の言語の技術
- Puppetter(JavaScript)
- BeautifulSoup(Python)
- etc.

### => 得意でないので見送った。

---
## Rubyをキメると  
## 気持ちいい(by Matz)

---
## インフラ
- GCP の機運が高まっていた
- 多少の経験と学習のしやすさは AWS

---
## AWSを選択
- ついでに AWS資格も取得してみた。
- IAMとVPCに不安があった。

---
## 諦めたこと
- AWS ECS(Docker)の利用は見送り
- EC2を選択

---
## Rails に足りないもの
- HTTPリクエストを作ってレスポンスを保存するクローリング
- レスポンスを解析するスクレイピング
- 継続運用するための種々の構成

---
## その前に要件を確認

---
## 対象のサイト
- サイト数は数個〜十数個
- URL の種類は1サイトあたり20~30個ぐらい。
- 1サイトあたり数千ページから数十万ページ
- 状態がたまに変わる
  - 公開終了、セール、coming soon とか。
- URLに一意を示すID(コード値) を持っていることが多い。
- ログインしなくても参照可能なページが多い。
- 最大頻度でも1日一回クロールで十分そう。

---
## URLが登場する箇所

- `<body>` 中の静的な `a` タグ `href` 属性
- `<head>` タグ内の `<link>` タグ
  - `rel="canonical"`
  - `rel="next"`
  - `rel="prev"`
- JavaScript が叩きに行っているエンドポイント
- HTTP リダイレクトの Location ヘッダ
- sitemap.xml
- robots.txt
- JSON-LD 内の microdata

---
## URL をどう保存するか
- 単純な文字列だけではダメそう

---
## 同じURLでも
- HTTPヘッダで制御されている場合がある。
  - cookie
  - referer
  - 無限スクロールのAPIリクエストでHTTPヘッダで制御しているサイトがあった。
    - リクエスト時にページ番号を `X-xxx-PAGE: 2` で送る。
    - レスポンスヘッダで `X-xx-RESULT: OK` だったら次のページあり

---
## ページネーション対応
- ページ番号などは、別途抽出、保存しておくと使いやすい。
  - あるページから次のページの URL を作る, etc.

---
## スクレイピングの  
## 基本モデル

- Location... 対象のサイトの URL を保存
- Crawling... Locationからクロールしたレスポンス結果を保存
- Scraping... Scraping の成功、失敗を保存
- 上記3つを `Crawler`クラスと `Scraper`クラスが操作する

---
## 必要な仕様
### 後日クロールできるようにする
  - 1日にクロールできる数には限界がある
    - [Admin's Bar 5: 86,400の壁](http://admins.bar/5/)

---
## 必要な仕様
### クローリングとスクレイピングを分ける
- 複数ページを連続で辿らない
  - どこでエラーが起きるかわからない
- 1つのURL単独で処理できるようにする
- URLのクロール順に依存しない
- サイトのドメインモデルをできるだけ分離
---
## 実行イメージ

---
```rb
> url = 'https://example.com/items/ID001'

> location = Location.register!(url)

> crawling = Crawler.crawl(location) # => 裏でHTTPリクエスト

> crawling.http_status # => 200

> crawling.body # => "<html><head>...</head></html>"

> scraping = Scraper.scrape(crawling)

> scraping.status # => 200 (OKの意)

> Item.find_by(code: 'ID001') # => 欲しい物ができているか確認
```

---
## Location
- URLと付随情報を保存する
  - リクエストパラメータ
  - リファラー（同じクラス）
  - ページネーション情報
  - 次回更新日
- URL というクラス名は避けた。

---
```rb
class Location < ApplicationRecord
  has_many :crawlings

  serialize :headers, JSON

  validates :url,
    presence: true,
    uniqueness: { scope: [:page, :per_page] }
  validates :page, 
    numericality: { only_integer: true },
    allow_blank: true
  validates :per_page,
    numericality: { only_integer: true },
    allow_blank: true

  def self.register!(location_url, page: nil, per_page: nil, referer: self.top_page)
    # 後述
  end
end
```

---
## リファラー重要
- `href=/index.html` などの相対パスが復元できない
- リファラーのホストをチェックするAPI
  - CORS
- リダイレクト時やリンクは Location を作って紐付ける

---
```rb
class Location < ApplicationRecord
  # HTTP Location
  belongs_to :following_location,
    optional: true, class_name: 'Location'
  # HTTP referer or href
  belongs_to :referer,
    optional: true, class_name: 'Location'
  # <meta link="canonical">
  belongs_to :canonical_location,
    optional: true, class_name: 'Location'
end
```

---
## Crawling
- Crawler が HtmlCrawler/ApiCralerに移譲したレスポンス結果を保存
  - レスポンスヘッダ
  - レスポンスボディ
  - クロール日時

---
```rb
class Crawling < ApplicationRecord
  belongs_to :location
  has_many :scrapings

  serialize :headers, JSON

  validates :http_status, presence: true
  validates :run_date, presence: true
end
```

---
## Scraping
- Scraper が FooScraper に移譲した処理結果を保存

---
```rb
class Scraping < ApplicationRecord
  belongs_to :crawling

  validates :status, presence: true
  validates :scraper_name, presence: true
end
```

---
## Location その他
- 自身の URL に対応する Crawler クラスと Scraper クラスを推測する

---
## 対象のURLどうする？
- Web ページにはいろんなものが置いてある
  - 広告、ソーシャルボタンとか
- ヘルプページなどはクロールしたくない
  - データをできれば少なく

---
## ドメインで絞る
- 定数でドメインのリストを持つ
- CDNの画像などを別にすればうまくいく

---
## config gem

おなじみ定数管理ライブラリ

---
## config/settings.yml

```yaml
hostname:
  mysite:
  top: example.com
  api: api.example.com
```

---
```rb
class Location < ApplicationRecord
  def self.target_hosts
    [
      Settings.hostname.mysite.top,
      Settings.hostname.mysite.api,
    ]
  end
end
```

---
## 除外するURLを
## DBテーブルにする

- 該当したらURLを登録しない
- もしくは、以後クロールしない
- `ExclusiveLocation` と名付けた

---
## 相対パス等の復元が必要
- URI モジュールを活用
  - `URI.parse` を通せば後は便利に使える。

---
```rb
class Location < ApplicationRecord
  def self.register!(location_url, page: nil, per_page: nil, referer: self.top_page)
    return nil if location_url.blank? || location_url.in?(['/', '#']) || REJECT_SCHEMES.any? {|scheme| location_url.start_with?(scheme) }
    return nil if !location_url.start_with?('http') && !location_url.start_with?('/')

    begin
      uri = URI.parse(location_url)
    rescue URI::InvalidURIError
      # エンコード間違い
      uri = URI.parse(CGI.escape(location_url))
    end

    return nil if uri.host && !uri.host.in?(self.target_hosts)
    return nil if uri.path.blank? && uri.fragment.present?

    referer_uri = URI.parse(referer&.url || HOST_URL)
    if uri.relative?
      uri.scheme ||= (referer_uri.scheme || DEFAULT_SCHEME)
      uri.host = Settings.hostname.mysite
    end
    page ||= parsed_query_page(uri.query)

    url = uri.to_s
    location = Location.find_or_initialize_by(url: url, page: page, per_page: per_page)
    location.referer ||= referer
    location.target  = ExclusiveLocation.target_url?(url)
    location.next_crawling_date ||= Date.today
    location.save!

    location
  rescue => e
    binding.pry if Rails.env.development?
    Rails.logger.error("Location.register!:#{e.message}:#{e.backtrace}")
  end
end
```

---
## 次はクローリング

---
## Crawler クラス

- Location クラスからクローリング処理を移譲するクラスを判別。

---
## 3種類のクローラーを  
## 用意

- HTMLクローラー(HtmlCrawler)
  - 普通にリクエストとレスポンスを受ける。
  - HTTPヘッダは Chrome 同等
- API クローラー(ApiCrawler)
  - JavaScript などからのアクセスのフリをする。
- ブラウザ型クローラー(BrowserCrawler)
  - ヘッドレスブラウザとしてアクセスする。

---
## HTTPクライアント  
## ライブラリの選定

- いろいろある。
- 使ってみたものをいくつか紹介

---
## OpenURI
- 最初はこれで始めた。
  - リダイレクトのオプションを知らず、検出できなかった。
  - デバッグで気づいた。
- 同様の理由でブラウザ操作型もやめた。
  - リクエストした URL と現在の URL の比較でしかリダイレクトの検出ができない。
  - phantom.js の開発終了

---
## Net::HTTP
- HTTPプロトコル専門ではないので使いにくかった。

---
## Mechanize
- 抽象度が高すぎて不向き

---
## phantom.js
- 今は亡きヘッドレスブラウザ
  - 仕事ではお世話になりました。
- JavaScript実行前のHTMLがわからない
- リダイレクトの検出に苦戦
  - リクエスト時のURLと現在のURLを比較

---
## 最終的に typhoeus
- libcurl ラッパー
  - 機能は一番豊富そう
- 並列リクエスト機能を使う可能性があった
  - 後述の理由により使わずじまい
- ブラウザ型は必要なかった。
  - ログイン不要
  - パーソナライズ箇所不要

---
## 諦めたもの
- VASILY(当時の社名)みたいなかっこいい並列リクエスト
  - [iQONを支えるクローラーの裏側](https://www.slideshare.net/TakehiroShiozaki/iqon-54979883)
  - 個人で扱うには複雑すぎる。

---
## サイトあたりの
## クローリングルール

- 1 つのドメインにたいしては、直列に URL を 1 つずつクロール

---
## クローリングの紳士協定

- [Webサイトをクローリングするときに気を付けたい2つのこと](https://ascii.jp/elem/000/001/177/1177656/)
  - 1秒に1回程度のアクセス頻度であること
  - 応答があってから新しい要求をするようプログラムを組んでいること
- Librahack事件
  - 今日は法律については触れないので注意されたし

---
### クロールの単位
- 単位時間あたり
  - 1日あたりで済んだ
- 同じURLを1回アクセス
- 対象サイトのページ数と更新頻度を見積もり、おおよそ十分だとわかった。

---
```rb
class ApiCrawler
  def crawl(location)
    REQUEST_HEADERS = {
      # ブラウザからコピーしたヘッダを Hash で定義
    }
    request_headers =
      if location.referer
        REQUEST_HEADERS.merge(
          'Referer' => location.referer.url,
        )
      else
        REQUEST_HEADERS.except('Referer')
      end
    @request =
        Typhoeus::Request.new(
          location.url,
          method: :get, 
          headers: request_headers
        )
    @request.run
  end
end
```

---
## Module#delegate

- ActiveSupport の便利拡張
- 外部ライブラリのラッパーをつくるときに移譲が便利にできる。

---
```rb
class ApiCrawler
  delegate :response, to: :@request
  delegate :headers, :status, to: :response
end
```

---
## HTTPスタータスの管理

Rails を使うと Rack が付いてくるので、 Rack::Utils をありがたく流用する。

---
### Rack::Utils::SYMBOL_TO_STATUS_CODE

- `Rack`
  - WebサーバーとAPサーバーのインターフェース
  - Railsに付いてくる
- HTTP ステータスの番号と内容を持ったハッシュ

---
```rb
class SiteA::Crawling < ApplicationRecord
  validates :http_status,
    presence: true,
    inclusion: { in: Rack::Utils::SYMBOL_TO_STATUS_CODE.values }

  Rack::Utils::SYMBOL_TO_STATUS_CODE.each do |description, status_code|
    define_method("#{description}?") do
      http_status_before_type_cast == status_code
    end
  end
  # 足りないのは追加
  def redirect?
    http_status.in?(300..308)
  end
end
```

---

```rb
crawling = Crawling.last
crawling.rediret?   # => false
crawling.ok?        # => false
crawling.not_found? # => true
```

---
## html の保存

- 現代の文字エンコーディングはほぼ `UTF-8`

- それなりにサイズが大きい
- 必要な箇所だけ削ったりせず、まるごと保存する方針
- JavaScript/CSSは保存しない
- できれば圧縮したい
- 画像、動画、音声などのファイルについては、今回は割愛

---
## html の保存

- とりあえずテキストで ActiveRecord に保存することにした。
- `Crawling#body`

---

```ruby
class Crawlings < ActiveRecord::Migration[5.0]
  def change
    create_table :crawlings do |t|
      t.belongs_to :location, foreign_key: { to_table: :locations }, null: false
      t.date :run_date, null: false
      t.integer :http_status, null: false
      t.string :crawler_name, null: false
      t.text :headers, null: false # レスポンスヘッダ
      t.text :body  # レスポンスボディ

      t.datetime :created_at, null: false
    end
  end
end
```

---
## AWS S3 に保存することを検討

- PostgreSQL に保存する場合、他のカラムに比べてサイズが大きいのが気になる
- HTML/JSON を AWS S3 に保存してはどうか？

---
## ActiveStorage
- ストレージのインターフェイス
  - Railsに付いてくる
- 設定をすれば、ローカルファイルや S3 への保存までよしなにやってくれる
- S3のホスティング機能と合わせて、クロールしたHTMLがブラウザで確認できる  
  - これは便利！

---
## S3 に保存する懸念(1)
- 自分の AWS のドメインのリファラーで対象サイトのファイルを取得するリクエストが飛んでしまうことに気づいた。
  - まずそう
  - うっかり認証外すと完全にアウト

---
## S3 に保存する懸念(2)
- 管理するストレージが RDS (AWS のRDBMS サービス) 以外を増やしたくなかった。
  - サービス以外に、キー管理なども増える

---
## S3 に保存する懸念(3)
- ActiveRecord に HTML が入ってないのはデバッグなどでなにかと面倒そう。
  - 裏側でHTTPリクエストが走っているなど

---
## S3 はやっぱナシ

### しかし、圧縮はしたい。

---
## よくよく考えると...

### ブラウザは圧縮されたHTMLを受け取っている

---
## Accept-Encoding /
## Content-Encoding

### リクエスト時に `Accept-Encoding: gzip, deflate, br` を付ける
### => レスポンス時に `Content-Encoding: gzip` で body を gzip で返ってくる
###    (Web サーバーが対応していれば)

## 思いついた！
### => gzip でレスポンスが返ってきたらそのままバイナリ保存、取り出すときに gzip を戻す
### テキストでレスポンスが返ってきたら、 gzip 圧縮してバイナリで保存、取り出すときに gzip を戻す

---
## Vary ヘッダー

- 「これによってはレスポンスを変えるよ」というレスポンスヘッダー
- gzip の場合は、 `Vary: cccept-encoding` というレスポンスヘッダーが返る。

---
## ActiveSupport::Gzip

- gzip 処理ができる Zlib ラッパー
  https://api.rubyonrails.org/classes/ActiveSupport/Gzip.html
- Railsに付いてくる。

---
## ActiveRecordActiveRecord::AttributeMethods#[],  
## ActiveRecordActiveRecord::AttributeMethods#[]=

- ActiveRecord のアクセサを上書きするときに使える。
- ActiveRecord の DB からキャストされた属性が入っている。
- 今なら attributes API で置き換えられるかも？

---
## Migration ファイルの修正

```diff
class Crawlings < ActiveRecord::Migration[5.0]
  def change
    create_table :crawlings do |t|
      t.belongs_to :location, foreign_key: { to_table: :locations }, null: false
      t.date :run_date, null: false
      t.integer :http_status, null: false
      t.string :crawler_name, null: false
      t.text :headers, null: false
-      t.text :body
+      t.binary :body

      t.datetime :created_at, null: false
    end
  end
end
```

---
### gzip 判定

```rb
class Crawling < ApplicationRecord
  serialize :headers, JSON

  # NOTE: 汎用的に使うなら 小文字に揃える
  def gzip?
    headers['Vary'] == 'Accept-Encoding' &&
      headers['Content-Encoding'] == 'gzip'
  end
end
```

---
### ActiveRecord アクセサのオーバーライド
```rb
class Crawling < ApplicationRecord
  def body
    return nil unless self[:body]

    if gzip?
      ActiveSupport::Gzip.decompress(self[:body])
    else
      # NOTE: `.endode('utf-8')` raise the error below.
      # `Encoding::UndefinedConversionError: "\xE6" from ASCII-8BIT to UTF-8`
      self[:body].force_encoding(Settings.encoding.mysite)
    end
  end
end
```

---
```rb
class Crawling < ApplicationRecord
  def body=(body_html)
    self[:body] =
      if body_html.blank?
        nil
      elsif gzip?
        ActiveSupport::Gzip.compress(body_html)
      else
        body_html
      end
  end
end
```

---
## ところがある日、
## gzip がうまくいっていない

- 同じHTTPヘッダは複数返ってくることがある。
- 思い込み注意

---

```
Vary: Accept-Encoding
Vary: User-Agent
```

---
## 同一HTTPヘッダの動作

- Rack の仕様では、配列になる。
- 型が String の場合と Array の場合がある。
- とても困る。

---
## Object クラスを拡張してしまえ

### `Object#equal_or_include?` が誕生

```rb
class Object
  def equal_or_include?(other)
    (self == other) || (respond_to?(:include?) && include?(other))
  end
end
```

---
## config/initializers/
- 標準クラスをアプリケーション独自に拡張する場合はここに置いておけばよい。
- やりすぎ注意

---
```diff
class Crawling < ApplicationRecord
  def gzip?
-   headers['Vary'] == 'Accept-Encoding' &&
+   headers['Vary'].equal_or_include?('Accept-Encoding') &&
      headers['Content-Encoding'] == 'gzip'
  end
end
```

できた。

---
## それでもある日、 gzip がうまくいっていない
- `Content-Encoding: gzip` が返ってきても `Vary: Accept-Encoding` も返すとは限らない。

> nginxは設定ファイルにgzip_vary on;と書かないと
> Vary: Accept-Encoding をレスポンスヘッダに付加しません。
https://qiita.com/cubicdaiya/items/09c8f23891bfc07b14d3

---
## `Accept-Encoding` チェックをやめた

### Object#equal_or_include? 消滅

### config/initializers/ に平和が訪れる

---
## 教訓
### サーバーの設定が突然変わることもある。

---
## ここでテストについて

2008-03-24 はてなブックマークの作り直しについて
https://naoya-2.hatenadiary.org/entry/20080324/1206354054

---

> 自分は常々、テスト駆動開発とサービス開発は相性が悪いなと思っていました。新しい機能を作っているときや、新しいサービスを作っているときは、自分でも答えが見えていない状態で作っていることが多くあります。コードを書いているうちに少しずつ問題が解決されていって、最初は見えていなかったものが見えるようになり、答えがみえるようになる、ということが多々あります。作っては壊しを何度も繰り返すこともあります。
> テスト駆動開発では、細かい単位とは言え、ある程度事前に何を作るかを決めてテストを書きます。このプロセスが相性が悪かったのです。

---
### 最初は試行錯誤するので `rails console` で確認するだけ

DB の migration を何回やったことか。

---
## テスト書き始め
- スクレイピングはテストを書かないと進捗が上がらない
- 他の処理は、保存に失敗する、処理ステータスでわかる、例外が上がるのですぐわかる
- HTML を保存しては、スクレイピングの結果が合っているを確認する地味作業
  - 外部サイトなので、レスポンスはモックする

---
## 諦めたもの
### 当初は Minitest を憶えようと  
### したが、進捗が上がらず、  
### 使い慣れた RSpec に切り替える

---
## 多様なページレイアウト
## にどう対応するか

---
### URLから対応する Scraper を判定する
- 対応できたこととこだけ動かす
  - XxxScraper をどんどん追加
  - Null Objectパターンと組み合わせて、できたら再処理
  - `Scraping#scraper_name`
- そのためにはクロールとスクレイピングを分離させる

---
## URL から対応する  
## Scraper をどうやって
## 判定するか？

---
## 諦めたもの
- 最初は Rails の `config/routes.rb` のような、URLからスクレイピングのクラスを判別するかっこいいルーターを考えた。

---
## できた実装

```rb
class Location < ApplicationRecord
  def guess_scraper_class
    [
        BarScraper,             # /bar
        FooScraper,             # /foo
        ItemsScraper,           # /items/ID001
        # NOTE: 以下30行くらいクラスが並ぶ
        # ...
        DefaultScraper,         # Null Object
    ].detect {|scraper|
      scraper.match_url?(url)
    }
  end
end
```

---
## XxxScraper
- 実装する public メソッドは 2 つだけ
  - `.match_url?`
  - `#scrape`
    - 中にめちゃくちゃprivateメソッドがある
    - XPath や `each` 、 `to_s` / `to_i` まみれ


---
## 諦めたもの
- VASILY のような XPath のDB保存
- 過去にクローリングしたHTMLも解析できないといけない
  - スクレイピング時にクローリングした日付で判定して、ガッツリ分岐する

---
## 次はスクレイピング

---
## nokogiri
- HTML/XMLパーサー
- Ruby on Railsのセットアップでたまに躓くあれ
- XPathやCSSセレクターで要素を取得し、Stringに変換

---
## Ruby の基本
## ライブラリっぽく
## 扱える

### 必ずしもそうではなかった。
### `#index` メソッドにパッチが出せた。

---
## 複数サイトを扱う方法
- 実はこれまで `Location` や `Crawling` などと書いて説明してきたが、実際には対象サイトのモジュール名が付いている。

---
- 例
  - `FooSite::Location`
  - `BarSerivce::Location`
- DBのテーブル名にもプリフィックス付き

---
## モジュールの名前空間を
## 使う場合の問題点

- コードの重複が多い。
  - 横並びの修正がタイヘン
- 思わぬところでサイト固有の問題を発見するため、共通化に二の足を踏む。

---
## ドメインモデル

---
## コード体系
### 常にこういう構造を持つことにした。
- `id` とは別に `code` というユニーク制約を持つ属性
  - テーブルはNULL不可
- `name` という属性を持つ
- `code` `name` を持たない ActiveRecord はすべて紐付け等をするもの
- 対応するURLを生成するぐらい
  - ドメインモデルとクローラー側のモデルの結合度を少なくする

---
```rb
module CodeAccessable
  def to_param
    code
  end

  def original_path
    raise NotImplementedError
  end

  def original_url
    "https://#{Settings.hostname.mystite}" + original_path
  end
end
```

---
```rb
class Item < ApplicationRecord
  include CodeAccessable

  validates :code, presence: true, uniqueness: true

  def original_path
    "/items/#{code}"
  end
end
```

---
## データモデル、  
## データベース設計

---
## 同じモノでもサイトに
## よって呼び方が違う
- Item / Goods
- Genre / Category
- Menu / Section
- Movie/ Video

---
## 対象サイトの名前に
## 合わせる
- 認知負荷の低減
  - 実際にスクレイピング処理を書いてみるとわかる。
- 各サイトで微妙にデータ構造が違う
- 属性のマッピング情報を作ることも検討したが、デバッグの効率性などから見送った。

---
## どのURLからクロールしても問題ない設計

- `has_many` & `belongs_to` の組み合わせの問題点
  - belongs_to は先に親のモデルのスクレイピングが終わっていないと使えない。
    - 親のいないレコードとか作らないですよね？
    - できないこともないけど...
- 中身はなくてもコード値のみ設定された ActiveRecord インスタンスが必要になる

---
## has_many through 多用
- code 属性を持つモデルの has_many は has_many through を使う
- モデル数が1.5倍

---
## 諦めたこと

### active_hash gem

---
## ActiveHash

- DBテーブル使わずActiveRecordのように振る舞う
- レコード数が少ない場合に便利

---
## ActiveRecord
## に戻した

`has_many through` が使えない。

---
### ページのレイアウト変更に強い設計

- 以前にスクレイピング済みのモデルに対して、次回のスクレイピングでサイトのデザイン変更により、取得済み Item#name を空文字や `nil` で更新してしまうといったおそれがある。
  - 微細な改修が入るとか
  - キャンペーンバナーが挿入されたり

---
```rb
class Item < ApplicationRecord
  validates :code, presence: true, format: { /ID\d+/ } ## 常に presence: true
  validates :name, length: { maximum: 100 } ## ここでは presence: true を指定しない

  with_options(on: :complete) do
    validates :name, presence: true
  end
end
```

### スクレイピングの処理の最後

```rb
def scrape
  # めっちゃたくさんのメソッド呼び出し

  item.save!(:complete) # => name が消えてしまっていたら raise する
end
```

---
## 問題点

- `update!` メソッドにコンテキストを渡せない
  - コードが地味に増えるを嫌ってやめた。
  - スクレイピングコードのノウハウが溜まってきたのもある。

### => `audit gem で変更履歴を持つことで代替とした。

---
## 諦めたこと

- VASILY社のようにXPathをDBに保存するのもやめた。
  - 一人で開発するには非常に書きにくい。

---
## パンくずリスト

- パンくずリストは2タイプある
  - 階層構造
    - カテゴリー
  - 画面遷移順

---
## パンくずリストの気づき

- パンくずリストは多対多の構造をしていることがある
  - 両端がポリモーフィック関連の `has_many through` が爆誕
- 恣意的に別のリンクになっていたりする
  - `<a href="/link">text</a>` タグの `href` とテキストが一致していなかったりする

---
## has_many / has_one

### has_one だと思っていたらまさかの...

---
## 最近はSPAっぽい
## サイトも多い
- Single Page Application
- 1 つページが HTML と 1 つ以上の API のレスポンスで構成される

--- 
## SPA例
- ブラウザにはHTMLのURLが表示される
  - HTML上でhrefに入っているのはこれ
- URL中のコードに対応したAPIでJSONを取得してHTMLに反映される

HTML URL `https://example.com/items/ID0001`
API URL  `https://example.com/api/items/ID0001`

--- 
## わかったこと
- ほとんどのサイトのAPIはサブドメインかパスの先頭で判別可能
  - サブドメイン型 `https://api.example.com/items/ID0001`
  - 先頭パス型 `https://example.com/api/v1/items/ID0001`

---
## 設計
1. とりあえず HTML URL をクロール
1. HTML URL のスクレイピング時に APIのURLのみ(Location)を作って保存
  - 他のことはやらない
1. JSON API のクロール
1. JSON API のスクレイピング時にドメインモデルを作って保存

---
## ブラウザ型のスクレイピングツールを使えば、
## HTML ページのみのクロールで済むのでは？

---
## API (JSON) を叩くとよい理由(1)
- DOM 要素を XPath や CSS セレクターや抽出するよりも JSON の方が正確な構造化がされている
  - HTML では並列にならんでいるけどデータモデルは子供

---
## API (JSON) を叩くとよい理由(2)
- より正確な値が手に入る
  - HTML 上では `分` と表示されているのが、実際は `秒` で取得できる

---
## API (JSON) を叩くとよい理由(3)
- メタデータがついていることもある
  - タグやラベルの分類
  - セールの期間
  - リスト系 API の総件数

---
## API (JSON) を叩くとよい理由(4)
- 対象のサイトでのデータベースのテーブル名やカラム名に近い名前が手に入る
- 例
    - HTML 上ではコード体系が同じ2階層の "MENU"
      `/MENU0001/MEMU0002` のような URL
      両方同じコード体型なので、何回層なのか、制限はあるのは不明
    - JSON を叩くと実際は "genre" と "category" の親子構造だったことがわかる

## API (JSON) を叩くとよい理由(4) - 2
- 過剰な実装を避けられる
- 名前付けや保存時の属性判別に悩むことが大幅に減り、開発が加速する
- 某作品にならって、真名と呼んでいる（余談）

---
## HTMLよりAPIの方が
## 変わりにくく変更に強いか？
- 実際はそんなに期待できないという感想
  - サイトデザインががらっと変わると、同時にAPIが新設・廃止されるケースも多い。

---
## 謎のSPAサイト
- 「SPAなのにどこのAPIも叩いてないぞ？」
  - HTML にはコンテンツの情報が見当たらない
  - Chrome Developer Tools の Network タブにリクエストが見当たらない

---
### 発見！！
`<script>` タグの中の JavaScript にコード辺を発見する。

```
<script>
   ...
    xxx.reactContext = {...}; // この JSON にページの内容が詰まっている
   ...
</script>
```

---
## JavaScript から  
##  JSON を抽出する

```rb
  ## script 中の .reactContext = {}; を抽出
  def react_context_json
    script =
      html.xpath('//script/text()').map(&:text).detect { |script|
        script.match?(/\.reactContext = /)
      }.split(/\.reactContext = /)[1][0...-1]

    json = JSON.load(script)

    deep_transform_hex_to_char(json)
  end
```

---
## JSON

- 標準ライブラリ
- とりあえず load 呼んだら Hash になる

---
```rb
  private

  def deep_transform_hex_to_char(object)
    case object
    when Hash
      object.transform_values { |val| deep_transform_hex_to_char(val) }
    when Array
      object.map { |val| deep_transform_hex_to_char val }
    when String
      ## NOTE: 記号が "x20" "x3F" などで埋まっている
      object.gsub(/(x[0-9A-F]{2})/) { |xHH| "0#{xHH}".to_i(16).chr }
    else
      object
    end
  end
```

#### ActiveSupport の Hash 拡張にインスパイア

---
## String#gsub

- UNIXコマンドの `sed 's/A/B/g'` 相当

---
## Integer#chr

> 与えられたエンコーディング encoding において self を文字コードと見た時、それに対応する一文字からなる文字列を返します。 引数無しで呼ばれた場合は self を US-ASCII、ASCII-8BIT、デフォルト内部エンコーディングの順で優先的に解釈します。

---
## String#to_i

> 基数を指定することでデフォルトの 10 進以外に 2 〜 36 進数表現へ変換できます。 
[https://docs.ruby-lang.org/ja/latest/method/String/i/to_i.html](https://docs.ruby-lang.org/ja/latest/method/String/i/to_i.html)

---
```rb
  def deep_transform_hex_to_char(object)
    case object
      # ...
    when String
      ## NOTE: 記号が "x20" "x3F" などで埋まっている
      object.gsub(/(x[0-9A-F]{2})/) { |xHH| "0#{xHH}".to_i(16).chr }
    # ...
    end
  end
```

---
## 悲報

### 数カ月後、この JavaScript コードが消滅し、 HTML から取得することになった。

### 真名が手に入っただけでもよしとしよう...。

---
## デザイン変わるよね？

- クローリング日時からベタにロジック分岐
  - `Crawing#run_date` 
  - リファクタリングするかもしれないが、そうそう起きない。
- 地味にテストを書く

---
## まとめ
- クローリングとスクレイピングの実例を示した。
- RubyとRailsはクローラーでの開発に使える。

---
## Q&A

---
## Tips紹介

---
### 隅つきカッコ

`<h1>【セール中】なんとか商品名</h1>` みたいなやつ

- 期間などで、動的に入ってくる。
- 判別不能

- 気づいたら除去する。
- そのために、スクレイピングはやり直せるように作っている。

---
### 嘘つき count

### リストを返すJSONに `count` という属性が含まれていることがある

```json
{
    "count": 111,
    "items": [
        {
            "id": 111,
            "name": "AAA",
            ...
        },
        {
            "id": 222,
            "name": "BBB",
            ...
        },
        ...
    ]
}
```

## 嘘つき count
### 実際にクロールすると、件数がたまに合わない
### => 信じない。最後のページの件数がページネーション単位と同じだったらもう1ページ URL を追加することにした。

---
### HTMLとDOMは違うのだよ

- Chrome DevTools などから XPath をコピーすると、取得できないことがある。
- `<table>` タグには `<thead>` と `<tbody>` が補われるという仕様がある。
- ブラウザを信じない

---
## Rails の仕様変更

- breaking changes がパッチバージョンでしれっと出てる。
- テストコード重要

---
## サイトマップのリンクに間違いがある

「このリンク間違ってる」
- 対応するコード書くのはめんどう

---
## サポートページから連絡

---
## 返事来た。

> ※本メールの内容を無断転載・複製・引用のために使用することを禁止させていただきます。

割愛

---
## 複雑な URL

---
### ?_=12340000000000000

---
### 最初はこのようなナイーブな実装を試みた。(省略)

### この実装の問題(省略)
- リストをどうする
- IDの管理をどうする
  - おおよそ1始まりで整数だったりするが、空き番がわからない。管理するロジックを考えなくてはならない。
  /items/ID00011
  - 特に、過去に存在していた URL なのかどうかわからない

---
```rb
class Location < ApplicationRecord
  DEFAULT_SCHEME = 'https'
  REJECT_SCHEMES = ['javascript', 'tel', 'mailto']
  ORIGIN = "#{DEFAULT_SCHEME}://#{Settings.hostname.example.com}" # config gem
  HOST_URL = ORIGIN + '/'

  belongs_to :following_location, optional: true, class_name: 'Location'
  belongs_to :referer, optional: true, class_name: 'Location'
  belongs_to :canonical_location, optional: true, class_name: 'Location'

  has_many :crawlings, class_name: 'Crawling'

  serialize :headers, JSON # リクエストヘッダー

  validates :url, presence: true, uniqueness: { scope: [:page, :per_page] }, format: { with: %r|\Ahttps?://[^\n]+\z| }, on: :create
  validates :page, numericality: { only_integer: true }, allow_blank: true
  validates :per_page, numericality: { only_integer: true }, allow_blank: true
end
```
