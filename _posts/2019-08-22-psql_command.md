---
layout:     post
title:      psql 
subtitle:   psql 
date:       2019-08-22
author:     zxll
catalog: true
tags:
    - linux
    - apache
    - mysql
    - fuelphp
    - psql 
---

psql 常用命令

[详细用法可以参考：https://www.yiibai.com/html/postgresql/2013/080332.html](https://www.yiibai.com/html/postgresql/2013/080332.html)

#### 连接数据库

连接数据库, 默认的用户和数据库是postgres
`psql -U user -d dbname -h ** -p 5432`

登录公司的数据库

```
psql -U tana -d goo_all -h new-db.goo-net.com -p 5432
/usr/local/pgsql/bin/psql -U tana -d goo_all -h new-db.goo-net.com -p 5432
/usr/local/pgsql/bin/psql -U tana -d shinsha -h new-db.goo-net.com -p 5432
# 不像前两个命令登录db后，在执行命令，办法是 "- c"
/usr/local/pgsql/bin/psql -U tana -d goo_all -h new-db.goo-net.com -p 5432 -c "SELECT brand_cd FROM t_article_cars GROUP BY brand_cd ORDER BY brand_cd;"
```

#### offset 作用

> select * from persons limit  A  offset  B; 
>
> select * from persons limit 5 offset 0 ;
>
> 意思是，起点0开始查询，返回5条数据。

```
goo_all=# select owner_cd,goo_car_id,car_cd,absyear,pre_c from zaiko limit 5;
 owner_cd |      goo_car_id       |  car_cd  | absyear | pre_c
----------+-----------------------+----------+---------+-------
 0302527  | 0170130601H4001033005 | 10552003 | 2007年  |     1
 0300337  | 0170130601H4000114007 | 10153015 | 2001年  |     1
 0302600  | 0170130601H4002668003 | 10604003 | 1995年  |     1
 0300413  | 0170130601H4000375031 | 10501004 | 2012年  |     1
 0103955  | 700010395530190515035 |          | 2017年  |     1
(5 rows)

goo_all=# select owner_cd,goo_car_id,car_cd,absyear,pre_c from zaiko limit 2 offset 2;
 owner_cd |      goo_car_id       |  car_cd  | absyear | pre_c
----------+-----------------------+----------+---------+-------
 0302600  | 0170130601H4002668003 | 10604003 | 1995年  |     1
 0300413  | 0170130601H4000375031 | 10501004 | 2012年  |     1
(2 rows)
```



#### SQL 执行时间分析

```
goo_all=# EXPLAIN  SELECT count(*) FROM zaiko;
                                   QUERY PLAN
---------------------------------------------------------------------------------
 Aggregate  (cost=9295.95..9295.96 rows=1 width=0)
   ->  Append  (cost=0.00..9158.76 rows=54877 width=0)
         ->  Seq Scan on zaiko  (cost=0.00..0.00 rows=1 width=0)
         ->  Seq Scan on zaiko_10 zaiko  (cost=0.00..208.39 rows=1139 width=0)
         ->  Seq Scan on zaiko_11 zaiko  (cost=0.00..466.65 rows=2565 width=0)
         ->  Seq Scan on zaiko_12 zaiko  (cost=0.00..393.81 rows=2081 width=0)
         ->  Seq Scan on zaiko_13 zaiko  (cost=0.00..4148.75 rows=26275 width=0)
         ->  Seq Scan on zaiko_14 zaiko  (cost=0.00..224.55 rows=1155 width=0)
         ->  Seq Scan on zaiko_15 zaiko  (cost=0.00..2439.83 rows=14683 width=0)
         ->  Seq Scan on zaiko_16 zaiko  (cost=0.00..274.15 rows=1415 width=0)
         ->  Seq Scan on zaiko_17 zaiko  (cost=0.00..251.37 rows=1237 width=0)
         ->  Seq Scan on zaiko_18 zaiko  (cost=0.00..117.38 rows=638 width=0)
         ->  Seq Scan on zaiko_19 zaiko  (cost=0.00..633.88 rows=3688 width=0)
(13 rows)
```

```
goo_all=# EXPLAIN (analyze,buffers) SELECT count(*) FROM zaiko;
                                                           QUERY PLAN
--------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=9295.95..9295.96 rows=1 width=0) (actual time=138.365..138.366 rows=1 loops=1)
   Buffers: shared hit=8610
   ->  Append  (cost=0.00..9158.76 rows=54877 width=0) (actual time=0.013..113.782 rows=54880 loops=1)
         Buffers: shared hit=8610
         ->  Seq Scan on zaiko  (cost=0.00..0.00 rows=1 width=0) (actual time=0.002..0.002 rows=0 loops=1)
         ->  Seq Scan on zaiko_10 zaiko  (cost=0.00..208.39 rows=1139 width=0) (actual time=0.009..1.701 rows=1139 loops=1)
               Buffers: shared hit=197
         ->  Seq Scan on zaiko_11 zaiko  (cost=0.00..466.65 rows=2565 width=0) (actual time=0.012..3.602 rows=2567 loops=1)
               Buffers: shared hit=441
         ->  Seq Scan on zaiko_12 zaiko  (cost=0.00..393.81 rows=2081 width=0) (actual time=0.005..2.939 rows=2081 loops=1)
               Buffers: shared hit=373
         ->  Seq Scan on zaiko_13 zaiko  (cost=0.00..4148.75 rows=26275 width=0) (actual time=0.010..31.873 rows=26277 loops=1)
               Buffers: shared hit=3886
         ->  Seq Scan on zaiko_14 zaiko  (cost=0.00..224.55 rows=1155 width=0) (actual time=0.006..1.508 rows=1155 loops=1)
               Buffers: shared hit=213
         ->  Seq Scan on zaiko_15 zaiko  (cost=0.00..2439.83 rows=14683 width=0) (actual time=0.011..17.283 rows=14675 loops=1)
               Buffers: shared hit=2293
         ->  Seq Scan on zaiko_16 zaiko  (cost=0.00..274.15 rows=1415 width=0) (actual time=0.006..1.762 rows=1423 loops=1)
               Buffers: shared hit=260
         ->  Seq Scan on zaiko_17 zaiko  (cost=0.00..251.37 rows=1237 width=0) (actual time=0.010..1.631 rows=1237 loops=1)
               Buffers: shared hit=239
         ->  Seq Scan on zaiko_18 zaiko  (cost=0.00..117.38 rows=638 width=0) (actual time=0.006..0.760 rows=638 loops=1)
               Buffers: shared hit=111
         ->  Seq Scan on zaiko_19 zaiko  (cost=0.00..633.88 rows=3688 width=0) (actual time=0.005..4.438 rows=3688 loops=1)
               Buffers: shared hit=597
 Total runtime: 138.487 ms
(26 rows)
```

#### db手动停止和启动

```
su - postgres
/usr/local/pgsql/bin/postgres -D /usr/local/psql/data >logfile 2>&1 &
/usr/local/pgsql/bin/pg_ctl stop -D /usr/local/psql/data
```

#### 常用小命令

| 命令                 | 意义                                                    |
| -------------------- | ------------------------------------------------------- |
| \x                   | 格式化输出                                              |
| \c dbname            | 切换数据库,相当于mysql的use dbname                      |
| \l                   | 列举数据库，相当于mysql的show databases                 |
| \dt                  | 列举表，相当于mysql的show tables                        |
| \d tblname           | 查看表结构，相当于desc tblname,show columns from tbname |
| \d+ tblname          | 查看表结构，带有字段描述信息                            |
| \di                  | 查看索引                                                |
| \s                   | 历史命令                                                |
| \du                  | 查看用户                                                |
| \dv                  | 查看视图                                                |
| \df                  | 查看函数                                                |
| \df+                 | 查看函数具体定义信息                                    |
| \sf+                 | 函数名: 查看函数的创建语句                              |
| \dy                  | 查看触发器                                              |
| \dx                  | 查看添加的PostgreSQL扩展模块                            |
| \dp viewortable      | 查看表或视图的权限                                      |
| \ev 和 \sv           | 编辑和显示视图 psql9.6版本                              |
| select version();    | 查看版本详细信息                                        |
| show server_version; | 查看版本信息                                            |

#### 常用db操作语句

| 命令                                                         | 意义                                           |
| ------------------------------------------------------------ | ---------------------------------------------- |
| create database [数据库名];                                  | 创建数据库：                                   |
| drop database [数据库名];                                    | 删除数据库：                                   |
| alter table [表名A] rename to [表名B];                       | 重命名一个表：                                 |
| drop table [表名];                                           | 删除一个表：                                   |
| alter table [表名] add column [字段名] [类型];               | 在已有的表里添加字段：                         |
| alter table [表名] drop column [字段名];                     | 删除表中的字段：                               |
| alter table [表名] rename column [字段名A] to [字段名B];     | 重命名一个字段：                               |
| alter table [表名] alter column [字段名] set default [新的默认值]; | 给一个字段设置缺省值：                         |
| alter table [表名] alter column [字段名] drop default;       | 去除缺省值：                                   |
| insert into 表名 ([字段名m],[字段名n],......) values ([列m的值],[列n的值],......); | 在表中插入数据：                               |
| update [表名] set [目标字段名]=[目标值] where [该行特征];    | 修改表中的某行某列的数据：                     |
| delete from [表名] where [该行特征]; <br/>delete from [表名];--删空整个表 | 删除表中某行数据：                             |
| TRUNCATE TABLE tablename;                                    | 清空表并保留表结构                             |
| goo_all=# select pg_size_pretty(pg_database_size('goo_all')); | 查看数据库大小：                               |
| select pg_size_pretty(pg_relation_size('rtu_zaiko_10'));     | 查看表的大小：【数据都在子表中】rtu_zaiko大小0 |

#### 数据的导入与导出

- 参考地址

  > https://www.cnblogs.com/baxk/p/4895573.html
  >
  > https://blog.csdn.net/allen_oscar/article/details/9061573

- 导出备份数据库

  > pg_dump -h localhost -U postgres databasename > /tmp/databasename.bak.yyyymmdd.sql
  >
  > pg_dump -h localhost -U gn_owner calender  > /tmp/test_dump_calender.190909.sql

  - 也可以这样

    > -W, --password           force password prompt (should happen automatically)
    >
    > -f, --file=FILENAME         output file or directory name
    >
    > > pg_dump -U username -W dbname -f /tmp/filename.sql
    > >
    > > [gn_owner@da-goo-a001 local]$ pg_dump -U gn_owner -W calender -f /tmp/test_dump_calender.190909_01.sql
    > > Password:

- 导入恢复数据库(sql文件是pg_dump导出的文件就行，可以是整个数据库，也可以只是单个表，也可以只是结构等)：

  > psql -h localhost -U postgres -d databasename < /tmp/databasename.bak.yyyymmdd.sql

- 导出某个表

  > -t, --table=TABLE           dump the named table(s) only
  >
  > > pg_dump -h localhost -U postgres -t tablename dbname > test.sql

- 导出某个表的结构，加参数"-s"

  > -s, --schema-only           dump only the schema, no data
  >
  > > pg_dump -h localhost -U postgres -t tablename -s dbname > test_construct.sql

- 导出某个表的数据，加参数"-a"

  > -a, --data-only             dump only the data, not the schema
  >
  > > pg_dump -h localhost -U postgres -t tablename -a dbname > test_data.sql

- 标准化输出

  > -F, --format=c|d|t|p        output file format (custom, directory, tar, plain text) #可以去掉空行等数据

  ```
  psql -U tana -d goo_contents -h pgsv1121 -p 5432 -c "select h_price from target_mail;" -A -F'|' > /tmp/7999912_client.csv
  ```

#### 事务处理

- 针对update，delete等，写好后，要给其他人check后再执行

```
begin;
UPDATE zaiko_13 SET catch_string='大判キャッチ４'||CHR(9) WHERE goo_car_id='700010403530180319004';
commit;
```

#### psql11.2本地化安装手顺

```
su root
cd /usr/local/src
wget https://ftp.postgresql.org/pub/source/v11.2/postgresql-11.2.tar.bz2
tar -jxvf postgresql-11.2.tar.bz2
mkdir /usr/local/postgresql-11.2
chown postgres:postgres postgresql-11.2
cd /usr/local/src/postgresql-11.2
./configure \
--prefix=/usr/local/postgresql-11.2 \
--with-perl
make && make install
cd /usr/local
ln -s  postgresql-11.2 psql
su - postgres
/usr/local/psql/bin/initdb -D /usr/local/psql/data
------------------
初始化后显示：
Success. You can now start the database server using:
启动
/usr/local/psql/bin/pg_ctl -D /usr/local/psql/data -l logfile start
------------------
启动
/usr/local/psql/bin/postgres -D /usr/local/psql/data >logfile 2>&1 &
停止
/usr/local/psql/bin/pg_ctl stop -D /usr/local/psql/data
改配置文件
/usr/local/psql/data/postgresql.conf
↓
listen_addresses = '*'
/usr/local/psql/data/pg_hba.conf
↓
host    all             all             192.168.39.94/32        trust
在psql中查看函数信息命令：\df
由于开发环境和本地环境的函数不同，其中range要编译，方法如下：
将开发环境下的pg_function.tgz拿到本地环境来
tar -zxvf pg_function.tgz
ls
cd pg_function
ls
make clean
make
把root的改成postgres的所有者，所属组
[postgres@new-db6 psql]$ /usr/local/psql/bin/psql
psql (11.2)
Type "help" for help.
postgres=# CREATE FUNCTION range(text, text, varchar, varchar)
postgres-# RETURNS FLOAT4
postgres-# AS '/usr/local/psql/pg_function/pg_range.so',
postgres-# 'pg_range'
postgres-# LANGUAGE 'c';
CREATE FUNCTION
postgres=# \df
                                      List of functions
 Schema | Name  | Result data type |               Argument data types                | Type
--------+-------+------------------+--------------------------------------------------+------
 public | range | real             | text, text, character varying, character varying | func
(1 row)
postgres=# \sf+ range
        CREATE OR REPLACE FUNCTION public.range(text, text, character varying, character varying)
         RETURNS real
         LANGUAGE c
1       AS '/usr/local/psql/pg_function/pg_range.so', $function$pg_range$function$
postgres=#
-------------------------------------------------------------------
/usr/local/pgsql/bin/psql -d postgres -U postgres -h localhost
postgres=# \du
/usr/local/psql/bin/psql -d postgres -U postgres -h localhost
CREATE USER ecochu;
CREATE USER idac;
CREATE USER pdc;
CREATE USER pdc_view;
CREATE USER blog;
ALTER ROLE blog SUPERUSER CREATEROLE CREATEDB REPLICATION;
CREATE USER ex_owner;
ALTER ROLE ex_owner SUPERUSER CREATEROLE CREATEDB REPLICATION;
CREATE USER gn_owner;
ALTER ROLE gn_owner SUPERUSER CREATEROLE CREATEDB REPLICATION;
CREATE USER goo_owner;
ALTER ROLE goo_owner SUPERUSER CREATEROLE CREATEDB REPLICATION;
CREATE USER gp_owner;
ALTER ROLE gp_owner SUPERUSER CREATEROLE CREATEDB REPLICATION;
CREATE USER gw_owner;
ALTER ROLE gw_owner SUPERUSER CREATEROLE CREATEDB REPLICATION;
CREATE USER ic_owner;
ALTER ROLE ic_owner SUPERUSER CREATEROLE CREATEDB REPLICATION;
CREATE USER svi;
ALTER ROLE svi SUPERUSER CREATEROLE CREATEDB REPLICATION;
CREATE USER tana;
ALTER ROLE tana SUPERUSER CREATEROLE CREATEDB REPLICATION;
CREATE USER webmaster;
ALTER ROLE webmaster SUPERUSER CREATEROLE CREATEDB REPLICATION;
CREATE DATABASE goo_all OWNER tana ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE shinsha OWNER tana ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE toppage_all OWNER tana ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE goo_contents OWNER tana ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE blog OWNER blog ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE estimate OWNER tana ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE goobike OWNER svi ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE goonet OWNER gn_owner ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE inspect OWNER gn_owner ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE goo_access OWNER tana ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE banner OWNER tana ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE dirsystem OWNER tana ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE goo_premium OWNER tana ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE adv OWNER tana ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE sell_next OWNER tana ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE kurumado OWNER tana ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE newcar_lease OWNER tana ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE access_db OWNER tana ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE auctiondb OWNER tana ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE map OWNER tana ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE bluebook OWNER tana ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE goo_passport OWNER tana ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
CREATE DATABASE goopartsdb OWNER tana ENCODING='EUC_JP' LC_COLLATE='C' LC_CTYPE='C' TEMPLATE template0;
[postgres@new-db ~]$ /usr/local/pgsql/bin/psql
psql (9.5.6)
Type "help" for help.
postgres=# \dt
No relations found.
postgres=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 blog      | Superuser, Create DB                                       | {}
 ecochu    |                                                            | {}
 ex_owner  | Superuser, Create role, Create DB                          | {}
 gn_owner  | Superuser, Create role, Create DB                          | {}
 goo_owner | Superuser, Create role, Create DB                          | {}
 gp_owner  | Superuser, Create role, Create DB                          | {}
 gw_owner  | Superuser, Create role, Create DB                          | {}
 ic_owner  | Superuser, Create role, Create DB                          | {}
 idac      |                                                            | {}
 pdc       |                                                            | {}
 pdc_view  |                                                            | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 svi       | Superuser, Create role, Create DB                          | {}
 tana      | Superuser, Create role, Create DB                          | {}
 webmaster | Superuser, Create role, Create DB                          | {}
postgres=# \l
                                   List of databases
     Name     |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
--------------+----------+----------+-------------+-------------+-----------------------
 access_db    | tana     | EUC_JP   | C           | C           |
 adv          | tana     | EUC_JP   | C           | C           |
 auctiondb    | tana     | EUC_JP   | C           | C           |
 banner       | tana     | EUC_JP   | C           | C           |
 blog         | blog     | EUC_JP   | C           | C           |
 bluebook     | tana     | EUC_JP   | C           | C           |
 dirsystem    | tana     | EUC_JP   | C           | C           |
 estimate     | tana     | EUC_JP   | C           | C           |
 goo_access   | tana     | EUC_JP   | C           | C           |
 goo_all      | tana     | EUC_JP   | C           | C           |
 goo_contents | tana     | EUC_JP   | C           | C           |
 goo_passport | tana     | EUC_JP   | C           | C           |
 goo_premium  | tana     | EUC_JP   | C           | C           |
 goobike      | svi      | EUC_JP   | C           | C           |
 goonet       | gn_owner | EUC_JP   | C           | C           |
 goopartsdb   | tana     | EUC_JP   | C           | C           |
 inspect      | gn_owner | EUC_JP   | C           | C           |
 kurumado     | tana     | EUC_JP   | C           | C           |
 map          | tana     | EUC_JP   | C           | C           |
 newcar_lease | tana     | EUC_JP   | C           | C           |
 postgres     | postgres | UTF8     | en_GB.UTF-8 | en_GB.UTF-8 |
 sell_next    | tana     | EUC_JP   | C           | C           |
 shinsha      | tana     | EUC_JP   | C           | C           |
 template0    | postgres | UTF8     | en_GB.UTF-8 | en_GB.UTF-8 | =c/postgres          +
              |          |          |             |             | postgres=CTc/postgres
 template1    | postgres | UTF8     | en_GB.UTF-8 | en_GB.UTF-8 | =c/postgres          +
              |          |          |             |             | postgres=CTc/postgres
 toppage_all  | tana     | EUC_JP   | C           | C           |
(26 rows)
/usr/local/psql/bin/pg_restore -U tana -d goo_all -h localhost < /vagrant/playbook/roles/dbrestore/files/goo_all.dump
/usr/local/psql/bin/pg_restore -U tana -d shinsha -h localhost < /vagrant/playbook/roles/dbrestore/files/shinsha.dump
/usr/local/psql/bin/pg_restore -U tana -d toppage_all -h localhost < /vagrant/playbook/roles/dbrestore/files/toppage_all.dump
/usr/local/psql/bin/pg_restore -U tana -d goobike -h localhost < /vagrant/playbook/roles/dbrestore/files/goobike.dump
/usr/local/psql/bin/pg_restore -U tana -d bluebook -h localhost < /vagrant/playbook/roles/dbrestore/files/bluebook.dump
/usr/local/psql/bin/pg_restore -U tana -d goo_access -h localhost < /vagrant/playbook/roles/dbrestore/files/goo_access.dump
/usr/local/psql/bin/pg_restore -U tana -d goo_premium -h localhost < /vagrant/playbook/roles/dbrestore/files/goo_premium.dump
/usr/local/psql/bin/pg_restore -U tana -d access_db -h localhost < /vagrant/playbook/roles/dbrestore/files/access_db.dump
/usr/local/psql/bin/pg_restore -U tana -d sell_next -h localhost < /vagrant/playbook/roles/dbrestore/files/sell_next.dump
/usr/local/psql/bin/pg_restore -U tana -d map -h localhost < /vagrant/playbook/roles/dbrestore/files/map.dump
/usr/local/psql/bin/pg_restore -U tana -d kurumado -h localhost < /vagrant/playbook/roles/dbrestore/files/kurumado.dump
/usr/local/psql/bin/pg_restore -U tana -d dirsystem -h localhost < /vagrant/playbook/roles/dbrestore/files/dirsystem.dump
/usr/local/psql/bin/pg_restore -U tana -d auctiondb -h localhost < /vagrant/playbook/roles/dbrestore/files/auctiondb.dump
/usr/local/psql/bin/pg_restore -U tana -d goonet -h localhost < /vagrant/playbook/roles/dbrestore/files/goonet.dump
/usr/local/psql/bin/pg_restore -U tana -d banner -h localhost < /vagrant/playbook/roles/dbrestore/files/banner.dump
/usr/local/psql/bin/pg_restore -U tana -d inspect -h localhost < /vagrant/playbook/roles/dbrestore/files/inspect.dump
/usr/local/psql/bin/pg_restore -U tana -d newcar_lease -h localhost < /vagrant/playbook/roles/dbrestore/files/newcar_lease.dump
/usr/local/psql/bin/pg_restore -U tana -d adv -h localhost < /vagrant/playbook/roles/dbrestore/files/adv.dump
/usr/local/psql/bin/pg_restore -U tana -d goo_contents -h localhost < /vagrant/playbook/roles/dbrestore/files/goo_contents_schema.dump
/usr/local/psql/bin/pg_restore -U tana -d estimate -h localhost < /vagrant/playbook/roles/dbrestore/files/estimate_schema.dump
/usr/local/psql/bin/pg_restore -U tana -d blog -h localhost < /vagrant/playbook/roles/dbrestore/files/blog_schema.dump
/usr/local/psql/bin/pg_restore -U tana -d goo_passport -h localhost < /vagrant/playbook/roles/dbrestore/files/goo_passport.dump
```


