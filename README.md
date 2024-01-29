# ДЗ otus-pgsql-hw-lesson-13

## реализовать свой миникластер на 3 ВМ.

Развёрнул 3 Виртуальных Сервера с POSTGRESQL-15

Создал для занятий БД otus_hw_13

### 1. На 1 ВМ создаем таблицы test для записи, test2 для запросов на чтение.

установил тип репликации - логический
alter system set wal_level = logical;
с последующим рестартом сервера

    otus_hw_13=# alter system set wal_level = logical;
    ALTER SYSTEM
    otus_hw_13=# \q
    bash-4.2$ su root
    Password:
    [root@mck-network-test data]# systemctl stop postgresql-15
    [root@mck-network-test data]# systemctl start postgresql-15
    [root@mck-network-test data]# su postgres
    bash-4.2$ psql
    Password for user postgres:
    psql (15.5)
    Type "help" for help.
    
    postgres=#
    postgres=# \c otus_hw_13
    You are now connected to database "otus_hw_13" as user "postgres".
    otus_hw_13=#


create table test1(id serial, data text);
create table test2(id serial, data text);

insert into tes1t(data) values('test1');


    otus_hw_13=# create table test1(id serial, data text);
    CREATE TABLE
    otus_hw_13=# create table test2(id serial, data text);
    CREATE TABLE
    otus_hw_13=#
    otus_hw_13=# insert into test1(data) values('test1');
    INSERT 0 1
    otus_hw_13=# insert into test2(data) values('test2');
    INSERT 0 1
    
    otus_hw_13=# select * from test1;
     id | data
    ----+-------
      1 | test1
    (1 row)
    
    otus_hw_13=# select * from test2;
     id | data
    ----+-------
      1 | test2
    (1 row)
    
    otus_hw_13=#


### 2. Создаем публикацию таблицы test1 и подписываемся на публикацию таблицы test2 с ВМ №2.

create publication test_1 for table test1;
\dRp+

    otus_hw_13=# ^C
    otus_hw_13=# create publication test_1 for table test1;
    CREATE PUBLICATION
    otus_hw_13=# \dRp+
                                 Publication test_1
      Owner   | All tables | Inserts | Updates | Deletes | Truncates | Via roo
    t
    ----------+------------+---------+---------+---------+-----------+--------
    --
     postgres | f          | t       | t       | t       | t         | f
    Tables:
        "public.test1"
    
    otus_hw_13=#

create subscription test_sub

connection 'host=10.15.100.28 port=5432 user=postgres password=postgres dbname=otus_hw_13'

publication test_2 with (copy_data = true);

    otus_hw_13=# create subscription test_sub
    connection 'host=10.15.100.28 port=5432 user=postgres password=postgres dbn  ame=otus_hw_13'
    publication test_1 with (copy_data = true);
    WARNING:  publication "test_1" does not exist on the publisher
    NOTICE:  created replication slot "test_sub" on publisher
    CREATE SUBSCRIPTION
    otus_hw_13=#

Проверил - придобавлении данных в таблицу на ВМ2 они появляются на ВМ1

    otus_hw_13=# insert into test2(data) values('test2_1');
    INSERT 0 1
    
    otus_hw_13=# select * from test2;
     id |  data
    ----+---------
      1 | test2
      1 | test2
      2 | test2_1
    (3 rows)


### 3. На 2 ВМ создаем таблицы test2 для записи, test1 для запросов на чтение.

    otus_hw_13=# create table test1(id serial, data text);
    CREATE TABLE
    otus_hw_13=# create table test2(id serial, data text);
    CREATE TABLE
    otus_hw_13=# insert into test1(data) values('test1');
    INSERT 0 1
    otus_hw_13=# insert into test2(data) values('test2');
    INSERT 0 1
    
    otus_hw_13=# select * from test1;
     id | data
    ----+-------
      1 | test1
    (1 row)
    
    otus_hw_13=# select * from test2;
     id | data
    ----+-------
      1 | test2
    (1 row)
    
    otus_hw_13=#
    



### 4. Создаем публикацию таблицы test2 и подписываемся на публикацию таблицы test1 с ВМ №1.

create publication test_2 for table test2;
\dRp+

    otus_hw_13=# ^C
    otus_hw_13=# create publication test_2 for table test2;
    CREATE PUBLICATION
    otus_hw_13=# \dRp+
                                 Publication test_2
      Owner   | All tables | Inserts | Updates | Deletes | Truncates | Via ro
    ot
    ----------+------------+---------+---------+---------+-----------+-------
    ---
     postgres | f          | t       | t       | t       | t         | f
    Tables:
        "public.test2"
    
    otus_hw_13=#

create subscription test_sub

connection 'host=10.15.100.27 port=5432 user=postgres password=postgres dbname=otus_hw_13'

publication test_1 with (copy_data = true);

        otus_hw_13=# create subscription test_sub
        otus_hw_13-# connection 'host=10.15.100.27 port=5432 user=postgres password=postgres dbname=otus_hw_13'
        otus_hw_13-# publication test_1 with (copy_data = true);
        NOTICE:  created replication slot "test_sub" on publisher
        CREATE SUBSCRIPTION
        otus_hw_13=#

подписка создана, и селектом видно что данные передаются 

### 5. 3-ю ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ).

        otus_hw_13=# create subscription test_sub1
        connection 'host=10.15.100.27 port=5432 user=postgres password=postgres dbname=otus_hw_13'
        publication test_1 with (copy_data = true);
        NOTICE:  created replication slot "test_sub1" on publisher
        CREATE SUBSCRIPTION
        otus_hw_13=#
        otus_hw_13=# create subscription test_sub2
        otus_hw_13-# connection 'host=10.15.100.28 port=5432 user=postgres password=postgres dbname=otus_hw_13'
        otus_hw_13-# publication test_2 with (copy_data = true);
        NOTICE:  created replication slot "test_sub2" on publisher
        CREATE SUBSCRIPTION
        otus_hw_13=#

Подписки созданы, селектом видно что данные заносятся

основная сложность при выполнении не запутаться в названиях подписок и публикаторов.

Сдаю без задания со *, так как проблемы с временем

