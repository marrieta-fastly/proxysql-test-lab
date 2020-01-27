# Test Case

Test case for ProxySQL

## Run the environment

Should be enough run

```bash
docker-compose up -d
```

## Making alias

```bash
alias mysql1_root="mysql --prompt='MySQL Root> ' -u root -prootpass -P 23306 -h 127.0.0.1"
alias mysql1="mysql --prompt='MySQL> ' -u user1 -ppass1 -P 23306 -h 127.0.0.1"
alias proxysql1="mysql --prompt='PS1> ' -u user1 -ppass1 -P 26133 -h 127.0.0.1"
alias proxysql1_admin="mysql --prompt='PS1 Admin> ' -u radmin -pradmin -P 26132 -h 127.0.0.1"
alias proxysql2="mysql --prompt='PS2> ' -u user1 -ppass1 -P 26233 -h 127.0.0.1"
alias proxysql2_admin="mysql --prompt='PS2 Admin> ' -u radmin -pradmin -P 26232 -h 127.0.0.1"
```

## Initialize the DB

Connect to the DB directly.

```bash
mysql1_root
```

Create the DB and grant user1 to the test1 db

```SQL
CREATE DATABASE test1;
GRANT ALL ON test1.* to user1@'%';
CREATE TABLE `test1`.`t1` (
  `i` int(11) DEFAULT NULL,
  `v` varchar(32) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

## Execute the test

Connect to `proxysql1`:

```SQL
USE test1;
SET AUTOCOMMIT=0;
INSERT INTO t1 VALUES ( 1, "rtest1");
INSERT INTO t1 VALUES ( 2, "rtest2");
SELECT * FROM t1;
ROLLBACK;
SELECT * FROM t1;
```

```SQL
SELECT * FROM t1;
USE test1;
SET AUTOCOMMIT=0;
INSERT INTO t1 VALUES ( 1, "ctest1");
INSERT INTO t1 VALUES ( 2, "ctest2");
COMMIT;
SELECT * FROM t1;
```

## Check autocommit variables

```bash
proxysql1_admin -e "SELECT * FROM runtime_global_variables WHERE variable_name like '%autocommit%';"

proxysql2_admin -e "SELECT * FROM runtime_global_variables WHERE variable_name like '%autocommit%';"
```


## Change `mysql-autocommit_false_is_transaction`

ON:

```SQL
UPDATE global_variables SET variable_value='true'
   WHERE variable_name = 'mysql-autocommit_false_is_transaction';
LOAD MYSQL VARIABLES TO RUNTIME;
```

OFF:

```SQL
UPDATE global_variables SET variable_value='false'
   WHERE variable_name = 'mysql-autocommit_false_is_transaction';
LOAD MYSQL VARIABLES TO RUNTIME;
```

