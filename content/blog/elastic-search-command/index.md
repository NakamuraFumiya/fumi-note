---
title: Elasticsearchのよく使うコマンド集
date: "2020-10-07T22:40:32.169Z"
description: 
---

備忘録としてまとめます。

# よく使うコマンド
#### インデックス一覧の確認
インデックス名や、インデックスの状態を確認することができる。

```
curl -XGET 'localhost:9200/_cat/indices?v'
```

#### インデックスのマッピングの確認

```
curl http://localhost:9200/{indexname}/_mapping?pretty
```

#### ドキュメント一覧
インデックスに入っているドキュメントの一覧を確認することができる。

```
curl -XGET 'localhost:9200/{index_name}/_search?pretty'
```

#### ドキュメント１件取得
インデックスにちゃんとドキュメントが入ったか確認することができる。

```
curl http://localhost:9200/{index_name}/_search?pretty -H "Content-type: application/json" -d '{"size":1}'
```

#### ドキュメントの件数確認

```
curl -sS -XGET 'localhost:9200/{indexName}/_count?pretty'
```

#### エイリアス確認

```
curl -XGET 'localhost:9200/_aliases?pretty'
```

## 参考
https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html

https://blog.kozakana.net/2019/10/elasticsearch-confirmation-commands/

