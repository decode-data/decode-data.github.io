# Event Parameters
## Overview
Event parameters are found in the `event_params` `STRUCT`, an unusually structured element which is the source of most of the complexity when working with the GA4 export in BigQuery.  It is a nested variation on a key-value pair, where the value can be found in one (and only one) of the value sub-columns.  The `STRUCT` schema can be represented as follows:

```text
event_params STRUCT
├── key STRING
└── value
    ├── string_value STRING
    ├── int_value INT64
    ├── float_value FLOAT64
    └── double_value FLOAT64
```

This structure means that new parameters can be added without changing the table schema, but it also means that data types can change over time, with no simple built-in mechanism for detection.

## Standard Event Parameters
The following event parameters are included as standard in the GA4 decoder output tables, whether they are observed in the source data or not.

| COLUMN NAME | DATA TYPE |
| --- | --- |
| anonymize_ip | STRING |
| batch_ordering_id | INT64 |
| batch_page_id | INT64 |
| campaign | STRING |
| debug_mode | INT64 |
| engaged_session_event | INT64 |
| engagement_time_msec | INT64 |
| entrances | INT64 |
| file_extension | STRING |
| file_name | STRING |
| form_destination | STRING |
| form_id | STRING |
| form_name | STRING |
| ga_session_id | INT64 |
| ga_session_number | INT64 |
| ignore_referrer | STRING |
| link_classes | STRING |
| link_domain | STRING |
| link_id | STRING |
| link_text | STRING |
| link_url | STRING |
| medium | STRING |
| outbound | STRING |
| page_location | STRING |
| page_path | STRING |
| page_referrer | STRING |
| page_title | STRING |
| percent_scrolled | INT64 |
| search_term | STRING |
| session_engaged | STRING |
| source | STRING |
| term | STRING |
| video_current_time | INT64 |
| video_duration | INT64 |
| video_percent | STRING |
| video_provider | STRING |
| video_title | STRING |
| video_url | STRING |
| visible | STRING |

The default name for the event parameter output `STRUCT` is `parameter`, but this is configurable upon installation.  

These event parameter values can then be accessed concisely by using dot notation, for example: `parameter.video_title`.

### Inconsistent Data Types
#### `session_engaged`
The `session_engaged` column values have been observed to be either `STRING` or `INT64` values, which is automatically corrected for in the GA4 decoder. This can be seen in the `GA4_event_params` decoder function, for which the generated code is below.

```sql
(SELECT COALESCE(SAFE_CAST(value.int_value AS STRING), value.string_value) FROM UNNEST(event_params) WHERE key='session_engaged') AS `session_engaged`
```

Note that many articles and guides wrongly extract the `session_engaged` value as either an `INT64` or `STRING` column, which we have observed to under-report engaged sessions by 3-12%.

### Automatic Type Coalescence
When multiple data types are observed for an event parameter in the source data, if only one of the value sub-columns is used then data will be lost. 

The GA4 decoder controls for this by combining `COALESCE` and `SAFE_CAST` as above, with the output data type dependent on which combination of non-null values are observed. They will be coalesced to a common [supertype](https://cloud.google.com/bigquery/docs/reference/standard-sql/conversion_rules#supertypes). For example, if `STRING` and `INT64` values are observed, they resulting field will be a `STRING`, and if `INT64` AND `FLOAT64` values are observed, the resulting field will be a `FLOAT64`.

## Custom Event Parameters
When additional parameters are detected in the source data, they will also be included in the `parameter` output `STRUCT`. These will be found _after_ the standard alphabetically listed event parameters, so they can be easily identified.  Recommended event parameters are included _if_ they are in the expected or observed custom event parameters.

## Output Schema
The schema of the output `parameter` `STRUCT` is therefore:

```text
event_params STRUCT
├── anonymize_ip STRING
├── batch_ordering_id INT64
├── batch_page_id INT64
├── campaign STRING
├── debug_mode INT64
├── engaged_session_event INT64
├── engagement_time_msec INT64
├── entrances INT64
├── file_extension STRING
├── file_name STRING
├── form_destination STRING
├── form_id STRING
├── form_name STRING
├── ga_session_id INT64
├── ga_session_number INT64
├── ignore_referrer STRING
├── link_classes STRING
├── link_domain STRING
├── link_id STRING
├── link_text STRING
├── link_url STRING
├── medium STRING
├── outbound STRING
├── page_location STRING
├── page_path STRING
├── page_referrer STRING
├── page_title STRING
├── percent_scrolled INT64
├── search_term STRING
├── session_engaged STRING
├── source STRING
├── term STRING
├── video_current_time INT64
├── video_duration INT64
├── video_percent STRING
├── video_provider STRING
├── video_title STRING
├── video_url STRING
├── visible STRING
└── [custom event parameters]
```