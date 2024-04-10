# Events
## Overview
The `sessions` table is a flattened date-partitioned session-level aggregation of the `events` table. The attibution model defaults to `first_click`, but can also be configured as `last_click`, `first_click_non_direct` or `last_click_non_direct`.

One row represents one single discrete session.

## Schema
### Aggregation Columns
The following columns are used as aggregation dimensions, meaning that one row will correspond to a unique combination of these column values.

| COLUMN NAME | DATA TYPE | DESCRIPTION
| --- | --- | ---
| session_id | STRING | Unique ID per session
| project_id | STRING | `project_id` containing the `ga4_dataset_id`
| dataset_name | STRING | `dataset_name` of the GA4 dataset
| analytics_property_id | STRING | `analytics_property_id` of the GA4 dataset
| event_id | STRING | Unique ID per event
| stream_id | STRING | ID of GA4 stream
| platform | STRING | Web or App
| user_pseudo_id | STRING | Unique ID corresponding to identified user

### Aggregated Metadata Columns
The following columns are aggregated in order to make sense at a session-level aggregation.

| COLUMN NAME | DATA TYPE | DEFINITION | DESCRIPTION
| --- | --- | --- | ---
| event_date | DATE | `MIN(event_date)` | The `event_date` of the first occurring event in a session
| session_start_timestamp | DATE | `MIN(event_timestamp)` | The `event_timestamp` of the first occurring event in a session
| session_end_timestamp | TIMESTAMP | `MAX(event_timestamp)` | The `event_timestamp` of the last occurring event in a session
| event_names | ARRAY<STRING> | `ARRAY_AGG(DISTINCT event_name IGNORE NULLS)` | An array of unique `event_name` values in a session

### Channel Grouping Column
The `channel_grouping` `STRING` column is added, in alignment with the Google [Default channel group](https://support.google.com/analytics/answer/9756891?hl=en&ref=ga4bigquery.com) logic.  This enables the `last_click_non_direct` and `last_click_non_direct` attribution models.

### Aggregated Metric Columns
All columns in the `count` metric `STRUCT` in the `events` table are summed across all events in a session, with the naming convention unchanged (e.g. `SUM(count.page_view) AS page_view`).  Additionally, the following fields are summed across all event metrics:

| COLUMN NAME | DATA TYPE | DEFINITION | DESCRIPTION
| --- | --- | --- | ---
| event_value_in_usd | FLOAT | `SUM(event_value_in_usd)` | Sum of `event_value_in_usd` across all events in a session
| count.total_sessions | INTEGER | `1 AS total_sessions` | An integer flag to enable `total_sessions` to be used as an output metric
| count.total_events | INTEGER | `SUM(count.total_events)` | The count of events in a session
| count.total_conversion_events | INTEGER | `SUM(count.total_conversions)` | Total conversion events in a session
| count.total_conversion_sessions | INTEGER | `MAX(count.total_conversions)` | Flag (`1`) if a session contains at least one conversion event
| parameter.total_engagement_time_msec | FLOAT | `SUM(parameter.engagement_time_msec)` | The sum of engagement time across all session events

### Attributed Columns
The following column values are selected based on the attribution model configured upon deployment(`first_click`, `last_click`, `first_click_non_direct` and `last_click_non_direct`), with direct-attributed events defined by the `channel_grouping`.  The attribution model for the deployment is then recorded in the `attribution_model` `STRING` column.

| COLUMN NAME | DATA TYPE 
| --- | --- 
| privacy_info | STRUCT
| user_first_touch_timestamp | TIMESTAMP
| user_ltv | STRUCT
| device | STRUCT
| geo | STRUCT
| app_info | STRUCT
| traffic_source | STRUCT
| event_dimensions | STRUCT
| collected_traffic_source | STRUCT
| channel_grouping | STRING
| local | STRUCT

### Aggregated Array Columns
Additionally, for `parameter` and `property` values observed at the `event` level, `ARRAY` fields of unique, non-null values are included for downstream anaytical purposes.  The naming convention is unchanged from the `event` table. 

If specific `parameter` or `property` sub-columns require different aggregation treatment (e.g. `SUM`), this is configurable upon deployment. By default the only parameter sub-column which is summed is `parameter.total_engagement_time_msec`, which is defined as `SUM(parameter.engagement_time_msec)` across all session events.