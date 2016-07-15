#### obmysqlcok3ed
INFORMATION_SCHEMA performance_schema  (case-sensitive)
#####Chapter 1. Using the mysql
######Specifying mysql Command
```
[client]
host     = localhost
user     = cbuser
password = cbpass
```
######Executing SQL Statements Interactively
exucute multiple commands inline
```
mysql -e "SELECT COUNT(*) FROM tb;SELECT NOW()" db
```
######Executing SQL Statements Read from a File
```
mysql db < limbs.sql
```
limbs.sql
```
DROP TABLE IF EXISTS limbs;
CREATE TABLE limbs
(
  thing VARCHAR(20),legs  INT
);

INSERT INTO limbs (thing,legs) VALUES('human',2);
```
######Controlling mysql Output
```
mysql -H -e "SELECT * FROM limbs WHERE legs=0" cookbook
```
redirect
```
echo "SELECT * FROM limbs WHERE legs=0" | mysql cookbook
```
output HTML
```
mysql -H -e "SELECT * FROM limbs WHERE legs=0" cookbook
```
do not output column name
```
mysql --skip-column-names -e "SELECT arms FROM limbs" cookbook
mysql -ss -e "SELECT arms FROM limbs" cookbook
echo "SELECT NOW()" | mysql -vvv
```
######Using User-Defined Variables
```
mysql
```
```
SELECT @max_limbs := MAX(arms+legs) FROM limbs;
SELECT * FROM limbs WHERE arms+legs = @max_limbs;
```
LAST_INSERT_ID()
```
SELECT @last_id := LAST_INSERT_ID();
```
```
SET @max_limbs = (SELECT MAX(arms+legs) FROM limbs);
SET @x = 1, @X = 2; SELECT @x, @X;
```
#####Chapter 3. Selecting Data from Tables
######Naming Query Result Columns
```
SELECT DATE_FORMAT(t,'%M %e, %Y') AS date_sent, CONCAT(srcuser,'@',srchost) AS sender,size FROM mail;
```
You cannot refer to column aliases in a WHERE clause. 
```
SELECT t, srcuser, dstuser, size/1024 AS kilobytes FROM mail WHERE size/1024 > 500;
```
######Removing Duplicate Rows
```
SELECT DISTINCT YEAR(t), MONTH(t), DAYOFMONTH(t) FROM mail;
```
######NULL
```
SELECT * FROM expt WHERE score IS NOT NULL;
```
IFNULL
```
SELECT subject, test, IFNULL(score,'Unknown') AS 'score' FROM expt;
```
######Create view
```
CREATE VIEW mail_view AS SELECT DATE_FORMAT(t,'%M %e, %Y') AS date_sent FROM mail;
```
use this view
```
SELECT date_sent, sender, size FROM mail_view WHERE size > 100000 ORDER BY size;
```
######skip and limit
```
SELECT ... FROM ... ORDER BY ... LIMIT 40, 20;
SELECT FOUND_ROWS();   #last query return result number of rows
```
######“Wrong” Sort Order
returns the names and birth dates for the four people born most recently
```
SELECT name, birth FROM profile ORDER BY birth DESC LIMIT 4;
```
in ascending order
```
SELECT * FROM (SELECT name, birth FROM profile ORDER BY birth DESC LIMIT 4) AS t ORDER BY birth;
```
######Calculating LIMIT Values from Expressions
To construct a two-argument LIMIT clause, evaluate both expressions before placing them into the statement string.

#####Chapter 4. Table Management
######
clone table structure
```
CREATE TABLE new_table LIKE original_table;
```
copy table contents
```
INSERT INTO new_table SELECT * FROM original_table;
```
######
copy some columns:
```
INSERT INTO dst_tbl (integercol, stringcol) SELECT val, name FROM src_tbl;
```
create empty table
```
CREATE TABLE dst_tbl SELECT col1,col2 FROM src_tbl WHERE FALSE;
```
create prop
```
CREATE TABLE dst_tbl (PRIMARY KEY (id), INDEX(state,city))
SELECT * FROM src_tbl;
```
others
```
CREATE TABLE dst_tbl (PRIMARY KEY (id)) SELECT * FROM src_tbl;
ALTER TABLE dst_tbl MODIFY id INT UNSIGNED NOT NULL AUTO_INCREMENT;
```
######Creating Temporary Tables
A temporary table can have the same name as a permanent table. In this case, the temporary table “hides” the permanent table for the duration of its existence,
```
CREATE TEMPORARY TABLE mail SELECT * FROM mail;
```
######Generating Unique Table Names
get id
```
SELECT CONNECTION_ID();
```
set prepare, deallocate
```
SET @tbl_name = CONCAT('tmp_tbl_', CONNECTION_ID());
SET @stmt = CONCAT('DROP TABLE IF EXISTS ', @tbl_name);
PREPARE stmt FROM @stmt;
EXECUTE stmt;
DEALLOCATE PREPARE stmt;
SET @stmt = CONCAT('CREATE TABLE ', @tbl_name, ' (i INT)');
PREPARE stmt FROM @stmt;
EXECUTE stmt;
DEALLOCATE PREPARE stmt;
```
######Checking or Changing a Table Storage Engine
check engine
```
mysql> select engine from information_schema.tables where table_schema='cookbook' and table_name = 'limbs';
SHOW TABLE STATUS LIKE 'mail'\G
```
change engine
```
ALTER TABLE mail ENGINE = MyISAM;
```
######Copying a Table Using mysqldump
```
mysqldump db tb > a.sql
```
copy a dump to another database: two steps
```
mysqldump db1 tb > mail.sql
mysql db2 < mail.sql
```
one step
```
mysqldump db1 tb | mysql -h other-host db2
```
or using ssh
```
mysqldump db1 tb | ssh other-host mysql db2
```




#####Chapter 5. Working with Strings
######String Properties
```
SHOW CHARACTER SET;
```
```
SET @s = CONVERT('abc' USING ucs2);
SELECT LENGTH(@s), CHAR_LENGTH(@s);
```

```
SHOW COLLATION LIKE 'latin1%';
```
create column with charset
```
CREATE TABLE t (c CHAR(3) CHARACTER SET latin1);
```
case insensitive, case sensitive
```
SELECT c FROM t ORDER BY c COLLATE latin1_swedish_ci;
SELECT c FROM t ORDER BY c COLLATE latin1_general_cs;
```
######
```
CREATE TABLE t (c1 CHAR(10), c2 VARCHAR(10));
INSERT INTO t (c1,c2) VALUES('abc       ','abc       ');
SELECT c1, c2, CHAR_LENGTH(c1), CHAR_LENGTH(c2) FROM t; # 3,10
```
create column with charset and collate
```
CREATE TABLE mytbl
(
  a VARCHAR(100) CHARACTER SET utf8 COLLATE utf8_danish_ci,
  b VARCHAR(100) CHARACTER SET sjis COLLATE sjis_japanese_ci
);
```
######Checking or Changing a String’s Character Set or Collation
```
SELECT USER(), CHARSET(USER()), COLLATION(USER());
```
```
SET NAMES utf8 COLLATE 'utf8_bin';
SELECT CHARSET('abc'), COLLATION('abc');
```
######
To convert a string from one character set to another, use the CONVERT() function:
```
SET @s1 = _latin1 'my string', @s2 = CONVERT(@s1 USING utf8);
SELECT CHARSET(@s1), CHARSET(@s2);
```
To change the collation of a string, use the COLLATE operator:
```
SET @s1 = _latin1 'my string', @s2 = @s1 COLLATE latin1_spanish_ci;
SELECT COLLATION(@s1), COLLATION(@s2);
```
change char set USING
```
SELECT b,UPPER(CONVERT(b USING latin1)) AS upper,LOWER(CONVERT(b USING latin1)) AS lower FROM t;
```
function syntax
```
CREATE FUNCTION initial_cap (s VARCHAR(255)) RETURNS VARCHAR(255) DETERMINISTIC RETURN CONCAT(UPPER(LEFT(s,1)),MID(s,2));
```



#####Chapter 6. Working with Dates and Times
######Choosing a Temporal Data Type











#####Chapter 22. Server Administration
######Configuring the Server
syntax
```
SET GLOBAL sort_buffer_size = 1024 * 256;
SET SESSION sort_buffer_size = 1024 * 1024;
```
alternative syntax
```
SET @@GLOBAL.sort_buffer_size = 1024 * 256;
SET @@SESSION.sort_buffer_size = 1024 * 1024;
```
######Managing the Plug-In Interface
show plugin dic
```
SELECT @@plugin_dir;
```
######Controlling Server Logging
mysql 5.7
```
[mysqld]
log_error=err.log
log_error_verbosity=1
```
```
[mysqld]
log-bin=binlog
max_binlog_size=4G
expire_logs_days=7
```
######Rotating or Expiring Logfiles
```
mysqladmin flush-logs
```
######Rotating Log Tables 
delete old table
```
DELETE FROM mysql.general_log WHERE event_time < NOW() - INTERVAL 1 WEEK;
```
schedule event(very important,hard)
```
CREATE EVENT expire_general_log ON SCHEDULE EVERY 1 WEEKnDO DELETE FROM mysql.general_log WHERE event_time < NOW() - INTERVAL 1 WEEK;
```
######Monitoring the MySQL Server
```
SHOW GLOBAL STATUS LIKE 'Threads_connected';
SELECT VARIABLE_VALUE FROM INFORMATION_SCHEMA.GLOBAL_STATUS WHERE VARIABLE_NAME = 'Threads_connected';
```
vertical output: mysql -E
```
mysql -E -e "SHOW ENGINE INNODB STATUS" | grep "Free buffers"
```
```
SHOW PROCESSLIST;
```
```
mysqladmin ping/status
```
show uptime
```
SHOW GLOBAL STATUS LIKE 'Uptime';
```
how many queries
```
SHOW GLOBAL STATUS LIKE 'Queries';
```
wait timeout, default=8 hours
```
SELECT @@wait_timeout;
SET wait_timeout = seconds;
```

key cached
```
SELECT @@innodb_buffer_pool_size, @@key_buffer_size;
```
or
```
SELECT * FROM INFORMATION_SCHEMA.GLOBAL_VARIABLES WHERE VARIABLE_NAME IN ('INNODB_BUFFER_POOL_SIZE','KEY_BUFFER_SIZE');
```
######Creating and Using Backups
```
mysqldump --routines --events --all-databases > dump.sql
```
 






#####Chapter 23. Security
######Managing User Accounts
```
CREATE USER 'user_name'@'host_name' IDENTIFIED WITH 'sha256_password';
SET old_passwords = 2;
SET PASSWORD FOR 'user_name'@'host_name' = PASSWORD('password');
```
grant
```
GRANT FILE ON *.* TO 'user1'@'localhost';
GRANT CREATE TEMPORARY TABLES, LOCK TABLES ON *.* TO 'user1'@'localhost';
```
grant colunm level
```
GRANT SELECT(User,Host), UPDATE(password_expired) ON mysql.user TO 'user1'@'localhost';
REVOKE SELECT(User,Host), UPDATE(password_expired) ON mysql.user FROM 'user1'@'localhost';
```
procedure level
```
GRANT EXECUTE ON PROCEDURE cookbook.exec_stmt TO 'user1'@'localhost';
REVOKE EXECUTE ON PROCEDURE cookbook.exec_stmt FROM 'user1'@'localhost';
```
show grants
```
SHOW GRANTS FOR 'user1'@'localhost';
```
rename user
```
RENAME USER 'currentuser'@'localhost' TO 'newuser'@'localhost';
```
enable a plugin (cannot enable last lines, cause error)
```
[mysqld]
plugin-load-add=validate_password.so
validate_password_length=10
validate_password_mixed_case_count=1
validate_password_number_count=2
validate_password_special_char_count=1
```
######Checking Password Strength
```
SELECT VALIDATE_PASSWORD_STRENGTH('abc') ;
```
######Expiring Passwords
```
ALTER USER 'cbuser'@'localhost' PASSWORD EXPIRE;
```
batch expire all non-anoy accounts  (need to review)
```
CREATE PROCEDURE expire_all_passwords()
BEGIN
  DECLARE done BOOLEAN DEFAULT FALSE;
  DECLARE account TEXT;
  DECLARE cur CURSOR FOR
    SELECT CONCAT(QUOTE(User),'@',QUOTE(Host)) AS account
    FROM mysql.user WHERE User <> '';
  DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

  OPEN cur;
  expire_loop: LOOP
    FETCH cur INTO account;
    IF done THEN
      LEAVE expire_loop;
    END IF;
    CALL exec_stmt(CONCAT('ALTER USER ',account,' PASSWORD EXPIRE'));
  END LOOP;
  CLOSE cur;
END;
```
######Assigning Yourself a New Password
set password
```
SET PASSWORD = PASSWORD('pass');
```
set password for others
```
SET PASSWORD FOR 'user_name'@'host_name' = PASSWORD('pass');
```

######Finding and Fixing Insecure Accounts
difference between the two hash formats
```
SET old_passwords = 0;
SELECT OLD_PASSWORD('mypass') AS old, PASSWORD('mypass') AS new\G
```
 find weak accounts
```
SELECT User, Host, plugin, Password FROM mysql.user WHERE (plugin = 'mysql_native_password' AND Password = '') OR plugin IN ('','mysql_old_password');
```
assign a plugin and set a password, use comma
```
UPDATE mysql.user SET plugin = 'mysql_native_password', Password = PASSWORD('mypass') WHERE User = 'ke' AND Host = 'localhost';
FLUSH PRIVILEGES;
```
######Disabling Use of Accounts with Pre-4.1 Passwords
```
SELECT @@secure_auth;
```
if returns 0, enable it
```
[mysqld]
secure_auth=1
```
######Finding and Removing Anonymous Accounts
```
SELECT User, Host FROM mysql.user WHERE User = '';
```
######Modifying “Any Host” and “Many Host” Accounts
rename user by change host:
```
RENAME USER 'user2'@'%.example.com' TO 'user2'@'ke.example.com';
```
using regexp
```
SELECT User, Host FROM mysql.user WHERE Host REGEXP '[%_]';
```
below are same
```
WHERE Host LIKE '%\%%' OR Host LIKE '%\_%';
WHERE Host REGEXP '[%_]';
```
