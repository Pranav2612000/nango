# DB configuration

## Supported databases

Currently, Nango supports only Postgres databases as a destination for the synced data. You can request support for more database types on this [issue](https://github.com/NangoHQ/nango-sync/issues/68).

## Where Nango stores data

Nango stores two kinds of data in the Postgres database:
- Nango internal data (sync configurations, jobs state etc.)
- The synced data

Whilst both of these are stored in the same Postgres database you can control where Nango stores the synced data, so that you can separate it from the Nango internal data.

### Nango internal data: Stored in the `nango` schema
Configuration objects necessary for Nango's execution (e.g. Syncs and jobs) are stored in the `nango` [database schema](https://www.postgresql.org/docs/current/ddl-schemas.html).

The raw synced data, in its original JSON format, is also stored in the nango schema in a table called `_nango_raw`. This single table contains the raw data from all Syncs combined.

### Synced data: Stored where you specify
When you add a Sync to Nango you can [tell it where to store the data](schema-mappings.md#destination-table) for this sync with the `mapped_table` config option. Multiple Syncs can send data to the same table or different ones and you can specify both the table name as well as the schema to be used.

This parameter is optional, in the (rare) case where you do not specify it Nango will create a new table called `_nango_sync_[SYNC-ID]` (`[SYNC-ID]` is the nango internal id of your sync, e.g. 1638) in the default schema of your postgres database.

## Specifying the Postgres database {#custom-database}

By default, Nango assumes there is a local Postgres database available with the following options (the default `docker compose` setup provides this one): 
```
host: localhost
port: 5432
user: nango
password: nango
database: nango
```

You can point Nango to a different database by adding the following environment variables to the `.env` file (in the root folder):

```
NANGO_DB_HOST=[your-host]
NANGO_DB_PORT=[your-port]
NANGO_DB_USER=[your-user]
NANGO_DB_NAME=[your-database-name]
NANGO_DB_PASSWORD=[your-password]
NANGO_DB_SSL=TRUE # Set to 'TRUE' if database requires SSL connections
```

If the `nango` schema (where Nango stores its internal config) does not yet exist in the database you specify it will get created the first time Nango runs.