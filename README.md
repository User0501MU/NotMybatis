・MySQLで管理者データの登・
mysql -u root -p
・データベースの作成
mysql> create database customer_management_db;
Query OK, 1 row affected (0.01 sec)

・作成したデータベースにアクセス
mysql> use customer_management_db;
Database changed

・管理者用のテーブルを作成
mysql> create table admin_tb (admin_id int(11) not null auto_increment,
    ->  name varchar(20) not null,
    -> password varchar(20) not null,
    -> primary key (admin_id)
    -> );
Query OK, 0 rows affected, 1 warning (0.03 sec)

・テーブルの定義を確認
mysql> desc admin_tb;
+----------+-------------+------+-----+---------+----------------+
| Field    | Type        | Null | Key | Default | Extra          |
+----------+-------------+------+-----+---------+----------------+
| admin_id | int         | NO   | PRI | NULL    | auto_increment |
| name     | varchar(20) | NO   |     | NULL    |                |
| password | varchar(20) | NO   |     | NULL    |                |
+----------+-------------+------+-----+---------+----------------+
3 rows in set (0.02 sec)

・管理者のデータを作成
mysql> insert into admin_tb(name , password) values("管理者1" , "11111111");
Query OK, 1 row affected (0.01 sec)

・作成した管理者のデータを確認
mysql> select * from admin_tb;
+----------+------------+----------+
| admin_id | name       | password |
+----------+------------+----------+
|        1 | 管理者1    | 11111111 |
+----------+------------+----------+
1 row in set (0.00 sec)

mysql>
～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～
・顧客テーブルの作成
$ mysql.server status
 ERROR! MySQL is not running
$ mysql.server start
Starting MySQL
.. SUCCESS! 
$ mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.36 Homebrew

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql>
・データベースの選・
mysql> show databases;
+------------------------+
| Database               |
+------------------------+
| information_schema     |
| customer_management_db |
| mysql                  |
| performance_schema     |
| sys                    |
+------------------------+
5 rows in set (0.01 sec)

mysql> use customer_management_db;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql>
・顧客テーブルの作
mysql> create table customer_tb (customer_id int(11) not null auto_increment,
    -> admin_id int(11) not null,
    -> name varchar(20) not null,
    -> address varchar(20), 
    -> registered_time datetime DEFAULT CURRENT_TIMESTAMP,
    -> updated_time datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    -> primary key (customer_id),
    -> foreign key (admin_id)
    -> references admin_tb(admin_id) 
    -> on delete cascade on update cascade)
    -> ENGINE=InnoDB  DEFAULT CHARSET=utf8
    -> ;
Query OK, 0 rows affected (0.17 sec)

mysql>

上記テーブル作成時に知っておくべきこと

登録日時と更新日時の自動設定
主キー
外部キー
外部キー制約
データベースエンジン（ストレージエンジン）

～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～
・テーブル定義の確認
mysql> desc customer_tb;
+-----------------+-------------+------+-----+-------------------+-----------------------------+
| Field           | Type        | Null | Key | Default           | Extra                       |
+-----------------+-------------+------+-----+-------------------+-----------------------------+
| customer_id     | int(11)     | NO   | PRI | NULL              | auto_increment              |
| admin_id        | int(11)     | NO   | MUL | NULL              |                             |
| name            | varchar(20) | NO   |     | NULL              |                             |
| address         | varchar(20) | YES  |     | NULL              |                             |
| registered_time | datetime    | YES  |     | CURRENT_TIMESTAMP |                             |
| updated_time    | datetime    | YES  |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
+-----------------+-------------+------+-----+-------------------+-----------------------------+
6 rows in set (0.03 sec)

mysql>
・顧客情報のデータを作成
mysql> insert into customer_tb(admin_id, name, address) values(1, "松藤春治郎", "北海道札幌市");
Query OK, 1 row affected (0.05 sec)

mysql>

・作成した顧客情報の確認
mysql> select * from customer_tb;
+-------------+----------+-----------------+--------------------+---------------------+---------------------+
| customer_id | admin_id | name            | address            | registered_time     | updated_time        |
+-------------+----------+-----------------+--------------------+---------------------+---------------------+
|           1 |        1 | 松藤春治郎      | 北海道札幌市       | 2022-12-09 01:47:03 | 2022-12-09 01:47:03 |
+-------------+----------+-----------------+--------------------+---------------------+---------------------+
1 row in set (0.01 sec)

mysql>

これでadmin_idが1である管理者の顧客情報が登録されました。

※今回は顧客情報を顧客一覧画面に表示させるために、直接データベースに登録しています。次回顧客登録画面にてブラウザからデータベースに顧客情報を登録する実装を行います。


ではadmin_idが1である管理者によって登録された顧客情報を全て閲覧するにはどういったSQLを実行すれば良いでしょうか？正解は以下です。

mysql> select * from customer_tb where admin_id = 1;
+-------------+----------+-----------------+--------------------+---------------------+---------------------+
| customer_id | admin_id | name            | address            | registered_time     | updated_time        |
+-------------+----------+-----------------+--------------------+---------------------+---------------------+
|           1 |        1 | 松藤春治郎      | 北海道札幌市       | 2022-12-09 01:47:03 | 2022-12-09 01:47:03 |
+-------------+----------+-----------------+--------------------+---------------------+---------------------+
1 row in set (0.09 sec)

mysql>
上記のSQLをJavaのプログラム上で実行すればログイン後に管理者が登録した顧客情報が一覧となって表示されるようになります。もちろん、管理者が複数人いた場合は管理者ごとで表示される顧客情報は異なります。


つまりログイン成功時にログインチェックのSQLと顧客情報を取得するSQLを実行する機能が必要です。ログインチェックの機能は前回作成しておりますので、今回は顧客情報を取得する機能を作成し、その結果を顧客情報一覧画面としてブラウザに表示させる実装を行っていきます。


その機能をLogin.javaに新たに追加していきます。


