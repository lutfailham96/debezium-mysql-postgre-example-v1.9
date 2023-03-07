### Start services

```shell
$ docker-compose up
```

### MySQL: Enabling the binlog
https://debezium.io/documentation/reference/1.9/connectors/mysql.html#enable-mysql-binlog

for MySql 5.x
```shell
mysql> SELECT variable_value as "BINARY LOGGING STATUS (log-bin) ::" FROM information_schema.global_variables WHERE variable_name='log_bin';
```

for MySql 8.x
```shell
mysql> SELECT variable_value as "BINARY LOGGING STATUS (log-bin) ::" FROM performance_schema.global_variables WHERE variable_name='log_bin';
```

If it is OFF, configure your MySQL server configuration file `my.cnf` with the following properties, which are described below:
```shell
server-id         = 223344
log_bin           = mysql-bin
binlog_format     = ROW
binlog_row_image  = FULL
expire_logs_days  = 10
```

### MySQL: Grant the required permissions to the user:

Login As `root` or `Super Admin`
```shell
$ mysql -u root -p
```

Grant the required permissions to the user `user` in this example
```shell
mysql> GRANT SELECT, RELOAD, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'user' IDENTIFIED BY '123456';

mysql> FLUSH PRIVILEGES;
```

## Kafka Connect Rest API
https://docs.confluent.io/platform/current/connect/references/restapi.html#kconnect-rest-interface

### Start JDBC Sink Connector (Postgre Target Table MOVIE)
```shell
$ curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://127.0.0.1:8083/connectors/ -d @jdbc-sink-table-movie.json
```

### Start JDBC Sink Connector (Postgre Target Table PERSON)
```shell
$ curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://127.0.0.1:8083/connectors/ -d @jdbc-sink-table-person.json
```

### Start Debezium MySQL CDC connector (MySQL Source)

```shell
$ curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://127.0.0.1:8083/connectors/ -d @mysql-source.json
```

### Insert or update example data to MySQL
```
mysql> INSERT INTO MOVIE(TITLE, DESCRIPTION) VALUES('Spiderman 2', 'good');

mysql> INSERT INTO MOVIE(TITLE, DESCRIPTION) VALUES('Spiderman 3', 'good');
```

### Now, the data in the Postgres database should also change
```shell
debeziumtest=# SELECT * FROM "public"."mysqldbserver1_mydbMOVIE";
 DESCRIPTION |      created_at      |                   TITLE                    | ID 
-------------+----------------------+--------------------------------------------+----
 good        | 2023-03-06T14:11:18Z | Spiderman 2                                | 1
 good        | 2023-03-06T15:32:54Z | Spiderman 3                                | 2
(2 rows)
```

#### Reference:
- https://github.com/debezium/debezium-examples/tree/1.x/unwrap-smt
- https://debezium.io/documentation/reference/1.9/connectors/mysql.html#enable-mysql-binlog