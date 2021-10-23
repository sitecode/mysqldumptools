# mysqldump Tools

## mysqldumpabout

Find out about a mysqldump file. Mysqldump and mysql versions, database name, the date the dump completed, table and view names and the number of insert statements per table. Default output is in JSON format.

Written in awk with a bash wrapper.

### Usage

Pipe the contents of the dump file to the mysqldumpabout command.

* View help:

```
./mysqldumpabout --help
```

* On the fly, find out about an uncompressed dump straight from the compressed dump file and monitor progress using `pv`, save output in json for later viewing/processing:

```
pv dump.sql.gz | gunzip -c | ./mysqldumpabout > dump.sql.gz.json
```

* Find out about an uncompressed dump, with text output (not the default JSON output):

```
gunzip -c dump.sql.gz | ./mysqldumpabout --text | less
```

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
gunzip -c dump.sql.gz | ./mysqldumpfilter --skipTables=1 --skipTriggers=1 > out.sql
```

* Import everything into `DB_NAME` but rewrite all trigger definers as USER@localhost the only existing user in that db:

```
gunzip -c dump.sql.gz | ./mysqldumpfilter --replaceTriggerDefiner '*' '`USER`@`localhost`' | mysql -u USER -p DB_NAME
```

#### Workaround entering MySQL password when using pv

Use mysql option `--defaults-extra-file` to specify user.cnf file with user/password in it. When using this option do not also specify user with `-u`. 

Create a file named `user.cnf` with permissions `600` (so only your user can read it) and with content:

```
[client]
username=USERNAME
password=PASSWORD
```

Then use like:

```
mysql --defaults-extra-file=./PATH/TO/user.cnf DB_NAME
```
