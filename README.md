# mysqldump Tools

## mysqldumpfilter

Filter out parts of a mysqldump file. Optionally skip tables (creates and/or inserts) and/or views, By default removes any `USE DB_NAME` statements, will include any predefined files in the directory as specified in the help, and will before any inserts remove any index keys and at the end of inserts add back the index keys to that table for any speed gain.

Written in awk with a bash wrapper.

### Usage

Pipe the contents of the dump file to the mysqldumpfilter command.

* View help:

```
./mysqldumpfilter --help
```

* On the fly, filter an uncompressed dump straight from the compressed dump file into your database of choice `DB_NAME_DIFFERENT_THAN_IN_DUMP_FILE` and monitor progress using `pv`:

```
pv dump.sql.gz | gunzip -c | ./mysqldumpfilter | mysql -u USER -p DB_NAME_DIFFERENT_THAN_IN_DUMP_FILE
```

* Only import tables that begin with table prefix `prefix_` into `DB_NAME`:

```
gunzip -c dump.sql.gz | ./mysqldumpfilter --onlyTablesInList=prefix_* | mysql -u USER -p DB_NAME
```

* To recover from an sql error on a table, restart import starting with the next table `next_table` (without having to reimport all the previous tables before `next_table`):

```
gunzip -c dump.sql.gz | ./mysqldumpfilter --skipTablesUntilTable=next_table | mysql -u USER -p DB_NAME
```

* Save to file `out.sql` just the views in the dump file for later viewing/processing:

```
gunzip -c dump.sql.gz | ./mysqldumpfilter --skipTables > out.sql
```
