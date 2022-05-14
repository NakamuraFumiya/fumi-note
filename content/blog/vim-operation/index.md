---
title: Vim基本操作
date: "2022-01-23T22:40:32.169Z"
description: 
---

# はじめに
サービスを運用していく中で障害が発生した時は
ログを確認することで原因調査に役立てることができるのですが
自分の場合、Vimの使い方がままならずログを追うのに苦戦していたので
今回改めて、ログを追うのに必要となる基本操作をピックアップして整理してみました。

他にもVimを覚えたらファイルの編集だったり、OSに依存せずコードを書けたりと
覚えておくと便利らしいです。


# 「Vi」と「Vim」の違い
「Vi」＝「Vim」で問題ないそうです。
Vimは2000年以降で普及し始めたviの進化版だそうです。
（vimは「Visual editor IMproved」の略で、viの改良版という意味）

# Vimコマンド
## vimの起動

|コマンド名|説明|
----+-----
|vim filename|ファイルを開くまたは新規作成する|
|view filename|読み取り専用でファイルを開く|

## カーソル移動

|コマンド名|説明|
----+-----
|h|左に移動|
|j|下の行の先頭文字 (空白ではない) に移動 |
|k|上の行の先頭文字 (空白ではない) に移動 |
|l|右に移動|
|Returnキー|下の行の先頭文字 (空白ではない) に移動 |
|Sfift + g|ファイル内の最終行に移動する|
|Ctrl-F|1 画面先のページを表示 |
|Ctrl-D|半画面先にスクロール  |
|Ctrl-B|1 画面前のページを表示 |
|Ctrl-U|半画面前にスクロール |


## 検索

|コマンド名|説明|
----+-----
|/string|文字列を検索|
|?string|文字列を逆方向に検索|
|n|検索方向の前方にある文字列を検索|
|N|検索方向の後方にある文字列を検索|

## 挿入

|コマンド名|説明|
----+-----
|i|インサートモードになる。次から打つ文字が挿入されていく。終えるときはESCキー|

## 削除
|コマンド名|説明|
----+-----
|x|カーソルのある場所の文字を削除する|
|dd|カーソルのある行を削除する|

## コピーペースト
|コマンド名|説明|
----+-----
|yy|行をバッファにコピーする|
|p|バッファの内容を貼り付ける|

## 変更を戻す
|コマンド名|説明|
----+-----
|u|変更を元に戻す|


## 保存と終了

|コマンド名|説明|
----+-----
|:w Returnキー|変更を保存|
|:q Returnキー|vimの終了(変更していた場合、:wなどで保存しないと終了できない)|
|:wq Returnキー|変更を保存してvimを終了|
|:q! Returnキー|変更を保存しないでvimを終了|

## その他
|コマンド名|説明|
----+-----
|esc|（様々なモードを）キャンセルする。|



# 最後に
Vimのことを少しだけ知れました。
実際にコマンドを動かしながら学習すると楽しいです。

