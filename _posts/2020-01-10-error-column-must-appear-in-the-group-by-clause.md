---
layout: post
title:  "ERROR: column ... must appear in the GROUP BY clause or be used in an aggregate function"
date:   2020-01-10
---

This makes sense, because if we try to `SELECT` a column not included in the `GROUP BY` clause and there are multiple possible values for that column in a group, the database will not know which value to return.

The SQL standard requires that selected columns be functionally dependent on those in the `GROUP BY` clause. This means that a primary key can be used in the `GROUP BY` clause in place of a full list of selected columns, where the selected columns are dependent on that key.

* <https://www.postgresql.org/docs/12/sql-select.html#SQL-GROUPBY>
* <https://dev.mysql.com/doc/refman/8.0/en/group-by-handling.html>
