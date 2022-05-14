---
title: 【Rails】scopeを使ってクエリメソッドを共通化する
date: "2020-08-27T22:40:32.169Z"
description: 
---

モデルのscopeについて勉強する機会があったので、紹介させていただきます！

Scopeとは、モデルへのメソッド呼び出しとして参照されるクエリに、
名前を付けてひとまとまりにしたものです。

## Scopeを定義する

例えば、あるBookのidを降順にし、かつ10件表示したいとします。
その場合、以下のようなメソッドを使います。

```ruby
Book.order(id: :desc).limit(10)
```

他にも条件を追加したい時、もっとメソッドを連結させる必要があり、コントローラーの記述量が増えます。
また、あまりにも長いと一見何の処理を行っているか分かりにくくなるかもしれません。

そこでscopeの出番です。

scopeを使うと以下のように書けます。

```ruby
class Book < ActiveRecord::Base
  scope :recent, -> { order(id: :desc).limit(10) }
end
```

このように定義しておけば、コントローラーで「Book.recent」という記述が可能になります。

```ruby
class BooksController < ApplicationController
  def index
    @books = Book.recent
  end
end
```

scopeは再利用ができるので、コントローラーで毎回クエリメソッドを連結させる必要がなく、
コードがすっきりするので読みやすくなります。

また、scopeは重ねて呼び出すことも可能です。

モデルにscopeを追加して

```ruby
class Book < ActiveRecord::Base
  scope :recent, -> { order(id: :desc).limit(10) }
  scope :costly, -> { where("price > ?", 2000) }
end
```
コントローラーで「Book.costly.recent」という記述が可能になります。

```ruby
class BooksController < ApplicationController
  def index
    @books = Book.costly.recent
  end
end
```

## 引数を渡す
スコープには引数を渡すことができます。

```ruby
class Book < ActiveRecord::Base
  scope :created_before -> (time) { where(" created_at < ?", time) }
end
```

引数付きのscopeは、クラスメソッドと同様の方法で呼び出すことができます。

```ruby
Book.created_before(Time.zone.now)
```

しかし、scopeに引数を渡す方法は、クラスメソッドとして定義する方法でも同じことができます。

```ruby
class Book < ActiveRecord::Base
  def self.created_before(time)
    where(" created_at < ?", time)
  end
end
```

ちなみにscopeに引数を渡すよりも、クラスメソッドで定義する方が推奨されています。

## 結局scopeを使うと何が良いのか

**１. クエリに名前を付けられるので、直感的なコードになる**

例えば{ where("price > ?", 2000) }に、costlyという名前をつけると簡潔になります。


**２. 修正箇所を限定できる**

scopeを再利用している場合、scopeの定義自体を修正するだけで済みます。


**３. コードの記述量が減る**

scopeを使った場合、同じ処理でメソッドを何回も連結させる必要がないので、コードがすっきりします。

## 最後に

コードはただ動けば終わりではなく、DRY原則に沿って保守しやすいコードを書けるように意識していきたいです。

そのためにもscopeなど知る必要があるので、リファレンスや技術書を積極的に活用していきたいなと思いました！

以上です！


