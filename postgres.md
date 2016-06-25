# A Better Null Display Character 

By default, `psql` will display null values with whitespace. This makes it
difficult to quickly identify null values when they appear amongst a bunch
of other data. You can pick a better display value for null characters with
`\pset null`. My preference is the following:

```
\pset null 'Ø'
```

I have this in my `.psqlrc` file so that it is used by default every time.

# Adding Composite Uniqueness Constraints 

There are two ways in Postgres to create a composite uniqueness constraint;
that is, a constraint that ensures that the combination of two or more
values on a table only appear once. For the following two code snippets,
assume that we have a table relating Pokemon and Trainers and that our
domain restricts each Trainer to only having at most one of each Pokemon.

The first approach is to create a `constraint` directly on the table:

```sql
alter table pokemons_trainers
  add constraint pokemons_trainers_pokemon_id_trainer_id_key
  unique (pokemon_id, trainer_id);
```

The second approach is to create a unique index:

```sql
create unique index pokemons_trainers_pokemon_id_trainer_id_idx
  on pokemons_trainers (pokemon_id, trainer_id);
```

# Aggregate A Column Into An Array 

PostgreSQL's `array_agg` function can be used to aggregate a column into an
array. Consider the following column:

```sql
> select num from generate_series(1,5) as num;
 num
-----
   1
   2
   3
   4
   5
```

By wrapping the `array_agg` aggregate function around `num` we are able to
*aggregate* the values in that column into an array, like so:

```sql
> select array_agg(num) from generate_series(1,5) as num;
  array_agg
-------------
 {1,2,3,4,5}
```

See the docs on [aggregate
functions](http://www.postgresql.org/docs/current/static/functions-aggregate.html)
for more details.

# Auto Expanded Display 

By default, postgres has expanded display turned off. This means that
results of a query are displayed *horizontally*.
At times, the results of a query can be so wide that line wrapping occurs.
This can make the results and their corresponding column names rather
difficult to read. In these situations, it is preferable to turn on expanded
display so that results are displayed *vertically*.
The `\x` command can be used to toggle expanded display on and off.

Having to toggle expanded display on and off depending on the way a
particular set of results is going to display can be a bit tedious.
Fortunately, running `\x auto` will turn on auto expanded display. This
means postgres will display the results normally when they fit and only
switch to expanded display when it is necessary.

h/t Jack Christensen

# Checking The Type Of A Value 

The `pg_typeof()` function allows you to determine the data type of anything
in Postgres.

```sql
> select pg_typeof(1);
 pg_typeof
-----------
 integer
(1 row)

> select pg_typeof(true);
 pg_typeof
-----------
 boolean
(1 row)
```

If you try it on an arbitrary string, it is unable to disambiguate which
string type (e.g. `text` vs `varchar`).

```sql
> select pg_typeof('hello');
 pg_typeof
-----------
 unknown
(1 row)
```

You just have to be a bit more specific.

```sql
> select pg_typeof('hello'::varchar);
     pg_typeof
-------------------
 character varying
(1 row)
```

[source](http://www.postgresql.org/docs/9.3/static/functions-info.html#FUNCTIONS-INFO-CATALOG-TABLE)

# Compute Hashes With pgcrypto 

The `pgcrypto` extension that comes with PostgreSQL adds access to some
general hashing functions. Included are `md5`, `sha1`, `sha224`, `sha256`,
`sha384` and `sha512`. Any of these hashing functions can be applied to an
arbitrary string using the `digest` function. Here are example of the `md5`
and `sha1` algorithms:

```sql
> create extension pgcrypto;
CREATE EXTENSION

> select digest('Hello, World!', 'md5');
               digest
------------------------------------
 \x65a8e27d8879283831b664bd8b7f0ad4

> select digest('Hello, World!', 'sha1');
                   digest
--------------------------------------------
 \x0a0a9f2a6772942557ab5355d76af442f8f65e01
```

See the [`pgcrypto` docs](
http://www.postgresql.org/docs/current/static/pgcrypto.html) for more
details.

# Configure The Timezone 

Running `show timezone;` will reveal the timezone for your postgres
connection. If you want to change the timezone for the duration of the
connection, you can run something like

```
> set timezone='America/New_York';
SET
> show timezone;
 TimeZone
------------------
 America/New_York
(1 row)
```

Now, if you run a command such as `select now();`, the time will be in
Eastern time.

h/t Jack Christensen

# Constructing A Range Of Dates 

PostgreSQL offers a number of range types including the `daterange` type.
This can be constructed using the `daterange()` function with two strings
representing the lower and upper bounds of the date range respectively.

```sql
> select daterange('2015-1-1','2015-1-5');
        daterange
-------------------------
 [2015-01-01,2015-01-05)
```

The lower bound is inclusive -- indicated by the `[` character -- and the
upper bound is exclusive -- indicated by the `)` character.

[source](http://www.postgresql.org/docs/current/static/rangetypes.html)

# Count Records By Type 

If you have a table with some sort of type column on it, you can come up
with a count of the records in that table by type. You just need to take
advantage of `group by`:

```sql
> select type, count(*) from pokemon group by type;

  type   | count
-----------------
 fire    |    10
 water   |     4
 plant   |     7
 psychic |     3
 rock    |    12
```

# Create A Composite Primary Key 

The unique identifier for a given row in a table is the *primary key*.
Generally, a row can be uniquely identified by a single data point (such as
an id), so the primary key is simply that single data point. In some cases,
your data can be more appropriately uniquely identified by multiple values.
This is where composite primary keys can lend a hand. Consider an example
`plane_tickets` table where each ticket can be uniquely identified by the
passenger and flight it is associated with:

```sql
create table plane_tickets (
  passenger_id integer references passengers not null,
  flight_id integer references flights not null,
  confirmation_number varchar(6) not null,
  seat_assignment varchar not null,
  primary key (passenger_id, flight_id)
);
```

# Create hstore From Two Arrays 

PostgreSQL allows the use of the `hstore` type by enabling the `hstore`
extension. One way to create an instance of `hstore` data is by passing two
arrays to the `hstore()` function. The first array is a set of keys and the
second array is the set of values.

```sql
> select hstore(array['one','two','three'], array['1','2','3']);
                hstore
--------------------------------------
 "one"=>"1", "two"=>"2", "three"=>"3"
```

The two arrays must be the same length or an error will occur.

```sql
> select hstore(array['one','two','three'], array['1','2']);
ERROR:  arrays must have same bounds
```

# Creating Conditional Constraints 

There are times when it doesn't make sense for a constraint to apply to all
records in a table. For instance, if we have a table of pokemon, we may only
want to apply a unique index constraint to the names of non-wild pokemon.
This can be achieved with the following conditional constraint:

```sql
create unique index pokemons_names on pokemons (names)
where wild = false;
```

If we try to insert a non-wild pokemon with a duplicate name, we will get an
error. Likewise, if we try to update a pokemon with a duplicate name from
wild to non-wild, we will get an error.

[source](http://www.postgresguide.com/performance/conditional.html)

# Day Of Week By Name For A Date 

In [Day Of Week For A Date](day-of-week-for-a-date.md), I explained how to
determine what day of the week a date is as an integer with PostgreSQL. This
used the `date_part()` function. By using the `to_char()` function with a
date or timestamp, we can determine the day of the week by name (e.g.
Monday). For instance, to determine what day today is, try a statement like
the following:

```sql
> select to_char(now(), 'Day');
  to_char
-----------
 Sunday
```

The `Day` part of the second argument is just one of many template patterns
that can be used for formatting dates and times.

See [Data Type Formatting
Functions](http://www.postgresql.org/docs/current/static/functions-formatting.html)
in the Postgres docs for more details.

# Day Of Week For A Date 

Given a `date` in PostgreSQL

```sql
> select '2050-1-1'::date;
    date
------------
 2050-01-01
```

you can determine the day of the week for that date with the `date_part()`
function

```sql
> select date_part('dow', '2050-1-1'::date);
 date_part
-----------
         6
```

The days of week are `0` through `6`, `0` being Sunday and `6` being
Saturday.

[source](http://www.postgresql.org/docs/current/static/functions-datetime.html)

# Default Schema 

Schemas can be used to organize tables within a database. You can see all
the schemas your database has like so

```sql
> select schema_name from information_schema.schemata;
    schema_name
--------------------
 pg_toast
 pg_temp_1
 pg_toast_temp_1
 pg_catalog
 public
 information_schema
(6 rows)
```

When you create a new table, it will need to be placed under one of these
schemas. So if you have a `create table posts (...)`, how does postgres know
what schema to put it under?

Postgres checks your `search_path` for a default.

```sql
> show search_path;
   search_path
-----------------
 "$user", public
(1 row)
```

From our first select statement, we see that there is no schema with my user
name, so postgres uses public as the default schema.

If we set the search path to something that won't resolve to a schema name,
postgres will complain

```sql
> set search_path = '$user';
SET
> create table posts (...);
ERROR:  no schema has been selected to create in
```

# Defining Arrays 

In postgres, an array can be defined using the `array` syntax like so:

```sql
> select array['a','b','c'];
  array
---------
 {a,b,c}
```

If you are inserting into an existing array column, you can use the array
literal syntax.

```sql
> create temp table favorite_numbers(numbers integer[]);
CREATE TABLE
> insert into favorite_numbers values( '{7,3,9}' );
INSERT 0 1
> select numbers[2] from favorite_numbers;
 numbers
---------
       3
```

Postgres also supports two-dimensional arrays.

```sql
select array[[1,2,3],[4,5,6],[7,8,9]] telephone;
         telephone
---------------------------
 {{1,2,3},{4,5,6},{7,8,9}}
```

# Edit Existing Functions 

In the `psql` console, use `\ef` with the name of a function to fetch and
open the definition of the function. The function will be opened in your
system `$EDITOR` in the form of a `create or replace function` query.

Executing

```sql
> \ef now
```

will open the following in your default editor

```sql
CREATE OR REPLACE FUNCTION pg_catalog.now()
 RETURNS timestamp with time zone
 LANGUAGE internal
 STABLE STRICT
AS $function$now$function$
```

# Escaping A Quote In A String 

In PostgreSQL, string (`varchar` and `text`) literals are declared with
single quotes (`'`). That means that any string containing a single quote
will need some escaping. The way to escape a single quote is with another
single quote.

```sql
> select 'what''s up!';
  ?column?
------------
 what's up!
```

[source](http://jonathansacramento.com/posts/20160122-improve-postgresql-workflow-vim-dbext.html)

# Export Query Results To A CSV 

Digging through the results of queries in Postgres's `psql` is great if you
are a programmer, but eventually someone without the skills or access may
need to check out that data. Exporting the results of a query to CSV is a
friendly way to share said results because most people will have a program
on their computer that can read a CSV file.

For example, exporting all your pokemon to `/tmp/pokemon_dump.csv` can be
accomplished with:

```sql
copy (select * from pokemons) to '/tmp/pokemon_dump.csv' csv;
```

Because we are grabbing the entire table, we can just specify the table name
instead of using a subquery:


```sql
copy pokemons to '/tmp/pokemon_dump.csv' csv;
```

Include the column names as headers to the CSV file with the `header`
keyword:

```sql
copy (select * from pokemons) to '/tmp/pokemon_dump.csv' csv header;
```

[source](http://stackoverflow.com/questions/1120109/export-postgres-table-to-csv-file-with-headings)

# Extracting Nested JSON Data 

If you are storing nested JSON data in a postgres JSON column, you are
likely going to find yourself in a situation where you need to access some
of those nested values in your database code. For instance, you may need to
get at the license number in this JSON column

```sql
  owner
--------------------------------------------------------------------------------
'{ "name": "Jason Borne", "license": { "number": "T1234F5G6", "state": "MA" } }'
```

Unfortunately, the `->` operator isn't going to do the trick. You need the
`json_extract_path` function

```sql
> select json_extract_path(owner, 'license', 'number') from some_table;

 json_extract_path
-------------------
   'T1234F5G6'
```

Read more about [JSON Functions and
Operators](http://www.postgresql.org/docs/9.4/static/functions-json.html).

# Find The Data Directory 

Where does postgres store all of the data for a database cluster? Well, in
its data directory. Where exactly that data directory is can depend on how
the database cluster was setup. Postgres can tell you right where to look,
though. Within `psql`, run

```sql
> show data_directory
```

Postgres will output the absolute path of the data directory.

[source](http://dba.stackexchange.com/questions/1350/how-do-i-find-postgresqls-data-directory)

# Fizzbuzz With Common Table Expressions 

In learning about CTEs (common table expressions) in postgres, I discovered
that you can do some interesting and powerful things using the `with
recursive` construct. The following solves the fizzbuzz problem for integers
up to 100

```sql
with recursive fizzbuzz (num,val) as (
    select 0, ''
    union
    select (num + 1),
      case
      when (num + 1) % 15 = 0 then 'fizzbuzz'
      when (num + 1) % 5  = 0 then 'buzz'
      when (num + 1) % 3  = 0 then 'fizz'
      else (num + 1)::text
      end
    from fizzbuzz
    where num < 100
)
select val from fizzbuzz where num > 0;
```

Check out [With Queries (Common Table Expressions)](http://www.postgresql.org/docs/9.4/static/queries-with.html)
for more details on CTEs.

# Generate A UUID 

Postgres has support for universally unique identifiers (UUIDs) as a column
data type via `uuid`. If you have a UUID column, you may need to generate a
UUID.  This requires the `uuid-ossp` module. This module provides a number
of functions for generating UUIDs including the `uuid_generate_v4()`
function which bases the UUID entirely off random numbers.

```sql
> create extension "uuid-ossp";
CREATE EXTENSION
> select uuid_generate_v4();
           uuid_generate_v4
--------------------------------------
 a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11
```

See the (http://www.postgresql.org/docs/9.4/static/uuid-ossp.html) for more
details on UUID generation functions.

# Generate Series Of Numbers 

Postgres has a `generate_series` function that can be used to, well,
generate a series of something. The simplest way to use it is by giving it
`start` and `stop` arguments

```sql
> select generate_series(1,5);
 generate_series
-----------------
               1
               2
               3
               4
               5
```

The default step is 1, so if you want to count backwards, you need to
specify a negative step


```sql
> select generate_series(5,1,-1);
 generate_series
-----------------
               5
               4
               3
               2
               1
```

You can use a larger step value to, for instance, get only multiples of 3

```sql
> select generate_series(3,17,3);
 generate_series
-----------------
               3
               6
               9
              12
              15
```

Trying this out with timestamps is left as an exercise for the reader.

# Get The Size Of A Database 

If you have connect access to a PostgreSQL database, you can use the
`pg_database_size()` function to get the size of a database in bytes.

```sql
> select pg_database_size('hr_hotels');
 pg_database_size
------------------
          8249516
```

Just give it the name of the database and it will tell you how much disk
space that database is taking up.

Checkout [the Postgres docs](http://www.postgresql.org/docs/current/static/functions-admin.html) for more details.

# Get The Size Of A Table 

In [Get The Size Of A Database](get-the-size-of-a-database.md), I showed a
PostgreSQL administrative function, `pg_database_size()`, that gets the size
of a given database. With the `pg_relation_size()` function, we can get the
size of a given table. For instance, if we'd like to see the size of the
`reservations` table, we can executing the following query:

```sql
> select pg_relation_size('reservations');
 pg_relation_size
------------------
          1531904
```

This gives us the size of the `reservations` table in bytes. As you might
expect, the referenced table needs to be part of the connected database and
on the search path.

See [the Postgres docs](http://www.postgresql.org/docs/current/static/functions-admin.html) for more details.

# Getting A Slice Of An Array 

Postgres has a very natural syntax for grabbing a slice of an array. You
simply add brackets after the array declaring the lower and upper bounds
of the slice separated by a colon.

```sql
> select (array[4,5,6,7,8,9])[2:4];
  array
---------
 {5,6,7}
```

Notice that the bounds are inclusive, the array index is `1`-based, and the
array declaration above needs to be wrapped in parentheses in order to not
trip up the array slice syntax.

You can also select rectangular slices from two dimensional arrays like so:

```sql
> select (array[[1,2,3],[4,5,6],[7,8,9]])[2:3][1:2];
     array
---------------
 {{4,5},{7,8}}
```

# Insert Just The Defaults 

If you are constructing an `INSERT` statement for a table whose required
columns all have default values, you may just want to use the defaults. In
this situation, you can break away from the standard:

```
> insert into table_name (column1, column2) values (value1, value2);
```

Instead, simply tell Postgres that you want it to use the default values:

```
> insert into table_name default values;
```

# Integers In Postgres 

Postgres has three kinds of integers. Or rather three sizes of integers.
There are `smallint` (`int2`), `integer` (`int4`), and `bigint` (`int8`)
integers. As you might expect, they are 2 byte, 4 byte, and 8 byte integers
respectively. They are also signed integers. All of this has implications
for what ranges of integers can be represented by each type.

The `smallint` integers have 2 bytes to use, so they can be used to
represent integers from -32768 to +32767.

The `integer` integers have 4 bytes to use, so they can be used to represent
integers from -2147483648 to +2147483647.

The `bigint` integers have 8 bytes to use, so they can be used to represent
integers from -9223372036854775808 to +9223372036854775807.

Though columns can be restricted to use a particular-sized integer, postgres
is smart enough to default to `integer` and only use `bigint` as necessary
when working with integers on the fly.

```sql
> select pg_typeof(55);
 pg_typeof
-----------
 integer

> select pg_typeof(99999999999999999);
 pg_typeof
-----------
 bigint
```

# Intervals Of Time By Week 

It is pretty common to use hours or days when creating a Postgres
interval. However, intervals can also be created in week-sized chunks

```sql
> select '2 weeks'::interval;
 interval
----------
 14 days
(1 row)

> select make_interval(0,0,7,0,0,0,0);
 make_interval
---------------
 49 days
(1 row)
```

# Is It Null Or Not Null? 

In PostgreSQL, the standard way to check if something is `NULL` is like so:

```sql
select * as wild_pokemons from pokemons where trainer_id is null;
```

To check if something is not null, you just add `not`:

```sql
select * as captured_pokemons from pokemons where trainer_id is not null;
```

PostgreSQL also comes with `ISNULL` and `NOTNULL` which are non-standard
ways of doing the same as above:


```sql
select * as wild_pokemons from pokemons where trainer_id isnull;
```

```sql
select * as captured_pokemons from pokemons where trainer_id notnull;
```

# Limit Execution Time Of Statements 

You can limit the amount of time that postgres will execute a statement
by setting a hard timeout. By default the timeout is 0 (see `show
statement_timeout;`) which means statements will be given as much time as
they need.

If you do want to limit your statements, to say, 1 second, you can set the
execution time like so

```sql
> set statement_timeout = '1s';
SET
> show statement_timeout;
 statement_timeout
-------------------
 1s
(1 row)
```

Any queries taking longer than 1 second will be aborted with the following
message output

```
ERROR:  canceling statement due to statement timeout
```

# List All Columns Of A Specific Type 

We can access information about all the columns in our database through the
`information_schema` tables; in particular, the `columns` table. After
connecting to a particular database, we can list all columns (across all our
tables) of a specific type. We just need to know the schema of the tables we
are interested in and the type that we want to track down.

My application's tables are under the `public` schema and I want to track
down all `timestamp` columns. My query can look something like this

```sql
> select table_name, column_name, data_type from information_schema.columns where table_schema = 'public' and data_type = 'timestamp without time zone';
   table_name    | column_name |          data_type
-----------------+-------------+-----------------------------
 articles        | created_at  | timestamp without time zone
 articles        | updated_at  | timestamp without time zone
 users           | created_at  | timestamp without time zone
 users           | updated_at  | timestamp without time zone
(4 rows)
```

Alternatively, I could look for both `timestamp` and `timestamptz` with a
query like this

```sql
> select table_name, column_name, data_type from information_schema.columns where table_schema = 'public' and data_type like '%timestamp%';
```

# List All Versions Of A Function 

Within `psql` you can use `\df` to list a postgres function with a given
name

```sql
> \df now
                              List of functions
   Schema   | Name |     Result data type     | Argument data types |  Type
------------+------+--------------------------+---------------------+--------
 pg_catalog | now  | timestamp with time zone |                     | normal
(1 row)
```

When a function has multiple definitions across a number of types, `\df`
will list all versions of that function

```sql
> \df generate_series
List of functions
-[ RECORD 1 ]-------+-------------------------------------------------------------------
Schema              | pg_catalog
Name                | generate_series
Result data type    | SETOF bigint
Argument data types | bigint, bigint
Type                | normal
-[ RECORD 2 ]-------+-------------------------------------------------------------------
Schema              | pg_catalog
Name                | generate_series
Result data type    | SETOF bigint
Argument data types | bigint, bigint, bigint
Type                | normal
-[ RECORD 3 ]-------+-------------------------------------------------------------------
Schema              | pg_catalog
Name                | generate_series
Result data type    | SETOF integer
Argument data types | integer, integer
Type                | normal
-[ RECORD 4 ]-------+-------------------------------------------------------------------
Schema              | pg_catalog
Name                | generate_series
Result data type    | SETOF integer
Argument data types | integer, integer, integer
Type                | normal
-[ RECORD 5 ]-------+-------------------------------------------------------------------
Schema              | pg_catalog
Name                | generate_series
Result data type    | SETOF timestamp with time zone
Argument data types | timestamp with time zone, timestamp with time zone, interval
Type                | normal
-[ RECORD 6 ]-------+-------------------------------------------------------------------
Schema              | pg_catalog
Name                | generate_series
Result data type    | SETOF timestamp without time zone
Argument data types | timestamp without time zone, timestamp without time zone, interval
Type                | normal
```

# List Database Users 

Within `psql`, type `\du` to list all the users for a database and their
respective permissions.

```bash
> \du
                              List of roles
 Role name  |                   Attributes                   | Member of
------------+------------------------------------------------+-----------
 jbranchaud | Superuser, Create role, Create DB, Replication | {}
 sampleuser | Create DB                                      | {}
 ```

# Max Identifier Length Is 63 Bytes 

In PostgreSQL, identifiers -- table names, column names, constraint names,
etc. -- are limited to a maximum length of 63 bytes. Identifiers longer than
63 characters can be used, but they will be truncated to the allowed length
of 63.

```sql
> alter table articles
    add constraint this_constraint_is_going_to_be_longer_than_sixty_three_characters_id_idx
    check (char_length(title) > 0);
NOTICE:  identifier "this_constraint_is_going_to_be_longer_than_sixty_three_characters_id_idx" will be truncated to "this_constraint_is_going_to_be_longer_than_sixty_three_characte"
ALTER TABLE
```

Postgres warns us of identifiers longer than 63 characters, informing us of
what they will be truncated to. It then proceeds to create the identifier.

If postgres is trying to generate an identifier for us - say, for a foreign
key constraint - and that identifier is longer than 63 characters, postgres
will truncate the identifier somewhere in the middle so as to maintain the
convention of terminating with, for example, `_fkey`.

The 63 byte limit is not arbitrary. It comes from `NAMEDATALEN - 1`. By default
`NAMEDATALEN` is 64. If need be, this value can be modified in the Postgres
source. Yay, open-source database implementations.

See [the postgres docs](http://www.postgresql.org/docs/current/static/sql-syntax-lexical.html#SQL-SYNTAX-IDENTIFIERS) for more details.
# pg Prefix Is Reserved For System Schemas

Have you ever tried to create a schema with `pg_` as the first part of the
name of the schema? If so, you probably didn't get very far. Postgres won't
let you do that. It reserves the `pg_` prefix for system schemas. If you try
to create a schema in this way, you'll get an *unacceptable schema name*
error.

```sql
> create schema pg_cannot_do_this;
ERROR:  unacceptable schema name "pg_cannot_do_this"
DETAIL:  The prefix "pg_" is reserved for system schemas.
```

# Pretty Print Data Sizes 

Use the `pg_size_pretty()` function to pretty print the sizes of data in
PostgreSQL. Given a `bigint`, it will determine the most human-readable
format with which to print the value:

```sql
> select pg_size_pretty(1234::bigint);
 pg_size_pretty
----------------
 1234 bytes

> select pg_size_pretty(123456::bigint);
 pg_size_pretty
----------------
 121 kB

> select pg_size_pretty(1234567899::bigint);
 pg_size_pretty
----------------
 1177 MB

> select pg_size_pretty(12345678999::bigint);
 pg_size_pretty
----------------
 11 GB
```

This function is particularly useful when used with the
[`pg_database_size()`](get-the-size-of-a-database.md) and
[`pg_relation_size()`](get-the-size-of-a-table.md) functions.

```sql
> select pg_size_pretty(pg_database_size('hr_hotels'));
 pg_size_pretty
----------------
 12 MB
```

# Restart A Sequence 

In postgres, if you are truncating a table or doing some other sort of
destructive action on a table in a development or testing environment, you
may notice that the id sequence for the primary key just keeps plugging
along from where it last left off. The sequence can be reset to any value
like so:

```sql
> alter sequence my_table_id_seq restart with 1;
ALTER SEQUENCE
```

[source](http://www.postgresql.org/docs/current/static/sql-altersequence.html)

# Restarting Sequences When Truncating Tables 

PostgreSQL's
[`truncate`](http://www.postgresql.org/docs/current/static/sql-truncate.html)
feature is a handy way to clear out all the data from a table. If you use
`truncate` on a table that has a `serial` primary key, you may notice that
subsequent insertions keep counting up from where you left off. This is
because the sequence the table is using hasn't been restarted. Sure, you can
restart it manually or you can tell `truncate` to do it for you. By
appending `restart identity` to the end of a `truncate` statement, Postgres
will make sure to restart any associated sequences at `1`.

```sql
truncate pokemons, trainers, pokemons_trainers restart identity;
```

# Send A Command To psql 

You can send a command to `psql` to be executed by using the `-c` flag

```bash
$ psql -c "select 'Hello, World!';"
   ?column?
---------------
 Hello, World!
(1 row)
```

Specify a particular database as needed

```bash
$ psql blog_prod -c 'select count(*) from posts;'
 count
-------
     8
(1 row)
```

h/t Jack Christensen

# Set A Seed For The Random Number Generator 

In PostgreSQL, the internal seed for the random number generator is a
run-time configuration parameter. This `seed` parameter can be set to a
particular seed in order to get some determinism from functions that utilize
the random number generator. The seed needs to be something between `0` and
`1`.

We can see this in action by setting the seed and then invoking `random()` a
couple times. Doing this twice, we will see the reproducibility we can
achieve with a seed.

```sql
> set seed to 0.1234;
SET

> select random();
      random
-------------------
 0.397731185890734

> select random();
      random
------------------
 0.39575699577108
(1 row)

> set seed to 0.1234;
SET

> select random();
      random
-------------------
 0.397731185890734

> select random();
      random
------------------
 0.39575699577108
```

The seed can also be configured with the `setseed()` function.

See [the PostgreSQL
docs](http://www.postgresql.org/docs/8.3/static/sql-set.html) for more
details.

# Set Inclusion With hstore 

In PostgreSQL, `hstore` records can be compared via set inclusion. The `@>`
and `<@` operators can be used for this. The `@>` operator checks if the
right operand is a subset of the left operand. The `<@` operator checks if
the left operand is a subset of the right operand.

```sql
> select '"one"=>"1", "two"=>"2", "three"=>"3"'::hstore @> '"two"=>"2"'::hstore;
 ?column?
 ----------
  t

> select '"one"=>"1", "two"=>"2", "three"=>"3"'::hstore <@ '"two"=>"2"'::hstore;
 ?column?
----------
 f
```

See the [`hstore` PostgreSQL
docs](http://www.postgresql.org/docs/current/static/hstore.html) for more
details.

# Sleeping 

Generally you want your SQL statements to run against your database as
quickly as possible. For those times when you are doing some sort of
debugging or just want your queries to look very computationally expensive,
PostgreSQL offers the `pg_sleep` function.

To sleep for 5 seconds, try the following:

```sql
> select now(); select pg_sleep(5); select now();
              now
-------------------------------
 2016-01-08 16:30:21.251081-06
(1 row)

Time: 0.274 ms
 pg_sleep
----------

(1 row)

Time: 5001.459 ms
              now
-------------------------------
 2016-01-08 16:30:26.252953-06
(1 row)

Time: 0.260 ms
```

As you'll notice, the `pg_sleep` statement took about 5 seconds.

[source](http://www.if-not-true-then-false.com/2010/postgresql-sleep-function-pg_sleep-postgres-delay-execution/)

# Special Math Operators 

Postgres has all the mathematical operators you might expect in any
programming language (e.g. `+`,`-`,`*`,`/`,`%`). It also has a few extras
that you might not be expecting.

Factorial Operator:

```sql
> select 5!;
 ?column?
----------
      120
(1 row)
```

Square Root Operator:

```sql
> select |/81;
 ?column?
----------
        9
(1 row)
```

Absolute Value Operator:

```sql
> select @ -23.4;
 ?column?
----------
     23.4
(1 row)
```

# String Contains Another String 

You can check if a string *contains* another string using the `position`
function.

```sql
> select position('One' in 'One Two Three');
 position
----------
        1
```

It returns the 1-based index of the first character of the first match of
that substring.

```sql
> select position('Four' in 'One Two Three');
 position
----------
        0
```

If the substring doesn't appear within the string, then the result is 0.

Thus, you can determine if a string *contains* another string by checking if
the value resulting from `position` is greater than 0.

# Temporarily Disable Triggers 

In general, you are always going to want your triggers to fire. That's why
they are there. Though special circumstances may arise where you need to
temporarily disable them. Use

```sql
> set session_replication_role = 'replica';
SET
```

By changing the

[replication role](http://www.postgresql.org/docs/9.4/static/runtime-config-client.html#GUC-SESSION-REPLICATION-ROLE) 
from `origin` to
`replica` you are essentially disabling all non-replica triggers across the
database (for that session). When you are done, you can simply set the
replication role back so that normal trigger behavior can resume

```sql
> set session_replication_role = 'origin';
SET
```

A more direct and fine-grained approach to disabling triggers is to use an
`alter table` command that targets a specific trigger.

h/t Jack Christensen

# Temporary Tables 

Create a temporary table in Postgres like so

```sql
create temp table posts (
    ...
);
```

This table (and its data) will only last for the duration of the session.
It is created on a schema specific to temporary tables. It is also worth
noting that it won't be autovacuumed, so this must be done manually as
necessary.

# Timestamp Functions 

There are a handful of timestamp functions available in postgres. The most
common one is probably `now()`. This is an alias of
`transaction_timestamp()` which the postgres docs describe as:

> Current date and time (start of current transaction)

Two other interesting timestamp functions are `statement_timestamp()` and
`clock_timestamp()`. The postgres docs describe `statement_timestamp()` as:

> Current date and time (start of current statement)

Using `statement_timestamp()` throughout a transaction will yield different
results from statement to statement.

The postgres docs describe `clock_timestamp()` as:

> Current date and time (changes during statement execution)

Using `clock_timestamp()` may even yield different results depending on
where it appears in a given statement.

Try running something like this to see:

```postgresql
> select clock_timestamp(), clock_timestamp(), clock_timestamp(), clock_timestamp();
        clock_timestamp        |        clock_timestamp        |        clock_timestamp        |        clock_timestamp
-------------------------------+-------------------------------+-------------------------------+------------------------------
 2015-03-20 14:58:49.832592-05 | 2015-03-20 14:58:49.832592-05 | 2015-03-20 14:58:49.832593-05 | 2015-03-20 14:58:49.832593-05
```

You'll notice that we see a change in the clock time at the microsecond
level mid-way through the statement.

sources: (http://www.postgresql.org/docs/9.1/static/functions-datetime.html) and
[Jack C.](http://hashrocket.com/team/jack-christensen)

# Toggling The Pager In PSQL 

When the pager is enabled in `psql`, commands that produce larger output
will be opened in a pager. The pager can be enabled within `psql` by running
`\pset pager on`.

If you'd like to retain the output of commands, perhaps as reference for
subsequent commands, you can turn the pager off. As you might expect, the
pager can be disabled with `\pset pager off`.

[source](http://stackoverflow.com/questions/11180179/postgresql-disable-more-output)

# Truncate All Rows 

Given a postgres database, if you want to delete all rows in a table, you
can use the `DELETE` query without any conditions.

```sql
> delete from pokemons;
DELETE 151
```

Though `DELETE` can do the job, if you really are deleting all rows to clear
out a table, you are better off using `TRUNCATE`. A `TRUNCATE` query will be
faster than a `DELETE` query because it will just delete the rows without
scanning them as it goes.

```sql
> truncate pokemons;
TRUNCATE TABLE
```

[source](http://www.postgresql.org/docs/8.2/static/sql-truncate.html)

# Truncate Tables With Dependents 

In [Truncate All Rows](truncate-all-rows.md), I talked about how
postgres's `truncate` can be used to quickly delete all rows in a table. In
practice this alone won't be very useful though, because tables usually have
other tables that depend on them via foreign keys. If you have tables `A`
and `B` where `B` has a foreign key referencing `A`, then trying to truncate
`A` will result in something like this:

```sql
> truncate A;
ERROR:  cannot truncate a table referenced in a foreign key constraint
```

Fortunately, `truncate` has some tricks up its sleeve.

If you know two tables are tied together via a foreign key constraint, you
can just truncate both of them at once:

```sql
> truncate A, B;
TRUNCATE TABLE;
```

If many tables are tied together in this way and you are looking to throw
all of it out, then a simpler approach is to cascade the truncation:

```sql
> truncate A cascade;
NOTICE:  truncate cascades to table "B"
TRUNCATE TABLE
```

Use these with care and potentially within transactions because your data
will go bye bye.

# Turn Timing On 

When digging around your database and running queries, it is helpful to
have an eye on the speed of those queries. This can give insight into
where there are needs for optimizations.

Turn timing on (and off) within `psql` by running `\timing`. With timing
on, the duration of each query will be displayed in milliseconds after the
output of the query.

# Two Ways To Compute Factorial 

In PostgreSQL, there are two ways to compute the factorial of a number.
There is a prefix operator and a postfix operator. The prefix operator is
`!!` and can be used like so:

```sql
> select !!5;
 ?column?
----------
      120
```

The postfix operator is `!` and can be used like so:

```sql
> select 5!;
 ?column?
----------
      120
```

See the [mathematical functions and operators
docs](http://www.postgresql.org/docs/8.1/static/functions-math.html)
for more details.

# Types By Category 

Postgres has many types, each of which fall into a particular category.
These categories include Array, Boolean, String, Numeric, Composite, etc.
Each of these categories has a corresponding code. For instance, numeric
types have a code of `N`. Using `N` I can get a list of all the numeric
types:

```sql
> select typname from pg_type where typcategory = 'N';
     typname
-----------------
 int8
 int2
 int4
 regproc
 oid
 float4
 float8
 money
 numeric
 regprocedure
 regoper
 regoperator
 regclass
 regtype
 regconfig
 regdictionary
 cardinal_number
(17 rows)
```

Check out
[`pg_type`](http://www.postgresql.org/docs/current/interactive/catalog-pg-type.html)
in the Postgres docs for a list of all categories and codes.

# Use A psqlrc File For Common Settings 

There are a handful of settings that I inevitably turn on or configure each
time I open up a `psql` session. I can save myself a little time and sanity
by configuring these things in a `.psqlrc` dotfile that is located in my
home directory. This will ensure my `psql` session is configured just how I
like it each time I launch it. Here is what my `~/.psqlrc` file currently
looks like:

```
\x auto
\timing
\pset null 'Ø'
```

# Use Argument Indexes 

In Postgres, each of the arguments you specify in a `select` statement has a
1-based index tied to it. You can use these indexes in the `order by` and
`group by` parts of the statement.

Instead of writing

```sql
select id, updated_at from posts order by updated_at;
```

you can write

```sql
select id, updated_at from posts order by 2;
```

If you want to group by a table's `type` and then order by the counts from
highest to lowest, you can do the following

```sql
select type, count(*) from transaction group by 1 order by 2 desc;
```

# Using Intervals To Offset Time 

Postgres Intervals can be used with time as a way of determining a
standard offset. For instance, I can concisely determine what the time was 2
hours earlier with

```sql
> select now() - '2 hours'::interval as earlier;
            earlier
-------------------------------
 2015-06-12 21:17:43.678822-05
```

or similarly

```sql
> select now() - interval '2 hours' as earlier;
            earlier
-------------------------------
 2015-06-12 21:17:43.678822-05
```

# Who Is The Current User 

You can determine the current user of a psql session by selecting on the `current_user`

```sql
> select current_user;

  current_user
----------------
   test_user
```

You can also select on the `user` which is an alias of `current_user`

```sql
> select user;

     user
----------------
   test_user
```

# Word Count for a Column 

Assuming I have a database with a posts table:

```sql
> select * from posts where id = 1;
 id |  title   |              content
----+----------+------------------------------------
  1 | My Title | This is the content of my article.
```

I can compute the word count of the content of a given post like so:

```sql
> select sum(array_length(regexp_split_to_array(content, '\s+'), 1)) from posts where id = 1;
 sum
-----
   7
```

[source](http://blog.lingohub.com/2013/07/sql-word-count-character-count-postgres/)
