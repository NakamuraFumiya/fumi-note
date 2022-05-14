---
title: サブクエリについて
date: "2021-04-04T22:40:32.169Z"
description:
---

# サブクエリとは
サブクエリ（副問い合わせ）とは、SQLの中に別のSQLを埋め込んで使う機能のことです。

# サンプルデータ
・「students」テーブル・・・生徒の一覧を表します。
・「test_scores」テーブル・・・テストの点数を表します。科目は「国語」、「算数」、「理科」、「社会」の４教科。

・studentsテーブル

id   | name            | gender | class
-----+-----------------+--------+-------
201  | さくら ももこ       | F      | 3-4
202  | はなわ かずひこ    | M      | 3-4
203  | ほなみ たまえ      | F       | 3-4
204  | まるお すえお      | M      | 3-4

・test_scoresテーブル

student_id | subject | score
------------+---------+-------
201 | 国語 | 60
201 | 算数 | 40
201 | 理科 | 40
201 | 社会 | 50
202 | 国語 | 60
202 | 算数 | 70
202 | 理科 | 50
202 | 社会 | 70
203 | 国語 | 80
203 | 算数 | 80
203 | 理科 | 70
203 | 社会 | 100
204 | 国語 | 80
204 | 算数 | 90
204 | 理科 | 100
204 | 社会 | 100

# サブクエリが返す結果
サブクエリ（つまり埋め込むSQL）がどんな結果を返すのかによって、サブクエリの使い方が決まります。
具体的には、SQL の結果が単一列なのか複数列なのか、また単一行なのか複数行なのかで4種類に分類できます。

## 単一列単一行のサブクエリ

・実行結果が単一列単一行のSQL

```SQL:国語の最高得点を検索するSQL
select max(score)
from test_scores
where subject = '国語'
```

```:実行結果（1列1行）
max
-----
80    ← 結果が単一列単一行（1列1行）
(1 row)
```

・単一列単一行のサブクエリ
サブクエリの返す結果を単一値として扱うことができます。

```SQL:国語で最高点をとった生徒のIDと点数を検索するSQL
select t.student_id, t.score
from student t
where t.subject = '国語'
  and t.score = ( select max(score) from test_scores where subject = '国語')
order by t.student_id; 
```

```:実行結果
student_id | score
-----------+-------
       203 |    80
       204 |    80
(2 rows)

```

ちなみに1行も返さないSQLをサブクエリに使うとエラーにはならず、結果が空になります。

## 複数列単一行のサブクエリ

・実行結果が複数列単一行のSQL

```SQL:国語の最高点を検索する
select subject, max(score)
from test_scores
where subject = '国語'
group by subject;
```

```:実行結果(2列1行)
subject | max
--------+-----
   国語 | 80     ← 結果が複数列単一行（2列1行）
(1 row)
```

・複数列単一行のサブクエリ
サブクエリの返す結果を複合値として扱うことができます。

```SQL:国語の最高点をとった生徒のIDと点数を表示するSQL
select t.student_id, t.score
from test_scores
where (t.subject, t.score) = (select subject, max(score) 
                              from test_scores
                              where subject = '国語' 
                              group by subject)
order by student_id;
```


```:実行結果
student_id | score
-----------+-------
       203 | 80
       204 | 80
(2 rows)
```

## 単一列複数行のサブクエリ

・実行結果が単一列複数行のSQL

```SQL:どれかの教科で100点満点をとった生徒のIDをすべて表示するSQL
select student_id
from test_scores
where score = 100;
```

```実行結果:1列3行
student_id
------------
       203    ← 結果が単一列複数行（1列3行）
       204
       204
(3 rows)
```

・単一列複数行のサブクエリ

```SQL:どれかの教科で100点満点をとった生徒をすべて表示
select s.*
from students s
where s.id in (
  select student_id
  from test_scores
  where score = 100
)
order by s.id;
```

```:実行結果
id  | name           | gender | class
-----+---------------+--------+-------
203 |     ほなみ たまえ |      F | 3-4
204 |     まるお すえお |      M | 3-4
(2 rows)
```

## 複数列複数行のサブクエリ

・実行結果が複数列複数行のSQL

```SQL:教科別の平均点を表示する
select subject, avg(score) as avg_score
from test_scores
group by subject;
```

```:実行結果
subject  | avg_score
---------+---------------------
    理科 | 65.0000000000000000
    国語 | 70.0000000000000000
    社会 | 80.0000000000000000
    算数 | 70.0000000000000000
(4 rows)
```

・複数列複数行のサブクエリ
テーブル代わりのサブクエリとして扱うことができます。

```SQL:平均点が70点に満たない教科とその平均点を検索する
select x.subject, x.avg_score
from (
  select subject, avg(score) as avg_score
  from test_scores
  group by subject
) x
where 70 > x.avg_score;
```

```:実行結果
subject  | avg_score
---------+---------------------
     理科 | 65.0000000000000000
(1 row)
```

# 参考
[わかりみSQL](https://booth.pm/ja/items/1576397)
