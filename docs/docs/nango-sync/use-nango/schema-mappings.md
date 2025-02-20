# Schema mappings

Nango offers a few different ways to transform and access the synced data in your database:
- Automatic JSON to SQL schema mapping
- Access to the raw JSON object 
- Custom, manually configured schema mappings (coming soon)

## Automatic JSON-to-SQL schema mapping

:::info
Automatically inferring a schema from API responses is tricky. If you run into issues or want to understand why your schema came out the way it did we are happy to help you in the [Slack community](https://nango.dev/slack)!

In the near future we will also support [custom mappings](#custommapping) which will give you full control over the destination schema of Nango's mapping.
:::

### How Nango determines the data schema
By default, Nango automatically maps the JSON objects returned from external APIs to SQL columns. The mapping rules are:
- Nested fields are flattened and the path is joined with `_` into a single column name
- Arrays are flattened into multiple columns with suffix `_[index]`
- Null values are ignored
- Data types are inferred, but currently only for these supported types: string, number, date, boolean

Here is an example: 

The following JSON response:

```json
{
  "field": true,
  "parent": {"nested": "string_value"},
  "nullField": null,
  "list": [1, 2]
}
```

turns into this SQL table: 

| field (boolean) | parent_nested (string)      | list_0 (number) | list_1 (number) |
| ----------- |----------- | ----------- | ----------- |
| true | string_value      | 1       | 2       |

### How Nango treats data that does not align with the schema
Once a schema with data types has been generated, Nango will only store values in the SQL table that align with the data type of the schema.
As an example, let's assume the field `num_users` has type number and these objects get returned by the API:
```json
[
  {
    "name": "obj1",
    "num_users": 23
  },
  {
    "name": "obj2",
    "num_users": 182
  },
  {
    "name": "obj3",
    "num_users": "nango is great"
  }
]
```
In this case Nango would store the value of `num_users` for obj1 and obj2, but not for obj3 (because the type string is not compatible with the schema's type number).
  
### How Nango deals with schema changes
Nango will also change the schema of the generated table if the schema of the API response changes.
Currently the following transformations are supported: 
- If a previously unseen field appears in the JSON, the relevant SQL column will be created (with the right data type)
- If a previously seen field is not present in the JSON nothing happens (if a field is there we store it's value, if its not there we just ignore that)
  

### How to disable Auto Mapping
Auto mapping is on by default for new Syncs. You can disable Auto Mapping for an individual Sync by setting the `auto_mapping` field to `false` in the [Sync config options](sync-all-options.md).

## Configuring a Sync's destination table {#destination-table}

You can configure a Sync's destination database table with the `mapped_table` parameter in the [Sync config options](sync-all-options.md). 

You can also configure multiple Syncs to send data to the same destination table, for this just pass the same value to the `mapped_table` parameter for these Syncs.

If you specify a table that does not already exist, it will be automatically generated. The data schema for this table will be automatically updated (according to the rules outlined above) based on the data to insert.

If you only specify a table name in the `mapped_table` parameter, e.g. `pokemon`, Nango will use (or create) the table in the [default schema of your Postgres](https://www.postgresql.org/docs/current/ddl-schemas.html) database (by default this is called `public`). If you want to use a table in a different schema you can tell this Nango by prefixing the table name with the schema name: For example, setting `mapped_table` to `myschema.pokemons` sends the synced data to the`pokemons` table in the database schema `myschema`.


## Custom Mapping (coming soon) {#custommapping}

We plan to introduce custom mappings soon. These will allow you to specify exactly (in code) how you want the JSON mapped to a SQL-table.
This will enable a few interesting features:
- Stable SQL schemas that are guaranteed not to change even as the API response changes (with optional alerts for response changes)
- The ability to specify which fields should be extracted from the JSON (and which should be ignored)
- Optional, more complex transformations & mappings (e.g. combining data from multiple JSON-fields into one SQL column, transforming values etc.)


## Accessing the raw JSON objects

Nango stores all the objects, in their original JSON form, in a combined SQL table called `_nango_raw` within the schema `nango`.