# REPEATABLE-READ事务隔离级别下，可能读出已删除的记录

事务1：
```sql
mysql> select @@global.tx_isolation,@@session.tx_isolation,@@tx_isolation;
+-----------------------+------------------------+-----------------+
| @@global.tx_isolation | @@session.tx_isolation | @@tx_isolation  |
+-----------------------+------------------------+-----------------+
| SERIALIZABLE          | REPEATABLE-READ        | REPEATABLE-READ |
+-----------------------+------------------------+-----------------+
1 row in set (0.00 sec)

mysql> start transaction;
Query OK, 0 rows affected (0.01 sec)

mysql> select * from t_number;
+-----+
| id  |
+-----+
| 123 |
| 124 |
+-----+
2 rows in set (0.00 sec)

mysql> delete from t_number where id=124;
Query OK, 1 row affected (0.01 sec)

mysql> select * from t_number;
+-----+
| id  |
+-----+
| 123 |
+-----+
1 row in set (0.00 sec)

mysql> commit;
Query OK, 0 rows affected (0.00 sec)
```

事务2：
```
mysql> select @@global.tx_isolation,@@session.tx_isolation,@@tx_isolation;
+-----------------------+------------------------+-----------------+
| @@global.tx_isolation | @@session.tx_isolation | @@tx_isolation  |
+-----------------------+------------------------+-----------------+
| SERIALIZABLE          | REPEATABLE-READ        | REPEATABLE-READ |
+-----------------------+------------------------+-----------------+
1 row in set (0.00 sec)

mysql> use test
Database changed
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from t_number;
+-----+
| id  |
+-----+
| 123 |
| 124 |
+-----+
2 rows in set (0.00 sec)
// --------- 此时事务1还没删除，读出两行记录 ---------

mysql> select * from t_number;
+-----+
| id  |
+-----+
| 123 |
| 124 |
+-----+
2 rows in set (0.00 sec)
// --------- 此时事务1已经执行了delete删除了一行，仍能读出两行记录 ---------

mysql> select * from t_number;
+-----+
| id  |
+-----+
| 123 |
| 124 |
+-----+
2 rows in set (0.00 sec)
// --------- 此时事务1已经commit，仍能读出两行记录 --------- 

mysql> commit;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from t_number;
+-----+
| id  |
+-----+
| 123 |
+-----+
1 row in set (0.00 sec)

```

