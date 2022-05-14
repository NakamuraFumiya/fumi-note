---
title: RailsアプリにElasticsearchを組み込む
date: "2020-09-28T22:40:32.169Z"
description: 
---

# Elasticsearchとは
ElasticsearchとはElastic社が開発しているOSSの全文検索エンジンです。
大量のドキュメントから目的の単語を含むドキュメントを高速で抽出することができます。

# RailsアプリでElasticsearchを扱う考え方
1.全文検索エンジンに検索対象のデータが入っている
2.アプリケーション側で検索すると、検索エンジンに対してクエリを発行し、結果が返却される
3.アプリケーション側で検索対象のデータが更新されると、連携して検索エンジンのデータも更新される

# RailsアプリでElasticsearchを扱う
### インデックスを作成する
Elasticsearchでは、データの保存場所としてインデックスを作成します。
リレーショナル・データベースでいうところのテーブルのようなものです。

まずは、RailsのモデルをElasticsearchでも扱うために専用のgemをインストールします。
以下をGemfileに記述し、bundle installします。

```
gem 'elasticsearch-model', github: 'elastic/elasticsearch-rails'
gem 'elasticsearch-rails', github: 'elastic/elasticsearch-rails'
```

bundle installが終わったら、Elasticsearchにインデックスを作成します。
検索対象としたいモデルの中で、Elasticsearch::Modelをincludeします。

```ruby
class Article < ActiveRecord::Base
  include Elasticsearch::Model
end
```

これでモデルでElasticsearchを扱う準備ができました。
インデックスは以下のようなコードで作成できます。

```ruby
Article.__elasticsearch__.create_index! force:true
```

### インデックスにドキュメントを入れる
Elasticsearchでは、インデックスに入っているデータのことをドキュメントと呼びます。
インデックスには検索対象にしたいデータを入れます。

以下のコードでElasticsearchにドキュメントをインポートします。

```ruby

Article.import
```

これでElasticsearchのインデックスの中に、ドキュメントが登録されます。

## ドキュメントを検索する

ドキュメントを検索するには、Elasticsearchにクエリを投げます。
以下のように書くことで、RailsからElasticsearchにクエリを投げることができます。

```ruby
response = Article.search 'hoge'
```

引数で検索文字列を指定することで、ドキュメントを検索することができます。

フロントからパラメータを受け取って検索すると以下のように書けます。

```ruby
def index
  @articles = Article.search(params)
end
```

## Rails側で検索対象のレコードが更新されると、それに伴ってElasticsearchのドキュメントも更新する

実際にサービスを運用するとなると、Rails側でレコードが更新されるとElasticsearchのドキュメントを更新する必要があります。

まず単にElasticsearchのドキュメントを更新するには、以下のように実装します。

```ruby
Article.first.__elasticsearch__.update_document
```

他にも、delete_documentというメソッドがあるので、これを使えばドキュメントの削除もできます。

上記のように明示的に書かなくても、レコードを更新した際に自動的にドキュメントを更新することもできます。
gemであるelasticsearch-modelでは、Elasticsearch::Model::CallbacksをModelにincludeしておくと、レコードを更新した際にElasticsearchのドキュメントを更新するクエリを投げてくれます。

```ruby
class Article
  include Elasticsearch::Model
  include Elasticsearch::Model::Callbacks
end
```

## RailsアプリにElasticsearchを組み込む

実際にArticleモデルの検索周りの処理を作ります。

```ruby:article-m/app/models/concerns/article/searchable.rb
require 'active_support/concern'
module Article::Searchable
  extend ActiveSupport::Concern

  included do
    include Elasticsearch::Model
    
    index_name "article"

    settings index: {
      number_of_shards: 1,
      number_of_replicas: 0
    } do
      mapping _source: { enabled: true } do
        indexes :id, type: 'integer', index: 'not_analyzed'
        indexes :title, type: 'string'
        indexes :content, type: 'text'
      end
    end
  end

  module ClassMethods
    def create_index!(options={})
      client = __elasticsearch__.client
      client.indices.delete index: "article" rescue nil if options[:force]
      client.indices.create index: "article",
      body: {
        settings: settings.to_hash,
        mappings: mappings.to_hash
      }
    end
  end
end
```

モジュールの中で、include Elasticsearch::Modelして便利なメソッド群を使えるようにします。

index_nameはインデックス名、settingsにはインデックスの設定を書きます。
number_of_shardsやnumber_of_replicasはシャードやレプリカの設定で、耐障害性や性能に関連します。

mappingはインデックスをどのように定義するか決めます。RDBでいうテーブルスキーマのようなものです。

create_index!は実際にインデックスを作成するヘルパーです。
__elasticsearch__.clientでElasticsearchのクライアントのオブジェクトが取れるので、
このクライアント経由でいろいろ操作できます。

作ったモジュールをモデルにincludeします。

```ruby:article-m/app/models/article.rb
class Article < ActiveRecord::Base
  include Article::Searchable
  def self.search_message(keyword)
    if keyword.present?
      query = {
        "query": {
          "match": {
            "message": keyword
          }
        }
      }
      Article.__elasticsearch__.search(query)
    else
      Article.none
    end
  end
end
```
もらったキーワードから検索のクエリを組み立てて、Article.__elasticsearch__.searchに渡します。
Articleモデルに対して__elasticsearch__.searchを呼び出すことで、elasticsearch-railsとelasticsearch-modelがクエリを投げてくれます。

コントローラーは以下のようになります。

```ruby:article-m/app/controllers/articles_controller.rb
class ArticlesController < ApplicationController
  def search
    @keyword = params[:keyword]
    @articles = Article.search_message(@keyword).paginate(page: params[:page])
  end
```

以上がRailsアプリにElasticsearchを組み込む一例です。

## 参考
[elastisearch-railsを使ってRailsでElasticsearchを動かす【初心者向け】](https://qiita.com/katsuhisa__/items/264f0c0c2085e6c27bd2)
[Railsアプリの検索処理にElasticsearchを組み込むのにやったことまとめ](https://qiita.com/minamijoyo/items/31118d4aa3d06513ad4d#elasticsearch%E3%82%92rails%E3%82%A2%E3%83%97%E3%83%AA%E3%81%AB%E7%B5%84%E3%81%BF%E8%BE%BC%E3%82%80)






