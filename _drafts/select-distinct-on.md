---
layout: post
title:  "SELECT DISTINCT ON"
date:   2020-01-03
---

`SELECT DISTINCT ON ( expression [, ...] )` keeps only the first row of each set of rows where the given expressions evaluate to equal. (`SELECT DISTINCT` considers all selected columns.) `SELECT DISTINCT ON` expressions must match initial `ORDER BY` expressions, because Postgres sorts records in order to filter duplicates.
