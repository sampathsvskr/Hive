## Create and drop database 
List vailable databases
```
SHOW DATABASES;
```
Creating database in default location /hive/warehouse/
```
CREATE DATABASE HivePractice;
```
```
CREATE DATABASE IF NOT EXISTS HivePractice;
```

Creating database in specific location
```
CREATE DATABASE HivePractice LOCATION '/user/practice/';
```
To use the database
```
USE HivePractice;
```
Drop the database<br>
In order to drop database, it should be empty.
```
DROP DATABASE HivePractice;
```
If database is not empty, we can use `CASCADE`<br>
It drops all tables before dropping the database.
```
DROP DATABASE IF EXISTS HivePractice CASCADE;
```
The default behavior is `RESTRICT`, where DROP DATABASE will fail if the database is not empty
```
DROP DATABASE IF EXISTS HivePractice RESTRICT;
```

You can use `PURGE` option to not move the data to .Trash directory, the data will be permanently removed and it can not be recovered.
```
DROP TABLE IF EXISTS employee PURGE;
```

## Create tables
```
SHOW TABLES:
```
```
SHOW TABLES IN HivePractice;
```

```
CREATE TABLE users 
(
id INT,
name STRING,
salary INT,
unit STRING
)
ROW FORMAT DELIMITED
COMMENT 'USER details'
FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
stored as textfile;
```
Details of created table.
```
DESCRIBE users;
```

```
DESCRIBE FORMATTED users;
```

```
CREATE TABLE IF NOT EXISTS users 
(
id INT,
name STRING,
salary INT,
unit STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
stored as textfile;
```

## External table in hive
Location is mandatory for external table.
```
CREATE EXTERNAL TABLE users 
(
id INT,
name STRING,
salary INT,
unit STRING
)
ROW FORMAT DELIMITED
COMMENT 'USER details'
FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
stored as textfile
location '/user/practice/';
```

## Temporary table in hive
-Temporary table are used to improve performance by storing data outside HDFS for intermediate use, or reuse, by a complex query.<br>
-Temporary table data persists only during the current Apache Hive session. Hive drops the table at the end of the session. <br>
-If you use the name of a permanent table to create the temporary table, the permanent table is inaccessible during the session unless you drop or rename the temporary table.<br>
-You can create a temporary table having the same name as another user's temporary table because user sessions are independent.<br> 
-Temporary tables do not support partitioned columns and indexes.<br>
-Stores at users scratch directory /tmp/hive/\<user>/* <br>
-Configure Hive to store temporary table data in memory or on SSD by setting `hive.exec.temporary.table.storage`<br>
Store data in memory - `hive.exec.temporary.table.storage=memory`<br>
Store data on SSD- `hive.exec.temporary.table.storage=ssd`<br>
-For more info, visit - https://sparkbyexamples.com/apache-hive/hive-temporary-table-usage-and-how-to-create/

```
CREATE TEMPORARY TABLE users_tmp 
(
id INT,
name STRING,
salary INT,
unit STRING
)
ROW FORMAT DELIMITED
COMMENT 'temporary details'
FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
stored as textfile;
```
## Creating table from other table.

### Create Table As Select (CTAS)
Create a table from the results of the select query.<br>
The target table cannot be an external table.<br>
The target table cannot be a list bucketing table.<br>

```
CREATE TABLE users_temp AS SELECT id,name FROM users WHERE salary >100000;
```

### LIKE
Table is created with the same schema as reffered table.<br> No rows will be present in the created table.
```
CREATE TABLE user_copy LIKE
```

### Text file table
```
CREATE TABLE users 
(
id INT,
name STRING,
salary INT,
unit STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
stored as textfile;
```

### Sequence file table
```
CREATE TABLE users
(
id INT,
name STRING,
salary INT,
unit STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
stored as SEQUENCEFILE;
```

### RC file table
RCFile (Record Columnar File) is a data placement structure designed for MapReduce-based data warehouse systems. Hive added the RCFile format in version 0.6. ... RCFile stores the metadata of a row split as the key part of a record, and all the data of a row split as the value part.
```
CREATE TABLE users
(
id INT,
name STRING,
salary INT,
unit STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
stored as RCFILE;
```
### PARQUET file table
```
CREATE TABLE users
(
id INT,
name STRING,
salary INT,
unit STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
stored as PARQUET;
```

### ORC file table
```
CREATE TABLE users
(
id INT,
name STRING,
salary INT,
unit STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
stored as ORC;
```

## Loading data