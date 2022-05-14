---
title: 基本的なSQL構文集
date: "2021-04-04T22:40:32.169Z"
description:
---

# 基本操作
## テーブルを作成

テーブルを作成するにはCREATE TABLE文を使う。

```
create table テーブル名 (カラム名 データ型 オプション, ・・・);
```

* テーブルを作成

```SQL:SQL
create table testtable (id integer primary key, name text not null unique, age integer);
```

```:実行結果
CREATE TABLE
```

## テーブルに新しい行を挿入

```
insert into テーブル名(カラム名、カラム名、カラム名)
values (値、値、値);
```
* １行挿入する

```SQL:SQL
insert into testtable(id, name, age)
values (1, 'Alice', 20);
```

```:実行結果
INSERT 0 1    ← 1行挿入されたことを表す
```

* 複数行挿入する

```SQL:SQL
insert into testtable(id, name, age)
values(1, 'Alice', 20)
     ,(2, 'Bob'  , 30)
     ,(3, 'Cathy', 40);
```

```:実行結果
INSERT 0 3    ← 3行挿入されたことを表す
```

## テーブルの行を検索

```
select カラム名 from テーブル名;
```

* テーブルに含まれる行をすべて表示

```:SQL
select *             ← 「*」はすべての列を表示
from testtable;      ← テーブルの行をすべて表示
```

```:実行結果
id   | name  | age 
-----+-------+----- 
1    | Alice | 20
2    | Bob   | 30
3    | Cathy | 40
(3 rows)
```

* 名前が「Bob」の行だけを検索、また名前と年齢の列だけを表示

```:SQL
select name, age       ← name列とage列だけを表示
from testtable
where name = 'Bob';    ← nameが「Bob」の行だけを表示
```

```:実行結果
id   | name  | age 
-----+-------+----- 
2    | Bob   | 30
(1 rows)
```

## テーブルの行を更新
update 文では条件を指定しなければすべての行が更新される。

* update文を使って、すべての行を更新する

```SQL:SQL
update testtable
set age = age + 1;
```

```:実行結果
UPDATE 3      ← 「3行更新された」という意味
```

* ある特定の行だけを更新する

```SQL:SQL
update testtable
set age = 27
where name = 'Bob';
```

```:実行結果
UPDATE 1      ← 「1行更新された」という意味
```

## テーブルから行を削除
 
* 条件に該当する行を削除する

```SQL:SQL
delete from testtable
where name = 'Bob';
```

```:実行結果
DELETE 1      ← 「1行削除した」という意味
```

* すべての行を削除する

```SQL:SQL
delete from testtable;
```

```:実行結果
DELETE 3
```

## テーブルを削除

```SQL:SQL
drop table testtable;
```

```:実行結果
DROP TABLE    ← 「テーブルが削除された」ことを表す
```

# SELECT文
## 使用するテーブル

```:members
id   | name     | height | gender
-----+----------+--------+--------
101  | エレン     | 170    | M
102  | ミカサ     | 170    | F
103  | アルミン   | 163    | M
104  | ジャン     | 175    | M
105  | サシャ     | 168    | F
106  | コニー     | 158    | M
```

## select句 ： 値の出力
何らかの値を出力するには、select句を使う。

```SQL:SQL
select 'Hello';
```

```:実行結果
?column?
----------
Hello
(1 row)
```

計算式の指定もできる。

```SQL:SQL
select 1 + 2 + 3 + 4;
```

```:実行結果
?column?
----------
10
(1 row)
```

## from句 ： テーブルを指定する
テーブルを指定してその中身（つまりテーブルに含まれている行）を出力するには、from句を使う。

・全ての行を検索する

```SQL:SQL
select * 
from members;
```

```:実行結果
id   | name     | height | gender
-----+----------+--------+--------
101  | エレン     | 170    | M
102  | ミカサ     | 170    | F
103  | アルミン   | 163    | M
104  | ジャン     | 175    | M
105  | サシャ     | 168    | F
106  | コニー     | 158    | M
```

・select句に列名を指定する

```SQL:SQL
select gender, name
from members;
```

```
gender  | name
--------+----------
M       | エレン
F       | ミカサ
M       | アルミン
M       | ジャン
F       | サシャ
M       | コニー
(6 rows)
```

## where句 ： 行を選択する
ある特定の行だけを選ぶ（あるいは除外する）には、where句で条件を指定する

```SQL:SQL
select *
from members
where name = 'ミカサ';
```

```:実行結果
id   | name     | height | gender
-----+----------+--------+--------
102  | ミカサ     | 170    | F
(1 rows)
```

andやorで条件をつなげることもできる。

```SQL:SQL
select * from members
where gender = 'M'
   or height >= 170;
```

```:実行結果
id   | name     | height | gender
-----+----------+--------+--------
101  | エレン     | 170    | M
102  | ミカサ     | 170    | F
103  | アルミン   | 163    | M
104  | ジャン     | 175    | M
106  | コニー     | 158    | M
(5 rows)
```

andでつなげた条件をさらにorでつなぐこともできる

```SQL:SQL
select * from members
where (gender = 'M' and height >= 170)
   or (gender = 'F' and height < 170);
```

```:実行結果
id   | name     | height | gender
-----+----------+--------+--------
101  | エレン     | 170    | M
104  | ジャン     | 175    | M
105  | サシャ     | 168    | F
(3 rows)
```

## order by句 ： 整列する
小さい順や大きい順に並び替えることを整列（Sort）という。
select文では、order by句を使って整列する。

```SQL:SQL
select *
from members
order by name;
```

```:実行結果
id   | name     | height | gender
-----+----------+--------+--------
103  | アルミン   | 163    | M
101  | エレン    | 170    | M
106  | コニー    | 158    | M
105  | サシャ    | 168    | F
104  | ジャン    | 175    | M
102  | ミカサ    | 170    | F
(6 rows)
```

順番を逆にしたい場合はdescをつける

```SQL:SQL
select *
from members
order by height desc;
```

```:実行結果
id   | name     | height | gender
-----+----------+--------+--------
104  | ジャン    | 175    | M
101  | エレン    | 170    | M
102  | ミカサ    | 170    | F
105  | サシャ    | 168    | F
103  | アルミン   | 163    | M
106  | コニー    | 158    | M
(6 rows)
```

order by句をつけないと順番は不定になる。
なるべくorder by句をつけるようにする。
同じ値のときは順番が定まらないので、ID順にするとよい。

## limit句とoffset句 ： 範囲を指定
先頭のn行だけ出力するにはlimit句を使う。
逆に、先頭からのn行は出力しない（スキップする）にはoffset句を使う。

先頭の 3 行をスキップしてから、続く 1 行だけを出力する。

```SQL:SQL
select * from members order by id
limit 1 offset 3;
```

```:実行結果
id   | name     | height | gender
-----+----------+--------+--------
104  | ジャン    | 175    | M
(1 rows)
```

## DBエンジン内部の実行順序
* select句はいちばん最後に実行
* それ以外は上から順に実行

※厳密には、select句はいちばん最後ではなく、order by句の前に実行される。
 基本は上記のように覚えておけば問題ない。

# グループ化と集約関数
## group by 句
group by 句を使うと、テーブルの内容をグループに分けて集約できる。

代表的な集約関数は以下。
集約関数は複数の値を受け取り、1つの値を返すような関数のこと。

・合計する(sum( ))
・平均する(avg( ))
・最大値(max( ))
・最小値(min( ))
・行数を数える(count( ))

```SQL:SQL
select gender, avg(height)
from members
group by gender;
```
## 桁数を指定する（to_char(x, y)）
avg(height)の結果が165.0000000000の時
to_char(avg(height), '999.99')で165.00にすることができる。

## nullを強制的に0に変換する(coalesce(x, y))
min(height)の結果がnullのとき、
coalesce(min(height), 0)で、nullを強制的に0に変換できる。

## having句
group by 句によって行をグループ化できる。
having句を使うと、それらのグループに対して選択条件を指定できる。

性別でグループ化してから、平均身長が170cm以上のグループだけを表示

```SQL:SQL
select gender, avg(height)
from members
group by gender
having avg(height) >= 170;
```

## where句とhaving句の違い
・where句はテーブルの行を対象とする。group by句の前に実行される。
・having句はグループを対象とする。group by句の後に実行される。

## よくある間違い
group by 句を使うと、select句に指定できるのは
「グループ化のキー」と「集約関数の呼び出し」だけ。
これ以外を指定すると、構文エラーになる。

## グループ化のキーに式を使う
名前の長さでグループ化し、それぞれの人数を数える

```SQL:SQL
select length(name), count(*)
from members
group by lenght(name);
```

列名は式の一種なので、当然group by 句にも式を指定できる。

## その他の集約関数
・ string_agg( )：文字列を連結する
・ array_agg( )：複数の値を配列にする
・ json_object_agg( )：JSON データを作る
・ json_agg( )：JSON 配列を作る

# 便利な機能
## コメント
・範囲コメント

```SQL:SQL
/* ←ここから範囲コメント　　　
　何書いても飛ばす　　
ここまでが範囲コメント→ */
```

・行コメント

```
--　←　ここから行末までが行コメント
select *  ← 行頭じゃなくても行コメント
from users;
```

## 列の別名
SQLでは列や式に別名(Alias)をつけることができる。

別名をつける理由は以下の通り。
（a）ヘッダのラベル名を変えるため
（b）長い列名や複雑な式に簡単な名前をつけて参照するため
（c）同じ名前を2回書かずに済ますため

```SQL:SQL
select id as ID, name as 名前, height as 身長
from members;
```

（asは省略可能）

```:実行結果
id  |  名前 | 身長
----+------+------
102 | ミカサ | 170
105 | サシャ | 168
(2 rows)

```

※実行結果のラベルは「ID」だけ変わらずに「id」のままになっている。
なぜかというと、SQLでは基本的に大文字と小文字を区別しない仕様なので、
「ID」と指定しても「id」と同じだと見なされるため。
この場合は「"ID"」のようにダブルクォーテーションで囲むとよい。

## テーブルの別名
SQLでは、テーブルにも別名（Alias）が付けられる。

```SQL:SQL
select m.id, m.name, m.height
from members m 
where m.gender = 'F'
order by m.id;

```

こうする理由は２つある。
（a）SQLのキーワードと同じ名前の列名を使うとき、キーワードでないことを明示するため。
（b）１つのSQLで複数のテーブルを扱う場合、どのテーブルの列名なのかを明示するため。

## 演算子 
### 文字列結合演算子

文字列を結合するには「||」という演算子を使います。

```SQL:SQL
select 'Hello, ' || 'World' || '!';
```

```:実行結果
?column?
---------------
Hello, World!
(1 row)
```

### パターンマッチ演算子
「like」という演算子を使うと、文字列がパターンにマッチしているかどうかを調べることができる。

「%」・・・どんな文字列にもマッチする（0文字でもよい）。
「_」・・・どんな1文字にもマッチする。

・「ン」で終わる名前だけを検索

```SQL:SQL
select name
from members
where name like '%ン';
```

## in演算子
「in」演算子は、複数の値を列挙して、その中に指定値が含まれているかどうかを調べる演算子。

```SQL:SQL
select *
from members
where name in ('エレン', 'ミカサ', 'アルミン')
order by id;
```

## null
nullとは値(データ)がないことを表すキーワード。

nullかどうかを調べるには、「is null」と「is not null」を使う。

```SQL:SQL
select name
from members
where height is null;
```
