## Crowler on Rails

---
## 自己紹介

- @suginoy
  - 「すぎの」（「い」は不要）
- フリーランスプログラマー
  - 川崎在住
- 富山県小矢部市出身
  - 某Rubyコミッタも同じ中学校

---
## 地域Ruby会議の開催  
## おめでとうございます！

- 第1回参加メンバー

---
## 今日のお題 

### Ruby on Rails を使って、個人でクローラーを作ってみた話をします。

---
## 動機

### ある日思い立つ。
### 「メタサーチエンジンを作ってみたい。」

---
### メタサーチエンジン(メタ検索エンジン)とは？

> メタ検索エンジン（メタけんさくエンジン）は、入力されたキーワードを複数の検索エンジンに送信し、得られた結果を表示するタイプの検索エンジン。メタサーチエンジン、横断検索エンジンとも呼ぶ。

Wikipedia「メタ検索エンジン」
https://ja.wikipedia.org/wiki/%E3%83%A1%E3%82%BF%E6%A4%9C%E7%B4%A2%E3%82%A8%E3%83%B3%E3%82%B8%E3%83%B3

- ホテルとか
- 映画館とか
- ECサイトとか

--- 
## 要は特定ジャンルの横断検索がしたい

- データが必要
- 往々にして、サイトがライバル企業どうしだったりする
- API があるのは稀

### => 取ってくるしかない

---
## クローラーを作ってみよう

#### ここでの定義
- クロール/クローラー
  - 外部WebサイトからHTML/JSON/XML/画像等を取得すること
- スクレイピング/スクレイパー
  - クローラーが取得したHTML/JSON/XML等からデータを取り出すこと(+保存)
- ドメインモデル
  - クロール対象のサイトのデータモデル

---
### 制約
- 一人でコツコツ作る
  - 時間は有限
  - コストは有限
  - データの修正等の手作業は最小限
  - あまりにも複雑な仕掛けは無理

---
### 誘惑
- 
  - 時間は有限
  - コストは有限
  - データの修正等の手作業はやりたくない

---
### Which HTTP Client

- Open URI
- Net::HTTP
- 
- Mechanize
- Selenium Web Driver
- Phantom.js
- other REST API libiraries


- nokogiri

---
## 使う技術

- Ruby
- Puppetter(JavaScript)
- BeautifulSoup(Python)

---
## Rails で作れるのでは？

- Rails は HTTP リクエストを受けてレスポンスを返す
- クローラーは HTTP リクエストして、レスポンスを解析する

### => Rails ベースで作れそう

---

- ActiveRecord 保存
- Rack::Utils
- ActiveSupprt の便利メソッド

---
### その他の技術
- Puppetter(JavaScript)
- BeautifulSoup(Python)

---
### インフラ
- GCP の機運が高まっていた
- 多少の経験と学習のしやすさで AWS の方がやりやすいと判断
  - ついでに AWS資格も取得してみた。
  - IAM と VPC に多少の不安があった。

### => AWS を使うことに

---
## 諦めた 
### Docker の利用は見送って EC2 に

### Rails に足りないもの

- HTTPリクエストを作る
- レスポンスの解析

---
### 書籍で紹介されているのには問題点がいくつかある

- 単発のページをスクレイピングする程度で、継続運用する手法がない
- 対象の Web サイトのその時点のスナップショットしか取れない
  - 公開終了
  - 30x 系
  - ページデザイン変更
  - サーバーメンテナンス、障害
- どう保存するか


---
### 使ってみた

### 

---
### 最終的に

- libcurl wrapper
- phantom.js も不要になった。

---
### 最終的に
理由

- ブラウザ操作型は、リダイレクトの検出が面倒だった。
  - リクエスト時の URL と現在ブラウザが開いている URL を比較 ,etc.
  - 同じ理由で、 Open URI は、リダイレクトのオプションが当初わからず捨ててしまった。
- HTTP ヘッダや cookie を柔軟に設定できる必要があった。
- 並列リクエストを使う可能性があった（後述の理由により最終的に使わず)

---
### ターゲットサイト
- 数千ページから数十万ページ
  - 商品とかカテゴリとかそういうのを想像してください
- 状態がたまに変わる
  - ID を持っていることが多い

### 

---
## テスト


最初は試行錯誤するので、 `rails console` で確認するだけ

```
自分は常々、テスト駆動開発とサービス開発は相性が悪いなと思っていました。新しい機能を作っているときや、新しいサービスを作っているときは、自分でも答えが見えていない状態で作っていることが多くあります。コードを書いているうちに少しずつ問題が解決されていって、最初は見えていなかったものが見えるようになり、答えがみえるようになる、ということが多々あります。作っては壊しを何度も繰り返すこともあります。

テスト駆動開発では、細かい単位とは言え、ある程度事前に何を作るかを決めてテストを書きます。このプロセスが相性が悪かったのです。
```

2008-03-24 はてなブックマークの作り直しについて
https://naoya-2.hatenadiary.org/entry/20080324/1206354054

---
## rails console でこんなことをしていた。

```rb
url = 'https://example.com/items/ID001'

location = Location.register!(url)

crawling = Crawler.crawl(location)

crawling.http_status # => 200

crawling.body # => "<html><head>...</head></html>"

scraping = Scraper.scrape(crawling)

scraping.status # => 200 (OKの意)

Item.find_by(code: 'ID001') # => 欲しい物ができているか確認
```

---
## テスト

- スクレイピングの処理はテストを書かないと進捗が上がらないことに気づく
- 他の処理は、保存に失敗する、処理ステータスでわかる、例外が上がるのですぐわかる
- HTML を保存しては、スクレイピングの結果が合っているを確認するという地味なテストコード
  - 外部サイトなので、レスポンスはモックする

## 諦めたもの

### 当初は Mninitest を憶えようとしたが、進捗が上がらず
### 使い慣れた RSpec に切り替える

## 最初の設計

- かっこいい URL からスクレイピング処理を判定する router
  - 最初は Rails の config/routes.rb のようなのを考えたがやめた。

```
Location のコード


```

---
## 諦めたもの

### 並列リクエスト 

- VASILYさんみたいなかっこいい並列リクエスト

iQONを支えるクローラーの裏側
https://www.slideshare.net/TakehiroShiozaki/iqon-54979883

---
## クローリングの紳士協定

- 1秒に1回程度のアクセス頻度であること
- 応答があってから新しい要求をするようプログラムを組んでいること

Webサイトをクローリングするときに気を付けたい2つのこと
https://ascii.jp/elem/000/001/177/1177656/

---
## サイトあたりのクローリングルール

- 1 つのドメインにたいしては、シリアルに URL を 1 つずつクロールする

---
## 必要な使用

- 複数ページのリンクをたどらない
  - どこでエラーになるかわからない
- 
- 後日でもリクエストできるようにする


- 対象サイトのページ数と更新頻度を見積もり、
- おおよそ十分だということがわかった。

SQL で次のような条件を書いて、

---
### ドメインを分ける
- クローラーの仕組みとスクレイピングの仕組みを分ける
- スクレイピングに1箇所でも失敗すると後続のクローリングができないのは困る
- 1 つの URL で完結するようにする

---
### クロールの単位
- 単位時間あたり
  - 1日あたりで済んだ
- 同じURLを1回アクセス

---
### 対象の URL はどうする？

- Web ページにはいろんなものが
- ソーシャルボタンとか

---
### データモデル、データベース設計

メタサーチエンジンを作る以上、同じモノでもサイトによって呼び方が違う。

- Item / Goods
- Genre / Category
- Video / Movie
- Menu / Section

### => 基本的に、対象のサイトの名前の合わせることにした。

- 認知不可の低減
  - 実際にAPI の JSON の属性名や CSS やHTMLのスクレイピング処理を書くと、こうしないときつい。
- 各サイトで微妙にデータ構造が違うため、
- 属性のマッピング情報を作ることも少しだけ検討したが、デバッグの効率性などから見送った。

### 諦めたもの

- VASILY のような XPath のDB保存

過去にクローリングしたHTMLも解析できないといけないため、
スクレイピング時にクローリングした日付で判定して、ガッツリ分岐する。

```rb

def scrape()

---
## 諦めたもの

### ドメインモデルの ActiveRecord にクロール日付を保存すること

- スクレイピングは、 ActiveRecord オブジェクトを大量に保存する処理になるため、普通に書くと煩雑でコード量が増える。
- きれいに拡張するアイデアがなかった。

### 
- 大統一データモデル(省略するかも)

メタサーチエンジンを作るため、サイトAの商品XとサイトBの商品Yが同じであるという情報をもつ必要がある

---
### コード体系

ActiveRecord はつねにこういう構造を持つことにした。

- `id` とは別に `code` というユニーク制約を持つ(null不可)
- `name` という属性を持つ
- 他の属性名は対象のサイトに合わせる

---
### 最初はこのようなナイーブな実装を試みた。

### この実装の問題
- リストをどうする
- IDの管理をどうする
  - おおよそ1始まりで整数だったりするが、空き番がわからない。管理するロジックを考えなくてはならない。
  /items/ID00011
  - 特に、過去に存在していた URL なのかどうかわからない

---
### 対象のドメインのモデルとクローラーのモデルの結合度を少なくする

- Location 
  - URLと付随情報を保存する
    - リクエストパラメータ
    - リファラー（同じクラス）
    - ページネーション情報
    - 次回更新日
  - 自身の URL に対応する Crawler クラスと Scraper クラスを推測する
  - 絶対パスで保存するようにパスからURL全体を復元したりもする
  - 設定したドメインしか保存できない

---
### URL はどこに登場するか？

- 
- 
- 


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

---
## 相対パス等の復元
### module URI 活用

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
      uri.host = Settings.hostname.unext.video_api
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
- ExclusiveLocation
  - 同一ドメイン内で、アクセスしないページを定義する
  - ヘルプページなど
- Crawler
  - Location を元に HTTP アクセスを行い、レスポンスを Crawling クラスに保存する
- Crawling
  - 日付ごとにレスポンスを保存する
    - http_status
    - header
    - body
    - クロール日付
  - 一部リダイレクト等で Location を作成する
- Scraper
  - Crawling を元に対象のドメインモデルのデータをもりもり作る
  - 成功/失敗を Scraping クラスに保存する
  - 同時に、検出した URL から Location の作成や次回更新日を更新する
- Scraping
  - Scraper の成功/失敗を持つ
  - Crawling ごとに成功するまでいくつでも

これらの ActiveRecord 継承クラス以外にも、Crawler / Scraper, CrawlerWorker/ScrapererWorker などが存在する(後述)

---
### Crawler

#### 

API かどうか

HTTP ヘッダーが


---
### 認証

- ブラウザ型は使わない

### 

---
### URI


---
### module URI

標準ライブラリ

注意


### 雰囲気

```rb
```

```rb
```

```rb
```

```rb
```


```
```

---
### ブラウザの URL が登場する箇所

---
### どのURLからクロールしても問題ない設計

- has_many & belongs_to の組み合わせの問題点
  - belongs_to は先に親のモデルのスクレイピングが終わっていないと使えない。
    - 親のいないレコードとか作らないですよね？
  - code 属性を持つモデルの has_many は has_many through を使う

- 中身はないが、 code 値のみ設定された ActiveRecord インスタンスが必要になることもある

- そのため、

特に、一度保存した値を 

たとえば、サイトの

次のスクレイピングでサイトのデザイン変更により、取得済み Item#name を空文字や `nil` で更新してしまうといったおそれがある。


 のような特別なコンテキストを作ったりしたが、最終的に煩雑でやめた。

```rb
class Item < ApplicationRecord
  validates :code, presence: true, format: { /ID\d+/ } ## 常に presence: true
  validates :name, length: { maximum: 100 } ## ここでは presence: true を指定しない

  with_options(on: :complete) do
    validates :name, presence: true
  end
end
```

スクレイピングの処理の最後で

```rb
item.save!(:complete) # => name が消えてしまっていたら raise する
```

とする。

=> `update!` メソッドにコンテキストを渡せないので、コードが地味に増えるを嫌ってやめた。
   - XPath の指定方法のノウハウが溜まってきたのもある。

before_save で `changes` を参照し、更新前後の値をチェックする DSL を作る
=> テーブルのカラムを調整する
設定が面倒になってやめる。
=> 結局 raise するのだから、

=> `audited` gem で変更履歴を持つことにした。

Vasily さんのように XPath をDBに保存するのもやめた。
一人で開発するには非常に書きにくい。

そのため、大胆に `crawling.run_date` で分岐することにした。
レイアウト変更は、数年に一度しか起こらない（ほんとか？）

---
### 起点となるページ
- サイトのホームページ '/'
- サイトマップページ
  - リニューアルでサイトマップページが消失したことがある
- `sitemap.xml` ( `robots.txt` を調べておく)

- その結果、その他の用途では `has_many` で問題ないものが、 has_many through 関連が
  - 数件のレコードしかできないものを ActiveRecord を使わずに `active_hash` で実装していたが、 active_hash は has_many through が使えないため、

---
### パンくずリスト

- パンくずリストの2タイプ
  - 階層構造
    - カテゴリー
  - 画面遷移順

気づき

- パンくずリストはM対Nの構造をしていることがある
  - 両端がポリモーフィック関連の `has_many through` が爆誕
- 恣意的に別のリンクになっていたりする
  - `<a href="/link">text</a>` タグの href と text が一致していなかったりする

---
## has_many / has_one

### has_one だと思っていたら、違った。

`table` タグ内の `tr` タグの数と実際にできた ActiveRecord の数を比較したら一致しない。

=> 元のデータに重複があった。

---
### HTTP スタータスの管理

Rails を使うと Rack が付いてくるので、 Rack::Utils をありがたく流用する。

---
### Rack::Utils::SYMBOL_TO_STATUS_CODE

HTTP ステータスの番号と内容を持ったハッシュ

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

  def redirect?
    http_status.in?(300..308)
  end
end
```

---
### Rack::Utils::SYMBOL_TO_STATUS_CODE

次のようなコードが書ける。

```rb
crawling = Crawling.last
crawling.ok? # => false
crawling.not_found? # => true
```

---
### html の保存

- 現代ではほぼ `UTF-8`

- それなりに大きい
- 必要な箇所だけ削ったりせず、まるごと保存する方針
- JavaScript/CSSは保存しない
- できれば圧縮したい
- 画像、動画、音声などのファイルについては、今回は割愛

---
### html の保存
- HTML/JSON を AWS S3 に保存することを検討した。
- S3 でそのままクロールした HTML がブラウザで確認できる。これは便利。

---
### AWS S3 に保存する問題点
- 自分の AWS のドメインのリファラーでターゲットサイトのファイルを取得するリクエストが飛んでしまうことに気づいた。まずそう。
- 管理するストレージが RDS (AWS のRDBMS サービス) 以外を増やしたくなかった。
  - サービス以外に、キー管理なども増える
- ActiveRecord 自身に HTML が入ってないのはデバッグなどでなにかと面倒そう。

---
### HTML を圧縮して保存したい

とりあえずテキストで ActiveRecord に保存することにした。
よくよく考えると、実はブラウザは圧縮されたHTMLを受け取っていることを思い出す。

---
### accept-encoding (リクエスト)/Content-Encoding(レスポンス)ヘッダー

リクエスト時に `accept-encoding: gzip, deflate, br` を付ける
レスポンス時に `content-encoding: gzip` で body を gzip で返ってくる
  (サーバーが対応していれば)

#### => gzip でレスポンスが返ってきたらそのままバイナリ保存、取り出すときに gzip を戻す
####    テキストでレスポンスが返ってきたら、 gzip 圧縮してバイナリで保存、取り出すときに gzip を戻す

---
### Vary ヘッダー

- 「これによってはレスポンスを変えるよ」というレスポンスヘッダー
- gzip の場合は、 `Vary: accept-encoding` というレスポンスヘッダーが返る。

---
## ActiveSupport::Gzip

- gzip 処理ができる Zlib ラッパー
  https://api.rubyonrails.org/classes/ActiveSupport/Gzip.html
- Rails の ActiveSupprt に付いてくる。

---
### ActiveRecordActiveRecord::AttributeMethods#[],
### ActiveRecordActiveRecord::AttributeMethods#[]=()

- ActiveRecord のアクセサを上書きするときに使える。
- ActiveRecord の DB からキャストされた属性が入っている。
- 今なら attributes API で置き換えられる？


---
## Migration

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
```rb
class Crawling < ApplicationRecord
  serialize :headers, JSON

  # NOTE: 汎用的に使うなら 小文字に揃える
  def gzip?
    headers['Vary'] == 'Accept-Encoding' &&
      headers['Content-Encoding'] == 'gzip'
  end

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
## ところがある日、gzip がうまくいっていない

- 同じ HTTP ヘッダーは複数返ってくることがある。

```
Vary: Accept-Encoding
Vary: User-Agent
```

- Rack の仕様では、配列になる。
- 型が String の場合と Array の場合がある。
- とても困る。

---
## Object クラスを拡張してしまう

Object#equal_or_include? が誕生

```config/initializers/object.rb
class Object
  def equal_or_include?(other)
    (self == other) || (respond_to?(:include?) && include?(other))
  end
end
```

---
## config/initializers/

- 標準ライブラリをアプリケーション独自に拡張する場合はここに置いておけばよい。
- やりすぎ注意(Objectクラス拡張しといてなんだが...)

---
```diff
class Crawling < ApplicationRecord
  serialize :headers, JSON

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

> nginxは設定ファイルにgzip_vary on;と書かないと
> Vary: Accept-Encoding をレスポンスヘッダに付加しません。
https://qiita.com/cubicdaiya/items/09c8f23891bfc07b14d3

- `Content-Encoding: gzip` が返ってきても `Vary: Accept-Encoding` も返すとは限らない。
- サーバーの設定が突然変わることもある。:cry:

---
## Object#equal_or_include? 消滅


`config/routes.rb` のようなルーターを思いつく
  => 即撤退


- ページデザイン変わるよね？

- `Crawing#run_date` からベタにロジック分岐
  - なにかのデザインパターンでリファクタリングするかもしれないが、そうそう起きるものではない。

HTMLを保存 -> rspec を書きながらロジック修正という、大変地味なことをしている

---
### Crawler

- HTML body が小さすぎる。


- エラーが出た。


---
### href には何が埋まっているか？

- リンク以外にもあるよ
  - `javascript:void(0)`
  - `tel:0120444444`
  - `mailto:user@example.com`

---
### URL として扱いたいもの

- "https://example.com/aaa/bbb.html"
- "/aaa/bbb.html"
- "//example.com/aaa/bbb.html"
- "bbb.html"
- "/"
- "#"
- "/#"
- ""

---
## 次はスクレイピング

- HTML/JSON/XML からほしいデータを保存する
- できるだけ正確なデータを取れるだけとりたい

---
## 最近は SPA も多い

- 1 つページが HTML と 1 つ以上の API のレスポンスで構成される

--- 
## 例

- ブラウザには HTML の URL が表示される
  - HTML 上で href に入っているのはこれ
- URL 中のコードに対応した API で JSON を取得して HTML に反映される

HTML URL `https://example.com/items/ID0001`
API URL  `https://example.com/api/items/ID0001`

## わかったこと

- ほとんどのサイトの API は サブドメインかパスの先頭が `api` で判別可能

サブドメイン型 `https://api.example.com/items/ID0001`
先頭パス型 `https://example.com/api/items/ID0001`

---
## 設計

- とりあえず HTML URL をクロール
- HTML URL のスクレイピング時に、 API の URL のみを作って保存
  - 他のことはやらない
- JSON API のクロール
- JSON API のスクレイピング時にドメインモデルを作って保存

---
## ブラウザ型のスクレイピングツールを使えば、
## HTML ページのみのクロールで済むのでは？

---
### API (JSON) を叩くとよい理由(1)
- DOM 要素を XPath や CSS セレクターや抽出するよりも JSON の方が正確な構造化がされている
  - HTML では並列にならんでいるけどデータモデルは子供

---
### API (JSON) を叩くとよい理由(2)
- より正確な値が手に入る
  - HTML 上では `分` と表示されているのが、実際は `秒` で取得できる

---
### API (JSON) を叩くとよい理由(3)
- メタデータがついていることもある
- タグやラベルの分類
- セールの期間
- リスト系 API の総件数

---
### API (JSON) を叩くとよい理由(4)
- 対象のサイトでのデータベースのテーブル名やカラム名に近い名前が手に入る

  例
    - HTML 上ではコード体系が同じ2階層の "MENU"
      `/MENU0001/MEMU0002` のような URL
      両方同じコード体型なので、何回層なのか、制限はあるのは不明
    - JSON を叩くと実際は "genre" と "category" の親子構造だったことがわかる
  - 過剰な実装を避けられる
  - 名前付けや保存時の属性判別に悩むことが大幅に減り、開発が加速する
  - 某作品にならって、真名と呼んでいる（余談）

---
### HTMLよりAPIの方が変わりにくく変更に強いか？
- 実際はそんなに期待できないという感想
  - サイトデザインががらっと変わると、同時にAPIが新設・廃止されるケースも多い。

---
### 力技

「SPA なのにどこの API も叩いてないぞ？」
  - HTML にはコンテンツの情報が見当たらない
  - Chrome Developer Tools の Network タブにリクエストが見当たらない

---
### 力技

`<script>` タグの中の JavaScript にコード辺を発見する。

```
<script>
   ...
    xxx.reactContext = {...}; // この JSON にページの内容が詰まっている
   ...
</script>
```

---
### 力技

JavaScript コード中の JSON を抽出

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

## Integer#chr

> 与えられたエンコーディング encoding において self を文字コードと見た時、それに対応する一文字からなる文字列を返します。 引数無しで呼ばれた場合は self を US-ASCII、ASCII-8BIT、デフォルト内部エンコーディングの順で優先的に解釈します。

## String#to_i

> 基数を指定することでデフォルトの 10 進以外に 2 〜 36 進数表現へ変換できます。 
https://docs.ruby-lang.org/ja/latest/method/String/i/to_i.html

## 悲報

### 数カ月後、この JavaScript コードが消滅し、 HTML から取得することになった。

### 真名が手に入っただけでもよしとする。

---



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
### 作ってみて気づいたコードの特徴

- ActiveRecord を始めとしたライブラリのAPIを呼び出す箇所は、ベタに書いてることが多い。
- わざとらしいほど説明的なメソッド名が多い
- DRY にできそうでできない

---
## Rails の仕様変更


---
## サイトマップのリンクに間違いがある

「このリンク間違ってる」
- 対応するコード書くのはめんどう

=> サポートに連絡する

詳細は割愛

> ※本メールの内容を無断転載・複製・引用のために使用することを禁止させていただきます。



---
### まとめ

- クローリングとスクレイピングの設計例を示した。
- Ruby と Rails はクローラーでの開発に使える。
