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
