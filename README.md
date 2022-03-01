SQLreduce: Reduce verbose SQL queries to minimal examples
=========================================================

![SQLreduce logo](docs/sqlreduce.png)

[SQLsmith](https://github.com/anse1/sqlsmith) has proven to be an effective
tool for finding bugs in different areas in the PostgreSQL server and other
products, including security bugs, ranging from executor bugs to segfaults in
type and index method implementations.

However, the random queries generated by SQLsmith that trigger some error are
most often very large and contain a lot of noise that does not contribute to
the error. So far, manual inspection of the query and tedious editing was
required to reduce the example to a minimal reproducer that developers can use
to fix the problem.

This issue is solved by SQLreduce. SQLreduce takes as input an arbitrary SQL
query which is then run against a PostgreSQL server. Various simplification
steps are applied, checking after each step that the simplified query still
triggers the same error from PostgreSQL. The end result is a SQL query with
minimal complexity.

SQLreduce is effective at reducing the queries from
[original error reports from SQLsmith](https://github.com/anse1/sqlsmith/wiki#score-list)
to queries that match manually-reduced queries.

More details on [how it works in the documentation](docs/howitworks.md).

# Requirements

* [PostgreSQL](https://www.postgresql.org/) -- database server running the query to be reduced
* [pglast](https://github.com/lelit/pglast) -- Python interface to libpg_query
* [libpg_query](https://github.com/pganalyze/libpg_query) -- PostgreSQL parser as library (requirement of pglast)
* [psycopg2](https://www.psycopg.org/) -- Python PostgreSQL driver
* [yaml](https://pyyaml.org/) -- Python YAML library

Debian/Ubuntu packages for pglast are shipped on [apt.postgresql.org](https://apt.postgresql.org).

```
apt install python3-pglast python3-psycopg python3-yaml
```

# Usage

```
usage: sqlreduce [-h] [-d DATABASE] [-f FILE] [--sqlstate] [-t TIMEOUT] [--debug] [query ...]

Reduce a SQL query to the minimal query throwing the same error

positional arguments:
  query                 Query to reduce to minimum

optional arguments:
  -h, --help            show this help message and exit
  -d DATABASE, --database DATABASE
                        Database or connection string to use
  -f FILE, --file FILE  Read query from file [Default: stdin]
  --sqlstate            Reduce query to same SQL state instead of error message
  -t TIMEOUT, --timeout TIMEOUT
                        Statement timeout [Default: 500ms]
  --debug
```

# Example

In 2018,
[SQLsmith found a segfault](https://www.postgresql.org/message-id/87woxi24uw.fsf@ansel.ydns.eu)
in PostgreSQL running Git revision 039eb6e92f. The reproducer back then was a
[huge 40-line, 2.2kB query](media/sqlreduce-screencast.sql):

```
select
  case when pg_catalog.lastval() < pg_catalog.pg_stat_get_bgwriter_maxwritten_clean() then case when pg_catalog.circle_sub_pt(
          cast(cast(null as circle) as circle),
          cast((select location from public.emp limit 1 offset 13)
             as point)) ~ cast(nullif(case when cast(null as box) &> (select boxcol from public.brintest limit 1 offset 2)
                 then (select f1 from public.circle_tbl limit 1 offset 4)
               else (select f1 from public.circle_tbl limit 1 offset 4)
               end,
          case when (select pg_catalog.max(class) from public.f_star)
                 ~~ ref_0.c then cast(null as circle) else cast(null as circle) end
            ) as circle) then ref_0.a else ref_0.a end
       else case when pg_catalog.circle_sub_pt(
          cast(cast(null as circle) as circle),
          cast((select location from public.emp limit 1 offset 13)
             as point)) ~ cast(nullif(case when cast(null as box) &> (select boxcol from public.brintest limit 1 offset 2)
                 then (select f1 from public.circle_tbl limit 1 offset 4)
               else (select f1 from public.circle_tbl limit 1 offset 4)
               end,
          case when (select pg_catalog.max(class) from public.f_star)
                 ~~ ref_0.c then cast(null as circle) else cast(null as circle) end
            ) as circle) then ref_0.a else ref_0.a end
       end as c0,
  case when (select intervalcol from public.brintest limit 1 offset 1)
         >= cast(null as "interval") then case when ((select pg_catalog.max(roomno) from public.room)
             !~~ ref_0.c)
        and (cast(null as xid) <> 100) then ref_0.b else ref_0.b end
       else case when ((select pg_catalog.max(roomno) from public.room)
             !~~ ref_0.c)
        and (cast(null as xid) <> 100) then ref_0.b else ref_0.b end
       end as c1,
  ref_0.a as c2,
  (select a from public.idxpart1 limit 1 offset 5) as c3,
  ref_0.b as c4,
    pg_catalog.stddev(
      cast((select pg_catalog.sum(float4col) from public.brintest)
         as float4)) over (partition by ref_0.a,ref_0.b,ref_0.c order by ref_0.b) as c5,
  cast(nullif(ref_0.b, ref_0.a) as int4) as c6, ref_0.b as c7, ref_0.c as c8
from
  public.mlparted3 as ref_0
where true;
```

SQLreduce can effectively reduce that monster to just this:

```
SELECT pg_catalog.stddev(NULL) OVER () AS c5 FROM public.mlparted3 AS ref_0
```

At the end of the video we can see some of the extra steps where SQLreduce has
tried to remove more parts of the query, but removing these also removes the
error.

```
sqlreduce -d 'dbname=regression' media/sqlreduce-screencast.sql
```

![SQLreduce screencast](media/sqlreduce-screencast.gif)

The run time of this example is entirely limited by the time PostgreSQL needs to
restart after crashing. SQLreduce itself is much faster.

# Authors

* Christoph Berg

# License

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
