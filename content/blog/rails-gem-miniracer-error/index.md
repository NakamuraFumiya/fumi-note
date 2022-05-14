---
title: mini_racerのインストールが、依存関係にあるlibv8で失敗する
date: "2020-12-04T22:40:32.169Z"
description: 
---

## mini_racerのインストールで失敗する
Railsあるある
依存関係にあるlibv8のインストールで失敗しているみたい。
(mini_racerのgemアップデートも同じこと起きやすいです)

```
An error occurred while installing libv8 (8.4.255.0), and Bundler cannot continue.
Make sure that `gem install libv8 -v '8.4.255.0' --source 'https://rubygems.org/'` succeeds before bundling.

In Gemfile:
  mini_racer was resolved to 0.3.1, which depends on
    libv8

```

エラーログを見ていくと以下辺りが怪しそう。

```
Error: Command 'vpython third_party/depot_tools/update_depot_tools_toggle.py --disable' returned non-zero exit
```

```
could not resolve options: failed to resolve Python script: stat
```

issueも挙がっていた！
https://github.com/rubyjs/libv8/issues/310

## 試したことはbundlerのアップデート
issueを見る限り、bundlerのバージョンが〜〜と書いてあったので、
bundlerのバージョンを上げてみる。
自分が使っていたbundlerのバージョンは2.1.4だった。

```
% bundler -v
Bundler version 2.1.4
```

最新版のbundlerをインストールする。
（自分の場合は、他プロジェクトに合わせたかったので、最新版ではなくバージョン指定でインストールしました。基本的には最新のもので良いと思います）

```
% gem install bundler -v 2.2.8
Fetching bundler-2.2.8.gem
Successfully installed bundler-2.2.8
Parsing documentation for bundler-2.2.8
Installing ri documentation for bundler-2.2.8
Done installing documentation for bundler after 13 seconds
1 gem installed
```

インストールされたことを確認

```
gem list bundler

*** LOCAL GEMS ***

bundler (2.2.8, 2.1.4, default: 1.17.2)
```

デフォルトは最新のものになるみたい

```
% bundler -v
Bundler version 2.2.8
```
早速、mini_racerをアップデートしてみる

```
Using launchy 2.5.0
Installing libv8-node 16.10.0.0 with native extensions
Using xpath 3.2.0
Using rubocop-ast 1.14
```

インストールできた！
(コンパイラが動くので結構時間かかりました)

## 結論
自分の場合は、bundlerのバージョンが原因だった。

めちゃくちゃ苦しめられたエラーだったので、突破できてよかったです。
