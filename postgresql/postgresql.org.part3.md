it is advisable to run PostgreSQL under a separate user account. This user account should only own the data that is managed by the server, and should not be shared with other daemons.

Before you can do anything, you must initialize a database storage area on disk. A database cluster is a collection of databases that is managed by a single instance of a running database server. It is completely up to you where you choose to store your data. There is no default, although locations such as `/usr/local/pgsql/data` or `/var/lib/pgsql/data` are popular. The data directory must be initialized before being used

```
$ initdb -D /usr/local/pgsql/data
```

Before anyone can access the database, you must start the database server `postgres` in the background.

```
$ postgres -D /usr/local/pgsql/data >logfile 2>&1 &
```

This shell syntax can get tedious quickly. Therefore the wrapper program `pg_ctl` is provided to simplify some tasks. 

the server must be run by the PostgreSQL user account and not by root or any other user. 

```
$ su postgres -c 'pg_ctl start -D /usr/local/pgsql/data -l serverlog'
```

Because of the way that the kernel implements memory overcommit, the kernel might terminate the PostgreSQL postmaster (the supervisor server process) if the memory demands of either PostgreSQL or another process cause the system to run out of virtual memory. See

https://www.postgresql.org/docs/current/kernel-resources.html#LINUX-MEMORY-OVERCOMMIT

https://www.postgresql.org/docs/current/encryption-options.html

https://www.postgresql.org/docs/current/auth-pg-hba-conf.html

In order to bootstrap the database system, a freshly initialized system always contains one predefined login-capable role. This role is always a “superuser”, and it will have the same name as the operating system user that initialized the database cluster with initdb unless a different name is specified. This role is often named postgres. In order to create more roles you first have to connect as this initial role.

A small number of objects, like role, database, and tablespace names, are defined at the cluster level and stored in the pg_global tablespace. Inside the cluster are multiple databases, which are isolated from each other but can access cluster-level objects. Inside each database are multiple schemas, which contain objects like tables and functions. So the full hierarchy is: cluster, database, schema, table 

When connecting to the database server, a client must specify the database name in its connection request. It is not possible to access more than one database per connection.

While individual databases in the cluster are isolated when considered from the user's perspective, they are closely bound from the database administrator's point-of-view.

```
SELECT datname FROM pg_database; -- list of dbs

CREATE DATABASE dbname OWNER rolename;
-- or 
createdb -O rolename dbname
```

It is the privilege of the owner of a database to remove it later. A role must be explicitly given permission to create databases (except for superusers, since those bypass all permission checks). Only the superuser is allowed to create a database for someone else.

Two additional databases, `template1` and `template0`, are also created during database cluster initialization. Whenever a new database is created within the cluster, `template1` is essentially cloned. This means that any changes you make in `template1` are propagated to all subsequently created databases. Only the superuser is allowed to create a database for someone else

Tablespaces in PostgreSQL allow database administrators to define locations in the file system where the files representing database objects can be stored. if the partition or volume on which the cluster was initialized runs out of space and cannot be extended, a tablespace can be created on a different partition and used until the system can be reconfigured. an index which is very heavily used can be placed on a very fast, highly available disk
```
CREATE TABLESPACE fastspace LOCATION '/ssd1/postgresql/data';
```
Creation of the tablespace itself must be done as a database superuser, but after that you can allow ordinary database users to use it. To do that, grant them the `CREATE` privilege on it.
```
CREATE TABLE foo(i int) TABLESPACE space1;
```

Locale support refers to an application respecting cultural preferences regarding alphabets, sorting, number formatting, etc. PostgreSQL uses the standard ISO C and POSIX locale facilities provided by the server operating system
```
initdb --locale=sv_SE
```
What locales are available on your system under what names depends on what was provided by the operating system vendor and what was installed. On most Unix systems, the command locale -a will provide a list of available locales.

https://www.postgresql.org/docs/current/locale.html

PostgreSQL, like any database software, requires that certain tasks be performed regularly to achieve optimum performance. It is the database administrator's responsibility to set up appropriate scripts, and to check that they execute successfully.

[check_postgres](https://bucardo.org/check_postgres/) is available for monitoring database health and reporting unusual conditions. 

PostgreSQL databases require periodic maintenance known as vacuuming. For many installations, it is sufficient to let vacuuming be performed by the autovacuum daemon, 

You might need to adjust the autovacuuming parameters described there to obtain best results for your situation. Some database administrators will want to supplement or replace the daemon's activities with manually-managed `VACUUM` commands

One possible compromise is to set the daemon's parameters so that it will only react to unusually heavy update activity, thus keeping things from getting out of hand, while scheduled VACUUMs are expected to do the bulk of the work when the load is typical at  night. If you have multiple databases in a cluster, don't forget to VACUUM each one;

If you have a table whose entire contents are deleted on a periodic basis, consider doing it with `TRUNCATE` rather than using `DELETE` followed by `VACUUM`. `TRUNCATE` removes the entire content of the table immediately, without requiring a subsequent `VACUUM` or `VACUUM FULL` to reclaim the now-unused disk space. The disadvantage is that strict MVCC semantics are violated.

The PostgreSQL query planner relies on statistical information about the contents of tables in order to generate good plans for queries. The autovacuum daemon, if enabled, will automatically issue `ANALYZE` commands whenever the content of a table has changed sufficiently. The daemon schedules `ANALYZE` strictly as a function of the number of rows inserted or updated; it has no knowledge of whether that will lead to meaningful statistical changes.

Tuples changed in partitions and inheritance children do not trigger analyze on the parent table. If the parent table is empty or rarely changed, it may never be processed by autovacuum, and the statistics for the inheritance tree as a whole won't be collected. It is necessary to run `ANALYZE` on the parent table manually in order to keep the statistics up to date.

The autovacuum daemon does not issue `ANALYZE` commands for foreign tables. If your queries require statistics on foreign tables for proper planning, it's a good idea to run manually-managed `ANALYZE` commands on those tables on a suitable schedule.

https://www.postgresql.org/docs/current/routine-vacuuming.html

In some situations it is worthwhile to rebuild indexes periodically with the `REINDEX` command.

If you simply direct the stderr of `postgres` into a file, you will have log output, but the only way to truncate the log file is to stop and restart the server. This might be acceptable if you are using PostgreSQL in a development environment, but few production servers would find this behavior acceptable.

https://www.postgresql.org/docs/current/logfile-maintenance.html

There are three fundamentally different approaches to backing up PostgreSQL data:

SQL dump, File system level backup, Continuous archiving

https://www.postgresql.org/docs/current/backup.html

https://www.postgresql.org/docs/current/high-availability.html

