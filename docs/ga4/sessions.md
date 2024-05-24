# Events
## Overview
The `sessions` table is a flattened date-partitioned session-level aggregation of the `events` table. The attibution model defaults to `last_click_non_direct`, but can also be configured as `last_click`, `first_click` or  `first_click_non_direct`.

One row represents one single discrete session.

## Schema
=== "Output `sessions` schema"
    ```text
        events
            ├── session_id STRING	
            ├── project_id STRING	
            ├── dataset_name STRING	
            ├── analytics_property_id STRING	
            ├── stream_id STRING	
            ├── platform STRING	
            ├── user_pseudo_id STRING	
            ├── is_active_user BOOLEAN	
            ├── is_engaged_session BOOLEAN	
            ├── event_date DATE	
            ├── session_start_timestamp TIMESTAMP	
            ├── session_end_timestamp TIMESTAMP	
            ├── event_names ARRAY<STRING>	
            ├── privacy_info STRUCT	
            ├── user_first_touch_timestamp TIMESTAMP	
            ├── user_ltv STRUCT	
            ├── device STRUCT	
            ├── geo STRUCT	
            ├── app_info STRUCT	
            ├── attribution_model STRING	
            ├── traffic_source STRUCT	
            ├── event_dimensions STRUCT	
            ├── collected_traffic_source STRUCT	
            ├── channel_grouping STRING	
            ├── local STRUCT	
            ├── event_value_in_usd FLOAT	
            ├── count STRUCT	
            └── parameter STRUCT	
    ```

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
| count.total_engaged_sessions | INTEGER | ```MAX(SAFE_CAST(parameter.session_engaged AS INT64))``` | An integer flag to indicate whether the session was classified as engaged
| count.total_events | INTEGER | `SUM(count.total_events)` | The count of events in a session
| count.total_conversion_events | INTEGER | `SUM(count.total_conversions)` | Total conversion events in a session
| count.total_conversion_sessions | INTEGER | `MAX(count.total_conversions)` | Flag (`1`) if a session contains at least one conversion event

The standard schema for the `count` `STRUCT` in the sessions table is therefore:

```text
    count STRUCT	
        ├── sessions INTEGER	
        ├── engaged_sessions INTEGER	
        ├── events INTEGER	
        ├── conversion_events INTEGER	
        ├── conversion_sessions INTEGER	
        ├── click INTEGER	
        ├── file_download INTEGER	
        ├── first_visit INTEGER	
        ├── form_start INTEGER	
        ├── form_submit INTEGER	
        ├── page_view INTEGER	
        ├── purchase INTEGER	
        ├── scroll INTEGER	
        ├── search INTEGER	
        ├── session_start INTEGER	
        ├── user_engagement INTEGER	
        ├── video_complete INTEGER	
        ├── video_progress INTEGER	
        ├── video_start INTEGER	
        └── view_search_results INTEGER	
```


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

### Parameter/Property Columns
Additionally, for `parameter` and `property` values observed at the `event` level, the `MAX` of the column value is taken, ensuring that if any value is `NOT NULL`, the `MAX` value will be propagated to the `sessions.parameter` or `sessions.property` `STRUCT`.  The naming convention is unchanged from the `event` table. 

If specific `parameter` or `property` sub-columns require different aggregation treatment (e.g. `SUM`), this is configurable upon deployment. By default the only parameter sub-column which is summed is `parameter.total_engagement_time_msec`, which is defined as `SUM(parameter.engagement_time_msec)` across all session events.

The standard schema for the `parameter` `STRUCT` in the sessions table is therefore:

```text
parameter STRUCT	
    ├── total_engagement_time_msec INTEGER	
    ├── anonymize_ip STRING	
    ├── batch_ordering_id INTEGER	
    ├── batch_page_id INTEGER	
    ├── campaign STRING	
    ├── debug_mode INTEGER	
    ├── engaged_session_event INTEGER	
    ├── engagement_time_msec INTEGER	
    ├── entrances INTEGER	
    ├── file_extension STRING	
    ├── file_name STRING	
    ├── form_destination STRING	
    ├── form_id STRING	
    ├── form_name STRING	
    ├── ga_session_id INTEGER	
    ├── ga_session_number INTEGER	
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
    ├── percent_scrolled INTEGER	
    ├── search_term STRING	
    ├── session_engaged STRING	
    ├── source STRING	
    ├── term STRING	
    ├── video_current_time INTEGER	
    ├── video_duration INTEGER	
    ├── video_percent STRING	
    ├── video_provider STRING	
    ├── video_title STRING	
    ├── video_url STRING	
    └── visible STRING	
```