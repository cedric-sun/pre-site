user create / delete
===============

grant permission
=================


create database
================
- `CREATE` privilege required.
- `CREATE OR REPLACE` supported
- database-level default charset and collation can be set.
- can have a comment for the new db


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



`LIKE` / `NOT LIKE`
=============
https://mariadb.com/kb/en/like/

Only 2 meta char allowed:
    % matches any number of characters, including zero.
    _ matches any single character.

If not being used in a `WHERE` clause, `LIKE` works on the first column for a multicolumn-result query.


Variables
========
2 class of variable: global and session

To Set a variable:
    SET character_set_server = ...

To Get a variable:
    SHOW VARIABLES LIKE 'character_set_server'

