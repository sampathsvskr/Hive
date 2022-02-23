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
### Drop database<br>
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
location STRING
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
SHOW CRAETE TABLE usres;
```
```
CREATE TABLE IF NOT EXISTS users 
(
id INT,
name STRING,
salary INT,
location STRING
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
location STRING
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
location STRING
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
## File Formats
### Text file table
```
CREATE TABLE users 
(
id INT,
name STRING,
salary INT,
location STRING
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
location STRING
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
location STRING
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
location STRING
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
location STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
stored as ORC;
```

## Loading data from csv file

Loading data from local file system
```
LOAD DATA LOCAL INPATH '/root/hive/test.csv' INTO TABLE users;
```

Loading data from file in HDFS 
```
LOAD DATA INPATH '/root/hive/test.csv' INTO TABLE users;
```

Overwrite the file.
```
LOAD DATA LOCAL INPATH '/root/hive/test.csv' OVERWRITE INTO TABLE users;
```
If table is partitioned, need to use `PARTITION` to load data into specific partition.
```
LOAD DATA LOCAL INPATH '/root/hive/test.csv' OVERWRITE INTO TABLE users PARTITION(location=2);
```
### Insert commands to populate data to hive tables

Insert single row.
```
INSERT INTO users VALUES 
(1,'sam',120000,"Hyderabad");
```
Insert multiple rows.
```
INSERT INTO users VALUES 
(1,'sam',120000,"Chennai"),
(2,'ram',150000,"Bangalore");
```
Insert selected columns
```
INSERT INTO users(id,name) VALUES 
(3,'tom'),
(4,'surya');
```
Insert from other table
```
INSERT INTO users SELECT * FROM temp_users;
```
Overwrite the existing table
```
INSERT OVERWRITE INTO users SELECT * FROM temp_users;
```

### Inserting to PARTITION table
Let's assume users table is partitioned based on location.<br>
We need to specify the column name inside PARTITION. Value can be mentioned along with parttion or as the last column

```
INSERT INTO users PARTITION(location='Chennai') VALUES
(10,'ravi',120000)
```
Here it is mandatory to keep the partition column value as the last column as we have not specified inside the PARTITION
```
INSERT INTO users PARTITION(location) VALUES
(11,'anil',120000,'Bangalore');
```
### Export hive table to HDFS
```
INSERT OVERWRITE 
DIRECTORY '/user/hive/export' 
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',' 
SELECT * FROM employee;
```

### Export hive table to LOCAL
```
INSERT OVERWRITE LOCAL
DIRECTORY '/tmp/hive/export' 
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',' 
SELECT * FROM employee;
```
If we have a huge table, this exports the data into multiple part files, we can combine them into a single file using Unix cat command as shown below.

```
cat /tmp/hive/export/* > output.csv
```

### Export as excel file
We can run the queries over hive tables directly using ``hive '-e'``
```
hive -e 'select * from table' > output.tsv
```
We can list all the queries to be executed in hql file, instead of executing each one manually.
```
hive -f queries.hql
```

## Drop tables
Drops metadata and data files stored in HDFS for internal tables, where as only metadata is dropped for extrenal tables.
```
DROP TABLE users;
```
```
DROP TABLE IF EXISTS users;
```