---
title: 【Rails】ActiveRecordのよく使うメソッド集(モデル検索、テーブル結合など)
date: "2020-09-12T22:40:32.169Z"
description: 
---

ActiveRecordには、データベースからオブジェクトを取り出す
検索メソッドが多数用意されています。

検索メソッドはwhereやgroupといったコレクションを返したり、
ActiveRecord::Relationのインスタンスを返します。

またfindやfirstなどの１つのエンティティを検索するメソッドの場合、
そのモデルのインスタンスを返します。

# 単一のオブジェクトを取り出す
## find
主キーで検索

```
# 主キーが1のユーザーを検索

User.find(1)

# SELECT * FROM users WHERE (users.id = 1) LIMIT 1
```

findメソッドで複数のオブジェクトを取得

```
# 主キーが1と10のユーザーを検索

User.find(1,10) # User.find([1,10])も可

# SELECT * FROM users WHERE (users.id IN (1,10))
```

findメソッドでマッチするレコードが見つからない場合、
ActiveRecord::RecordNotFound例外が発生します。

## first
デフォルトでは主キー順の最初のレコードを取得

```
User.first

# SELECT * FROM users ORDER BY users.id ASC LIMIT 1
```

firstメソッドで返すレコードの最大数を指定して取得

```
User.first(3)

# SELECT * FROM users ORDER BY users.id ASC LIMIT 3
```

orderを使って順序を変更した場合、
firstメソッドはorderで指定された属性にしたがって最初のレコードを取得

```
User.order(:name).first

# SELECT * FROM users ORDER BY users.name ASC LIMIT 1
```

firstメソッドは、モデルにレコードが1つもない場合にnilを返します。
このとき例外は発生しません。

## last
デフォルトでは主キーの順序にしたがって最後のレコードを取得

```
User.last

# SELECT * FROM users ORDER BY users.id DESC LIMIT 1
```

lastメソッドは、モデルにレコードが1つもない場合にnilを返します。
このとき例外は発生しません。

## find_by
与えられた条件にマッチするレコードのうち、最初のレコードを取得

```
User.find_by(name: "hoge")

# SELECT * FROM users WHERE (users.name = "hoge") LIMIT 1
```
find_byメソッドは、モデルにレコードが1つもない場合にnilを返します。
このとき例外は発生しません。

# 条件
## where
返されるレコードを制限するための条件を指定します。
SQL文でいうWHEREの部分に相当します。

```
User.where(name: "hoge")

# SELECT * FROM users WHERE (users.name = "hoge")
```

・NOT条件
SQLのNOTクエリはwhere.notで表せます。

```
User.where.not(name: "hoge")

# SELECT * FROM users WHERE (users.name != "hoge")
```

・OR条件
1つ目のリレーションでorメソッドを呼び出し、そのメソッドの引数に
2つ目のリレーションを渡します。

```
User.where(name: "hoge").or(User.where(name: "foo"))

# SELECT * FROM users WHERE (users.name = "hoge") OR (users.name = "foo") 
```

# 並び順
データベースから取り出すレコードを特定の順序で並び替え

```
# ひとかたまりのレコードを取り出し、created_atの昇順で並び替え

User.order(:created_at)

# SELECT * FROM users ORDER BY users.created_at ASC
```

```
複数のフィールドを指定して並び替え

User.order(name: :asc, created_at: :desc)

# SELECT * FROM users ORDER BY users.name ASC, users.created_at DESC
```

# 取り出すレコード数の上限を指定
## limit
主キー順の最初から数えて5件のレコードを取得

```
User.limit(5)

# SELECT * FROM users LIMIT 5
```

# テーブルの結合
## joins
デフォルトは内部結合(INNER JOIN)
内部結合は「結合条件に一致するレコードのみを取得」
この場合だとpostsテーブルのuser_idと、usersテーブルのidを指定

```
# モデル.joins(:関連名)
User.joins(:posts)

# SELECT users.* FROM users INNER JOIN posts ON (posts.user_id = users.id) 
```

## left_joins
左外部結合(LEFT OUTER JOIN)
「結合条件に一致するレコード」と「左側のテーブルにしかないレコード」を取得

```
User.left_joins(:posts)

# SELECT users.* FROM users LEFT OUTER JOIN posts ON (posts.user_id = users.id) 
```

# 関連付けを一括読み込みする
## includes
一括読み込み(eager_loading)とは、Model.findによって返されるオブジェクトに関連付けられたレコードを読み込むためのメカニズムであり、できるだけクエリの回数を減らすようにします。

・N+1クエリ問題
以下のコードは、ユーザーを10人検索して郵便番号を表示します。
usersテーブルとaddressesテーブルの関係は1対1とします。

```ruby

users = User.limit(10)

users.each do |user|
  puts user.address.postcode
end

```
このコードには問題があります。実行されたクエリの数が無駄に多いことです。
最初にユーザーを10人検索するのにクエリを1回発行し、次にそこから住所を取り出すのに
クエリを10回発行するので、合計11回のクエリが発行されます。

・N+1クエリ問題を解決する
ActiveRecordは、読み込まれるすべての関連付けを事前に指定することができます。
これはincludesを指定することで実現できます。
includesを指定すると、ActiveRecordは指定されたすべての関連付けが最小限のクエリ回数で
読み込まれるようにしてくれます。

上の例でいうと、User.limit(10)を書き直して、住所が一括で読み込まれるようにします。

```ruby

users = User.includes(:address).limit(10)

users.each do |user|
  puts user.address.postcode
end

```

最初の例では11回もクエリが発行されましたが、今回の例では2回にまで減りました。

```

SELECT * FROM users LIMIT 10
SELECT addresses.* FROM addresses 
  WHERE (addresses.user_id IN (1,2,3,4,5,6,7,8,9,10))

```

# 参考
[Railsガイド](https://railsguides.jp/active_record_querying.html#%E3%82%B0%E3%83%AB%E3%83%BC%E3%83%97)


















