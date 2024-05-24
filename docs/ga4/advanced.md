Additional advanced options are available to customise the custom model deployment, enabling explicit inclusion or exclusion of `event_name` values and `event_param`/`user_property` keys, in addition to other profiling options.

There are a number of reasons why you might want to pre-filter the data stream.  This could be because of historical setup issues, to exclude values defined during a testing period, automated migration from an external tool or Google, or simply because the `event_names`, `event_params` or `user_properties` you use have evolved over time. 

We provide a mechanism to pre-filter the stream to implement a variety of logical scenarios.

## Event Options
Advanced filters are implemented via the `event_options JSON` variable, which is passed as a `NULL` `JSON` value to the `decodedata.region.install_ga4_decoder` function in the default installation. `JSON` paths are parsed from the `event_options` variable, the schema for which will comprise a subset of the following `JSON` keys:

```json
{
    "start_date": "<[DATE]>",
    "end_date": "<[DATE]>",
    "stream_id_in": ["<ARRAY<STRING>"],
    "stream_id_not_in": ["<ARRAY<STRING>"],
    "event_name_like_any": ["<ARRAY<STRING>"],
    "event_name_not_like_any": ["<ARRAY<STRING>"],
    "event_param_like_any": ["<ARRAY<STRING>"],
    "event_param_not_like_any": ["<ARRAY<STRING>"],
    "user_property_like_any": ["<ARRAY<STRING>"],
    "user_property_not_like_any" ["<ARRAY<STRING>"]
}
```

Note that the placeholders need to be replaced by a valid `JSON` value or array. Placeholder `"<[DATE]>"` would be replaced by e.g. `"2024-01-01"` and placeholder `["<ARRAY<STRING>"]` would be replaced by e.g. `["12345678", "87654321"]`.

## Data Profile Filters
Note that in each logical case, events are _not_ filtered out, simply excluded from the profiling and therefore not included in the relevant output `STRUCT` (`event_count`, `event_param`, `user_property`).  This means that the row count for the result of a query against the `[dataset_id].GA4_EVENTS` date-bounded table function for a specific date range should _precisely_ match the row count in both the source GA4 table shard range `[dataset_id].events_*` _and_ the output table `[dataset_id].EVENTS`.

### Date Range
Date ranges filter the specific date range used to build the custom decoder functions, only including columns in the decoded model which are observed in the defined date range (in addition to the standard set of columns). Neither or both `start_date` and `end_date` need to be passed in order for the deployment to be executed. 

!!! info "advanced event_options: `start_date, end_date`"
    === "JSON"
        ```json
        {
            "start_date": "2023-01-01",
            "end_date": "2023-12-31"
        }
        ```

    === "GoogleSQL (Static)"
        ```sql
        DECLARE event_options JSON; 

        SET event_options = JSON """
        {
            "start_date": "2023-01-01",
            "end_date": "2023-12-31"
        };
        """;
        ```

    === "GoogleSQL (Dynamic)"
        ```sql
        DECLARE event_options JSON; 

        SET event_options = SELECT TO_JSON(( 
        SELECT AS STRUCT
        CURRENT_DATE - 365 AS start_date,
        CURRENT_DATE AS end_date));
        ```

### Stream ID
Filtering on `stream_id` is typically only required if there have been setup irregularities, which are more commonly observed on Firebase data streams rather than GA4.

#### stream_id_in
This will _only_ profile rows where the `stream_id` matches one defined in the `stream_id_in ARRAY<STRING>`:

!!! info "advanced event_options: `stream_id_in`"
    === "JSON"
        ```json
        {
            "stream_id_in": ["12345678"]
        }
        ```

    === "GoogleSQL"
        ```sql
        DECLARE event_options JSON; 

        SET event_options = JSON """
        {
            "stream_id_in": ["12345678"]
        };
        """
        ```

#### stream_id_not_in
This will _only_ profile rows where the `stream_id` does not match any defined in the `stream_id_in ARRAY<STRING>`:

!!! info "advanced event_options: `stream_id_not_in`"
    === "JSON"
        ```json
        {
            "stream_id_not_in": ["87654321", "98765432"]
        }
        ```

    === "GoogleSQL"
        ```sql
        DECLARE event_options JSON; 

        SET event_options = JSON """
        {
            "stream_id_not_in": ["87654321", "98765432"]
        };
        """
        ```

### Event Names
#### event_name_like_any
The `event_name` profile include filter is implemented using the `LIKE ANY` [Quantified Like Operator](https://cloud.google.com/bigquery/docs/reference/standard-sql/functions-and-operators#like_operator_quantified). It will only include values in the `event_name` profile and decoder which match one of the values in the `event_name_like_any` `STRING ARRAY` (note that wildcards can be used to match specific patterns).

Note that is is a more common pattern to filter *out* unwanted values from the profile by using the `event_name_not_like_any` filter option.

#### event_name_not_like_any
The `event_name` profile exclude filter is implemented using the `NOT LIKE ANY` [Quantified Like Operator](https://cloud.google.com/bigquery/docs/reference/standard-sql/functions-and-operators#like_operator_quantified). It will only include values in the `event_name` profile and decoder which ***do not*** match one of the values in the `event_name_not_like_any` `STRING ARRAY` (note that wildcards can be used to match specific patterns).

!!! info "advanced event_options: `event_name_not_like_any`"
    === "JSON"
        ```json
        {
            "event_name_like_any": ["test%", "http:%"]
        }
        ```

    === "GoogleSQL"
        ```sql
        DECLARE event_options JSON; 

        SET event_options = JSON """
        {
            "event_name_like_any": ["test%", "http:%"]
        };
        """
        ```

### Event Params
#### event_param_like_any
The `event_param` profile include filter is implemented using the `LIKE ANY` [Quantified Like Operator](https://cloud.google.com/bigquery/docs/reference/standard-sql/functions-and-operators#like_operator_quantified). It will only include values in the `event_param` profile and decoder which match one of the values in the `event_param_like_any` `STRING ARRAY` (note that wildcards can be used to match specific patterns).

Note that is is a more common pattern to filter *out* unwanted values from the profile by using the `event_param_not_like_any` filter option.

#### event_param_not_like_any
The `event_param` profile exclude filter is implemented using the `NOT LIKE ANY` [Quantified Like Operator](https://cloud.google.com/bigquery/docs/reference/standard-sql/functions-and-operators#like_operator_quantified). It will only include values in the `event_param` profile and decoder which ***do not*** match one of the values in the `event_param_not_like_any` `STRING ARRAY` (note that wildcards can be used to match specific patterns).

!!! info "advanced event_options: `event_param_not_like_any`"
    === "JSON"
        ```json
        {
            "event_param_not_like_any": ["test%", "http:%", "%_test"]
        }
        ```

    === "GoogleSQL"
        ```sql
        DECLARE event_options JSON; 

        SET event_options = JSON """
        {
            "event_param_not_like_any": ["test%", "http:%", "%_test"]
        };
        """
        ```

### User Properties
#### user_property_like_any
The `user_property` profile include filter is implemented using the `LIKE ANY` [Quantified Like Operator](https://cloud.google.com/bigquery/docs/reference/standard-sql/functions-and-operators#like_operator_quantified). It will only include values in the `user_property` profile and decoder which match one of the values in the `user_property_like_any` `STRING ARRAY` (note that wildcards can be used to match specific patterns).

Note that is is a more common pattern to filter *out* unwanted values from the profile by using the `user_property_not_like_any` filter option.

#### user_property_not_like_any
The `user_property` profile exclude filter is implemented using the `NOT LIKE ANY` [Quantified Like Operator](https://cloud.google.com/bigquery/docs/reference/standard-sql/functions-and-operators#like_operator_quantified). It will only include values in the `user_property` profile and decoder which ***do not*** match one of the values in the `user_property_not_like_any` `STRING ARRAY` (note that wildcards can be used to match specific patterns).

!!! info "advanced event_options: `event_param_not_like_any`"
    === "JSON"
        ```json
        {
            "user_property_not_like_any": ["test%", "http:%", "%_test"]
        }
        ```

    === "GoogleSQL"
        ```sql
        DECLARE event_options JSON; 

        SET event_options = JSON """
        {
            "user_property_not_like_any": ["test%", "http:%", "%_test"]
        }    
        """
        ```