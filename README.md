#### obmysqlcok3ed
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
