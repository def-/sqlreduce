SQLreduce: Reducing verbose SQL queries to minimal examples
===========================================================

[SQLsmith](https://github.com/anse1/sqlsmith) has proven to be an effective
tool for finding bugs in different areas in the PostgreSQL server and related
products, including security bugs, ranging from executor bugs to segfaults in
type and index method implementations.

However, the random queries generated by SQLsmith that trigger some error are
most often very large and contain a lot of noise that does not contribute to
the error. So far, manual inspection of the query and tedious editing was
required to reduce the example to a minimal reproducer that developers can use
to fix the problem.

This issue is addressed by SQLreduce. SQLreduce that takes as input an
arbitrary SQL query. The query is run against a PostgreSQL server, and various
simplification steps are applied, while checking that the simplified query
still exhibits the same error from PostgreSQL.

SQLreduce is effective at reducing the queries from
[original error reports from SQLsmith](https://github.com/anse1/sqlsmith/wiki#score-list)
to queries that match manually-reduced queries.

Requirements
------------

* [libpg_query](https://github.com/pganalyze/libpg_query) -- PostgreSQL parser as library
* [pglast](https://github.com/lelit/pglast) -- Python interface to libpg_query

Debian/Ubuntu packages for these are shipped on [apt.postgresql.org](https://apt.postgresql.org).

```
apt install python3-pglast
```

Authors
-------

* Christoph Berg

License
-------

Copyright (c) 2022, PostgreSQL Global Development Group

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies
of the Software, and to permit persons to whom the Software is furnished to do
so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.