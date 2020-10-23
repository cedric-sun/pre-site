CLI usage
===========
-h <host>       specify the host on which the sql server is running
-p<pswd>        (NO INTERVENING SPACE) specify password in-place on command line; It's a problem to have a space in between: `mysql -u user -p pswd` is connect to the database named `pswd` and input password later in the prompt.
-u <user>

create user
===============
e.g. create a user with no privilege
    CREATE USER IF NOT EXISTS

Privileges & Grant
==============
https://mariadb.com/kb/en/grant

Determine what privileges an user has:
    `SHOW GRANTS [FOR user]`

implicit user creation
------------
https://mariadb.com/kb/en/grant/#implicit-account-creation

`GRANT` can be used to implicitly create user at the same time you grant some privileges to that (future) user.

```plain
GRANT USAGE ON *.* TO 'user123'@'%' IDENTIFIED VIA PAM using 'mariadb' require ssl ;
Query OK, 0 rows affected (0.00 sec)
```

Role
==============
A role is a bundle of privileges. By assigning an user a role, that user gets all the privileges in the role bundle. Here goes the thought of (multiple) inheritance.

create database
================
- `CREATE` privilege required.
- `CREATE OR REPLACE` supported
- database-level default charset and collation can be set.
- can have a comment for the new db

create table
=============


Character Set (charset)and Collations
======================
https://mariadb.com/kb/en/character-sets/

A character set is a set of characters while a collation is the rules for comparing and sorting a particular character set.
A character set can have many collations associated with it, while each collation is only associated with one character set.
Charset and collation inherit in a cascading style: table level charset and collation, if not specified, will inherits those from db level,etc.

In MariaDB, the character set name is always part of the collation name.
    e.g. charset `big5` has a collation named `big5_chinese_ci`.

Each character set also has one default collation.
When changing a character set and not specifying a collation, the default collation for the new character set is always used.
- `SHOW CHARACTER SET` (equiv. `SHOW CHARSET`) displays all supported charset and their respective default collation.
- `SHOW COLLATION` displays all supported collation and the charset with which they must be used.
Both command retrieve information from database `information_schema`.

For Mariadb:
    default charset:    latin1
    default collation:  latin1_swedish_ci (may vary per linux distro)

Granularity from server-wide to column.

Charset and collation can be changed per client-server connection via global variables:
    e.g.
    SET character_set_connection = latin1           # change charset to latin1, and collation to to default collation of latin1
    SET collation_connection = latin1_german2_ci;   # change collation thus charset
    

Mind the terminology abuse: "character set" or "charset" here is actually "encoding" in common sense.
    e.g. `utf8` is said to be a charset here in MariaDB, but unicode people say "Unicode" is a charset and "utf8" is a encoding of Unicode.

Note that multiple language might be represented in the same charset, their collations differ:
```plain
MariaDB [(none)]> show collation like 'utf8mb4%';
+------------------------------+---------+------+---------+----------+---------+
| Collation                    | Charset | Id   | Default | Compiled | Sortlen |
+------------------------------+---------+------+---------+----------+---------+
| utf8mb4_general_ci           | utf8mb4 |   45 | Yes     | Yes      |       1 |
| utf8mb4_bin                  | utf8mb4 |   46 |         | Yes      |       1 |
| utf8mb4_unicode_ci           | utf8mb4 |  224 |         | Yes      |       8 |
| utf8mb4_icelandic_ci         | utf8mb4 |  225 |         | Yes      |       8 |
| utf8mb4_latvian_ci           | utf8mb4 |  226 |         | Yes      |       8 |
| utf8mb4_romanian_ci          | utf8mb4 |  227 |         | Yes      |       8 |
| utf8mb4_slovenian_ci         | utf8mb4 |  228 |         | Yes      |       8 |
| utf8mb4_polish_ci            | utf8mb4 |  229 |         | Yes      |       8 |
| utf8mb4_estonian_ci          | utf8mb4 |  230 |         | Yes      |       8 |
| utf8mb4_spanish_ci           | utf8mb4 |  231 |         | Yes      |       8 |
| utf8mb4_swedish_ci           | utf8mb4 |  232 |         | Yes      |       8 |
| utf8mb4_turkish_ci           | utf8mb4 |  233 |         | Yes      |       8 |
| utf8mb4_czech_ci             | utf8mb4 |  234 |         | Yes      |       8 |
| utf8mb4_danish_ci            | utf8mb4 |  235 |         | Yes      |       8 |
| utf8mb4_lithuanian_ci        | utf8mb4 |  236 |         | Yes      |       8 |
| utf8mb4_slovak_ci            | utf8mb4 |  237 |         | Yes      |       8 |
| utf8mb4_spanish2_ci          | utf8mb4 |  238 |         | Yes      |       8 |
| utf8mb4_roman_ci             | utf8mb4 |  239 |         | Yes      |       8 |
| utf8mb4_persian_ci           | utf8mb4 |  240 |         | Yes      |       8 |
| utf8mb4_esperanto_ci         | utf8mb4 |  241 |         | Yes      |       8 |
| utf8mb4_hungarian_ci         | utf8mb4 |  242 |         | Yes      |       8 |
| utf8mb4_sinhala_ci           | utf8mb4 |  243 |         | Yes      |       8 |
| utf8mb4_german2_ci           | utf8mb4 |  244 |         | Yes      |       8 |
| utf8mb4_croatian_mysql561_ci | utf8mb4 |  245 |         | Yes      |       8 |
| utf8mb4_unicode_520_ci       | utf8mb4 |  246 |         | Yes      |       8 |
| utf8mb4_vietnamese_ci        | utf8mb4 |  247 |         | Yes      |       8 |
| utf8mb4_croatian_ci          | utf8mb4 |  608 |         | Yes      |       8 |
| utf8mb4_myanmar_ci           | utf8mb4 |  609 |         | Yes      |       8 |
| utf8mb4_thai_520_w2          | utf8mb4 |  610 |         | Yes      |       4 |
| utf8mb4_general_nopad_ci     | utf8mb4 | 1069 |         | Yes      |       1 |
| utf8mb4_nopad_bin            | utf8mb4 | 1070 |         | Yes      |       1 |
| utf8mb4_unicode_nopad_ci     | utf8mb4 | 1248 |         | Yes      |       8 |
| utf8mb4_unicode_520_nopad_ci | utf8mb4 | 1270 |         | Yes      |       8 |
+------------------------------+---------+------+---------+----------+---------+
33 rows in set (0.001 sec)

```

To create / determine / alter default charset and collation at server / db / table / column level:
https://mariadb.com/kb/en/setting-character-sets-and-collations/

Ensure UTF-8 storage
====================
https://stackoverflow.com/questions/30074492/what-is-the-difference-between-utf8mb4-and-utf8-charsets-in-mysql

!!DO NOT USE `charset=utf8`!!

USE `utf8mb4` and a reasonable collation like `utf8mb4_unicode_ci` (which is the default my arch linux mariadb package use)

See: https://stackoverflow.com/questions/766809/whats-the-difference-between-utf8-general-ci-and-utf8-unicode-ci
For difference between `utf8mb4_unicode_ci` and `utf8mb4_general_ci`

special command `SHOW`
========================
https://mariadb.com/kb/en/show/
https://mariadb.com/kb/en/extended-show/



Pattern matching
==================
### basic `LIKE` / `NOT LIKE`
https://mariadb.com/kb/en/like/

case-insensitive by default;

Only 2 meta char allowed:
    % matches any number of characters, including zero.
    _ matches any single character.

### more powerful regular expression LIKE

- `REGEXP_LIKE()` function (not in MariaDB, use RLIKE instead there)
- `REGEXP` or `RLIKE` operators

case-insensitive by default;

```sql
MariaDB [menagerie]> select * from pet where owner REGEXP '^b';
+------+-------+---------+------+------------+-------+
| name | owner | species | sex  | birth      | death |
+------+-------+---------+------+------------+-------+
| Fang | Benny | dog     | m    | 1990-08-27 | NULL  |
| Slim | Benny | snake   | m    | 1996-04-29 | NULL  |
+------+-------+---------+------+------------+-------+
2 rows in set (0.000 sec)

```

3 ways to make regex pattern matching case-sensitive (mysql specific, since MariaDB doesn't have `REGEXP_LIKE`):
```sql
SELECT * FROM pet WHERE REGEXP_LIKE(name, '^b' COLLATE utf8mb4_0900_as_cs);
SELECT * FROM pet WHERE REGEXP_LIKE(name, BINARY '^b');
SELECT * FROM pet WHERE REGEXP_LIKE(name, '^b', 'c'); 
```

For MariaDB:
```sql
MariaDB [menagerie]> select * from pet where owner REGEXP BINARY '^b';
Empty set (0.000 sec)

MariaDB [menagerie]> select * from pet where owner REGEXP BINARY '^B';
+------+-------+---------+------+------------+-------+
| name | owner | species | sex  | birth      | death |
+------+-------+---------+------+------------+-------+
| Fang | Benny | dog     | m    | 1990-08-27 | NULL  |
| Slim | Benny | snake   | m    | 1996-04-29 | NULL  |
+------+-------+---------+------+------------+-------+
2 rows in set (0.001 sec)

```


Variables
========
2 class of variable: global and session

To Set a variable:
    SET character_set_server = ...

To Get a variable:
    SHOW VARIABLES LIKE 'character_set_server'


MariaDB data_type
=================
Most numeric type (except ...TODO) can be SIGNED, UNSIGNED, or ZEROFILL.
    e.g. CREATE TABLE t0 (c0 INT(4) UNSIGNED NOT NULL);

The number in the parentheses after type name (e.g. (4) above) is called the `display width` of this field. Application can retrieve this value in the column description, thus decide how to display the number. Whether use it is application's choice. It does not constrain the storage range in any sense.


Numberic Type
    Physical Type: total 9
        name        #bytes          range                                      (M) meaning             (D) meaning
        TINYINT     1 byte.     -128        to  127                            display width               /
        SMALLINT    2 byte.     -32768      to  32767                          display width               /
        MEDIUMINT   3 byte.     -8388608    to  8388607                        display width               /
        INT         4 byte.     -2147483648 to  2147483647                     display width               /
        BITINT      8 byte      -9223372036854775808 to 9223372036854775807    display width               /
        DECIMAL     TODO                                                       total # of digits      # of digits after the decimal point
                                                                               (the precision)         (the scale)
        FLOAT       32 bit float                                               (M,D): ditto; DEPRECATED: this mean sql engine will use non-ieee754 float; WILL BE REMOVED IN FUTURE;
        DOUBLE      64 bit double                                               (M,D): ditto; DEPRECATED: this mean sql engine will use non-ieee754 double; WILL BE REMOVED IN FUTURE;
        BIT                                                                    # of bits (1 to 64)         /
    Synonyms
        Boolean     TINYINT(1)
        Integer     INT
        DEC         DECIMAL
        NUMBER      DECIMAL (do not use; exist for Oracle compatibility)
        DOUBLE PRECISION        DOUBLE
        REAL        DOUBLE

(byte/text)String Type. (M) is in byte, meaning that one chinese character in utf8mb4 requires at least CHAR(3), i.e. charset sensitive.
Byte string differs from text string in that byte string has no concept of charset and collation thus can't be compared; padding is also different.
    name                                                    (M) meaning                                             if content < capacity
    CHAR(M)             fixed-length string                 fixed # of BYTE (0 to 255)                              right padded with SP (but trimed when retrieved)
    VARCHAR(M)          variable-length string              the maximum column length in characters (0 to 65535)    won't charge unused storage.
    BINARY(M)           fixed-length byte string            same as CHAR(M) (0 to 255)                              right padded with 0x00 (NOT removed when retrieved)
    VARBINARY(M)        variable-length byte string         same as VARCHAR(M)                                      won't charge unused storage.

Date & Time data type
total 5
    name
    DATE
    TIME
    DATETIME
    TIMESTAMP
    YEAR


* for limits of VARCHAR and VARBINARY mariadb doc say 65532 but mysql doc says 65535

SELECT
============================================
#### Clause Order
```
SELECT ...
FROM ... PARTITION ...
WHERE ...
GROUP BY ...
HAVING ...
WINDOW ...
ORDER BY ...
LIMIT ...
```

#### Column alias
`select_expr` can contain an alias:
```
SELECT CURDATE() as today;
````

This alias CAN ONLY be used in `GROUP BY`, `ORDER BY` and `HAVING` clause;
Specifically, This alias CANNOT be used in `WHERE` clause and other `select_expr`s.
https://dev.mysql.com/doc/refman/8.0/en/problems-with-alias.html

#### Select distinct
```sql
SELECT DISTINCT ... FROM ... [WHERE ...]
```

#### Select distinct count
```sql
SELECT COUNT(DISTINCT ...) FROM  ... [WHERE ...]
```

#### Sorted select
```sql
SELECT ... FROM ... ORDER BY ...; /* default: ascending*/
SELECT ... FROM ... ORDER BY ... DESC; /* descending */
```

For column of text string type, sort/comparison is case-insensitive by default; use keyword `BINARY` to make them case-sensitive:
```sql
SELECT ... FROM ... ORDER BY BINARY ...;
```

`ORDER BY` multiple columns is possible; Note that `ASC/DESC` is specific to each column:
```sql
SELECT ... FROM ... ORDER BY col1, col2 DESC; /* ascending sort on col1; for equal col1, descending sort on col2 */
```

ALTER
==============================

#### Sort a table in-place permanently
https://stackoverflow.com/questions/19824928/how-to-sort-a-mysql-table-in-a-permanent-way
```
ALTER TABLE <tbl_name> ORDER BY <col_name> [DESC];
```

Date Processing
======================
#### Addition
operator+ is overloaded for date arithmetic
```sql
SELECT '2018-01-01' + INTERVAL 1 MONTH; /* also notice that sql figured out that the string is a date. */
SELECT DATE_ADD('2018-01-01', INTERVAL 1 MONTH); /* equivalent */
```

#### datetime diff
```sql
/* TIMESTAMPDIFF(<unit>, <subtrahend_date>, <minuend_date>) */
SELECT TIMESTAMPDIFF(DAY, CURDATE()+INTERVAL 2 MONTH, CURDATE() + INTERVAL 1 MONTH);
+------------------------------------------------------------------------------+
| TIMESTAMPDIFF(DAY, CURDATE()+INTERVAL 2 MONTH, CURDATE() + INTERVAL 1 MONTH) |
+------------------------------------------------------------------------------+
|                                                                          -30 |
+------------------------------------------------------------------------------+
```

#### Extract component


Comparison
===============
`NULL` value is tested by `IS NULL` or `IS NOT NULL` rather by the inequality operator `<>`.

The ultimate join guide
========================
All "join" happens in the `table_references` non-terminal in `FROM` clause of `SELECT` statement.

CROSS JOIN - cartesian product
==============
Happens when you write comma separated table names

INNER JOIN
---------------
If table A is to INNER JOIN with table B,

`HAVING` vs `WHERE`
=====================
Condition specified in `WHERE` clause is evaluated on-the-fly for each row during the scanning of all row.
Condition specified in `HAVING` clause is evaluated on the result row set produced after `WHERE` condition is filetered.

They have very similar sematics. It's just the timing of perform filtering is different.
If all query condition can be tested on the fly, using `WHERE` to perform a one-pass check is enough.
For aggregate functions, it's usually a MUST to look at the result after `WHERE` have done its job, which must be done by `HAVING`.

Aggregate Functions
=================
List of all aggregate functions:
https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html

`GROUP BY` is the special clause that interacts with aggregate functions.

(from the link above)
> If you use an aggregate function in a statement containing no GROUP BY clause, it is equivalent to grouping on all rows.

You can `GROUP BY` multiple column (i.e. group by per combination of):
```sql
MariaDB [menagerie]> SELECT species,sex, COUNT(*) FROM pet GROUP BY species, sex;
+---------+------+----------+
| species | sex  | COUNT(*) |
+---------+------+----------+
| bird    | NULL |        1 |
| bird    | f    |        1 |
| cat     | f    |        1 |
| cat     | m    |        1 |
| dog     | f    |        1 |
| dog     | m    |        2 |
| hamster | f    |        1 |
| snake   | m    |        1 |
+---------+------+----------+
8 rows in set (0.000 sec)

```

When selecting column(s) together with aggregate functions, all those column name SHOULD appear in the `GROUP BY` clause. e.g.
```sql
MariaDB [menagerie]> SELECT sex, COUNT(*) FROM pet GROUP BY sex; /* Perfectly reasonable. */
+------+----------+
| sex  | COUNT(*) |
+------+----------+
| NULL |        1 |
| f    |        4 |
| m    |        4 |
+------+----------+
3 rows in set (0.001 sec)

MariaDB [menagerie]> SELECT name, sex, COUNT(*) FROM pet GROUP BY sex; /* Makes no sense. */
+----------+------+----------+
| name     | sex  | COUNT(*) |
+----------+------+----------+
| Whistler | NULL |        1 |
| Buffy    | f    |        4 |
| Bowser   | m    |        4 |
+----------+------+----------+
3 rows in set (0.001 sec)

```

Quoting
=====================
3 quote symbol exists in SQL:
#### Single quote

'string'        

#### Backtick Quote

`identifier`: Refer to a identifier.

```sql
SELECT id AS 'a', COUNT(*) AS cnt FROM tbl_name GROUP BY `a`; /* `...GROUP BY 'a'` doesn't work */
```

"???"   TODO

SQL Sharding
=================

Master Slave Replication
==================

