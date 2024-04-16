# BigQuery Functions
The following GoogleSQL functions are deployed to transform and augment inbound data.  The original published article can be found on Medium [here](https://medium.com/decode-data/essential-sql-functions-for-the-ga4-bigquery-events-export-adc07bcefc11).

### FORMAT_DATE
The [`FORMAT_DATE`](https://cloud.google.com/bigquery/docs/reference/standard-sql/date_functions#format_date) function is used to convert a `DATE` value into a `STRING` with a specific [date syntax](https://cloud.google.com/bigquery/docs/reference/standard-sql/format-elements#format_elements_date_time). In this case the `'%Y%m%d'` date syntax formats the output into a string in the format `YYYYMMDD`.

```sql
SELECT FORMAT_DATE('%Y%m%d', '2024-01-07')
```

```sql
> 20240107
```

### PARSE_DATE
The [`PARSE_DATE`](https://cloud.google.com/bigquery/docs/reference/standard-sql/date_functions#parse_date) function is the inverse of the `FORMAT_DATE` function, and is used to convert a `STRING` value into a `DATE` value, given an expected [date syntax](https://cloud.google.com/bigquery/docs/reference/standard-sql/format-elements#format_elements_date_time) in the input STRING value. 

```sql
SELECT PARSE_DATE('%Y%m%d', '20240107')
```

```sql
> 2024-01-07
```

### _TABLE_SUFFIX

The `_TABLE_SUFFIX` is technically a pseudo-column (a metadata column which is hidden but can be queried), which can be used in conjunction with a [wildcard query](https://cloud.google.com/bigquery/docs/querying-wildcard-tables) and the [`FORMAT_DATE`](https://cloud.google.com/bigquery/docs/reference/standard-sql/date_functions#format_date) function to efficiently query the `events_YYYYMMDD` table. Note that this query assumes that the `start_date` and `end_date` `DATE` variables have been set as script variables or table function arguments.

```sql
SELECT 
_TABLE_SUFFIX AS table_suffix, *
FROM `[project_id].analytics_#########].events_*`
WHERE _TABLE_SUFFIX BETWEEN 
FORMAT_DATE("%Y%m%d", start_date) AND 
FORMAT_DATE("%Y%m%d", end_date)
```

This query syntax returns all source table columns with the `_TABLE_SUFFIX` as a `STRING` column called `table_suffix` _before_ the rest of the columns.

### TO_JSON_STRING

The [`TO_JSON_STRING`](https://cloud.google.com/bigquery/docs/reference/standard-sql/json_functions#to_json_string) function returns a JSON-formatted `STRING` representation of the data passed to it as an argument. This argument can be a variable, column value or the name of a preceding common table expression, as in the example below.

```sql
WITH
add_ids AS (
  SELECT 
  'project_id' AS project_id,
  'analytics_123456789' AS dataset_name)

SELECT 
TO_JSON_STRING(add_ids) AS add_ids_json
FROM add_ids
```

```json
> {"project_id":"project_id","dataset_name":"analytics_123456789"}
```

`NULL` values are also preserved in the output, which makes it robust to unpredictable input data. This can be observed in the output from the example below.

```sql
WITH
add_ids AS (
  SELECT 
  'project_id' AS project_id,
  'analytics_123456789' AS dataset_name,
  NULL AS example_value)

SELECT 
TO_JSON_STRING(add_ids) AS add_ids_json
FROM add_ids
```

```json
> {"project_id":"project_id","dataset_name":"analytics_123456789","example_value":null}
```

`ARRAY` and `STRUCT` values also directly map to `JSON`, so _any_ BigQuery data structure can be represented in `JSON`.

Note that the function [`TO_JSON`](https://cloud.google.com/bigquery/docs/reference/standard-sql/json_functions#to_json) can be used interchangeably with this function, in which case the function will return a [`JSON`](https://cloud.google.com/bigquery/docs/json-data) value instead of a JSON-formatted `STRING` value. 


### SHA256
The [`SHA256`](https://cloud.google.com/bigquery/docs/reference/standard-sql/hash_functions#sha256) function is a hashing function, which returns a 44 character [`BYTES`](https://cloud.google.com/bigquery/docs/reference/standard-sql/data-types#bytes_type) value based on an input `STRING` or `BYTES` of any length. For a given input it will always return the same output, which makes it a useful function to check for uniqueness or identicality of inputs. Adding this to the example above gives a unique id for each row of data in the `add_ids` common table expression.

```sql
WITH
add_ids AS (
  SELECT 
  'project_id' AS project_id,
  'analytics_123456789' AS dataset_name,
  NULL AS example_value)

SELECT 
SHA256(TO_JSON_STRING(add_ids)) AS row_id
FROM add_ids
```

```plain
> XBbBYHTigXnQLJwZQ9dJ0y8TJexcWfhBiJWfp2pdVfE=
```

### TO_HEX
The [TO_HEX](https://cloud.google.com/bigquery/docs/reference/standard-sql/string_functions#to_hex) function is a function which converts a sequence of `BYTES` (as returned by the `SHA256` function above) into a hexadecimal `STRING`. This means that the resulting `STRING` will only contain the characters `0..9` and `a..f`, making it cleaner for certain additional operations and copy/paste actions in the user interface.

```sql
WITH
add_ids AS (
  SELECT 
  'project_id' AS project_id,
  'analytics_123456789' AS dataset_name,
  NULL AS example_value)

SELECT 
TO_HEX(SHA256(TO_JSON_STRING(add_ids))) AS row_id
FROM add_ids
```

```plain
> 5c16c16074e28179d02c9c1943d749d32f1325ec5c59f84188959fa76a5d55f1
```

### COALESCE
The [COALESCE](https://cloud.google.com/bigquery/docs/reference/standard-sql/conditional_expressions#coalesce) function returns the first non-null element from a sequence of values. Note that all elements in the sequence need to be coercible to a common [supertype](https://cloud.google.com/bigquery/docs/reference/standard-sql/conversion_rules#supertypes).

```sql
SELECT COALESCE (NULL,"value_x","value_y", NULL) AS value
```

```sql
> value_x
```

### SAFE_CAST
The [SAFE_CAST](https://cloud.google.com/bigquery/docs/reference/standard-sql/functions-and-operators#safe_casting) function is the `SAFE` version of the [CAST](https://cloud.google.com/bigquery/docs/reference/standard-sql/functions-and-operators#cast) function, meaning that if the input value cannot be `CAST` to the target data type, it will return `NULL` value instead of an error. Note that for many other functions this behaviour can be enforced by using the [SAFE. prefix](https://cloud.google.com/bigquery/docs/reference/standard-sql/functions-reference#safe_prefix).

```sql
SELECT SAFE_CAST("value_x" AS INT64) AS value
```

```sql
> null
```

Conversely, if using the CAST function, the function will error.

```sql
SELECT CAST("value_x" AS INT64) AS value
```

```plain
> Query error: Bad int64 value: value_x at [1:1]
```

### TIMESTAMP_MICROS
The [TIMESTAMP\_MICROS](https://cloud.google.com/bigquery/docs/reference/standard-sql/timestamp_functions#timestamp_micros) function converts a 16-digit unix timestamp (microseconds) to a `TIMESTAMP`.

```sql
SELECT TIMESTAMP_MICROS(1711353064567254) 
```

```sql
> 2024-03-25 07:51:04.567254 UTC
```

### DATE
The [DATE](https://cloud.google.com/bigquery/docs/reference/standard-sql/date_functions#date) function extracts the `DATE` from a `TIMESTAMP`. Used in conjunction with the `TIMESTAMP_MICROS` function, it enables conversion from a unix timestamp to a `DATE`.

```sql
SELECT DATE(TIMESTAMP_MICROS(1711353064567254));
```

```sql
> 2024-03-25
```

### TIME
The [TIME](https://cloud.google.com/bigquery/docs/reference/standard-sql/time_functions#time) function extracts the `TIME` (independent of the date) from a `TIMESTAMP`. Used in conjunction with the `TIMESTAMP_MICROS` function, it enables conversion from a unix timestamp to a `TIME`.

```sql
SELECT TIME(TIMESTAMP_MICROS(1711353064567254));
```

```sql
> 07:51:04.567254
```

### REGEXP_CONTAINS
The [REGEXP_CONTAINS](https://cloud.google.com/bigquery/docs/reference/standard-sql/string_functions#regexp_contains) function returns a `BOOL`, evaluating to `TRUE` if the input `STRING` matches the input [regular expression](https://github.com/google/re2/wiki/Syntax):

```plain
SELECT REGEXP_CONTAINS('somebody@email.com', r'@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+') AS email_is_valid
```

```plain
> true
```

If the `STRING` does not match the [regular expression](https://github.com/google/re2/wiki/Syntax), it will evaluate to `FALSE`.

```sql
SELECT REGEXP_CONTAINS('not_an_email.com', r'@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+') AS email_is_valid
```

```plain
> false
```

One important characteristic to note is that if either function argument is `NULL`, the expression will evaluate to `NULL`.

```sql
SELECT REGEXP_CONTAINS(NULL , r'@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+') AS email_is_valid
```

```plain
> null
```

```sql
SELECT REGEXP_CONTAINS('somebody@email.com', NULL) AS email_is_valid
```

```plain
> null
```