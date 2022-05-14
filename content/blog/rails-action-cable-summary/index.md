---
title: 【Rails】Action Cableについて
date: "2020-09-19T22:40:32.169Z"
description: 
---

## Action Cableが誕生した背景
ユーザーの入力なしに、最新の情報をWebに表示する「Realtime性」をもつリッチな体験を実現するために誕生しました。

Action Cableは、RailsのRESTとWebSocketのシームレスな統合です。
例えばAction Cableを使うことで、みなさんが使っているようなリアルタイムチャットアプリを
作成することもできます。 

<!-- ![スクリーンショット 2020-09-19 14.53.39.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/564392/05a5fc8e-92ea-00d0-6847-cc3bff1413e3.png) -->

## 用語について

### Websocket
・Web上でクライアント、サーバ間の双方向通信を実現するための通信規格
・送信できるデータはテキストまたはバイナリ
・HTTPとは別のプロトコルなのでCookieは送られない
・通信が確立した後はクライアント、サーバーどちらからでもメッセージを送ることが可能

### Connection
・Websocketとしてのコネクションを表すクラス
・Websocketとして通信が確立した時点でインスタンスが生成される
・開始時のリクエストはふつうのHTTPなのでCookieも送られる
・認証処理はこのクラス内に記述

### Consumer
・Websocketコネクションのクライアント（ブラウザなど）
・同じブラウザの異なるタブは完全に別のクライアントとして扱う

## サーバー側(Rails)の構成要素
### Conection
コネクションは、ApplicationCable::Connectionのインスタンスです。
このクラスでは、受信したコネクションを承認し、ユーザーを特定できた場合にコネクションを確立します。
そのための仕組みとして、identified_byを使います。
identified_byはコネクションIDであり、後で特定のコネクションを見つけるときに利用できます。

```ruby
# app/channels/application_cable/connection.rb
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :current_user

    def connect
      self.current_user = find_verified_user
    end

    private
      def find_verified_user
        if verified_user = User.find_by(id: cookies.encrypted[:user_id])
          verified_user
        else
          reject_unauthorized_connection
        end
      end
  end
end

```

### Channel
チャネルはMVCでいうコントローラーの役割を担います。

### Subscription
コンシューマーは、チャネルを購読するサブスクライバ側(Subscriber)の役割を果たします。
そして、コンシューマーのコネクションはサブスクリプション（Subscription: 購読）と呼ばれます。

```ruby
# app/channels/chat_channel.rb
class ChatChannel < ApplicationCable::Channel
  # コンシューマーがこのチャネルのサブスクライバ側になると
  # このコードが呼び出される
  def subscribed
  end
end
```

## 例1. ユーザーアピアランスの表示
これはユーザーがオンラインかどうかという情報を追跡するチャネルの例が以下です（オンラインユーザーの横に緑の点を表示する機能を作成する場合などに便利）。

まずは、サーバー側のアピアランスチャンネルを作成します。

```ruby
# app/channels/appearance_channel.rb
class AppearanceChannel < ApplicationCable::Channel
  def subscribed
    current_user.appear
  end

  def unsubscribed
    current_user.disappear
  end

  def appear(data)
    current_user.appear(on: data['appearing_on'])
  end

  def away
    current_user.away
  end
end
```

サブスクリプションが開始されると、subscribedコールバックがトリガーされ、
そのユーザーがオンラインであることを示します。

## 例2. 新しいWeb通知を受信する
Websocketコネクションを使って、サーバーからクライアント側の機能をリモート実行させます。

このWeb通知チャネルは、正しいストリームにブロードキャストを行ったときに、クライアント側でWeb通知を表示します。

サーバー側のWeb通知チャネルは以下です。

```ruby
# app/channels/web_notifications_channel.rb
class WebNotificationsChannel < ApplicationCable::Channel
  def subscribed
    stream_for current_user
  end
end
```

アプリケーションのどこからでも、web通知チャネルのインスタンスにコンテンツをブロードキャストできます。

```ruby
# このコードはアプリケーションのどこか（NewCommentJob あたり）で呼び出される
WebNotificationsChannel.broadcast_to(
  current_user,
  title: 'New things!',
  body: 'All the news fit to print'
)
```

## 参考
[Railsガイド](https://railsguides.jp/action_cable_overview.html#%E3%83%AF%E3%83%BC%E3%82%AB%E3%83%BC%E3%83%97%E3%83%BC%E3%83%AB%E3%81%AE%E8%A8%AD%E5%AE%9A)
[描いて理解する Action Cable]
(https://qiita.com/ksugimori/items/d5f4630858eeb56d9da5)



