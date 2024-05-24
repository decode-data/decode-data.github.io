# Events
## Overview
The `events` table is a flattened transformation of the raw inbound `events_YYYYMMDD` table, with the following structural and semantic changes:

- **Simplified flat output structure** for optimal query syntax
- **Simple event count metrics** for comparison and calculations
- **Standard and custom event parameters** in property-specific flat schema
- **Custom user properties** in property-specific flat schema
- **Geo-based local timestamps** for time-of-day analyses
- **Observation-based data type coalescence** to prevent type-based historic data loss
- **Session and event ids** to support robust data quality assurance processes

One row represents one single discrete event.

## Schema
=== "Original `events_YYYYMMDD` schema"

    ```text
    events_YYYYMMDD
        ├── event_date STRING	
        ├── event_timestamp INTEGER	
        ├── event_name STRING	
        ├── event_params ARRAY<STRUCT>	
        ├── event_previous_timestamp INTEGER	
        ├── event_value_in_usd FLOAT	
        ├── event_bundle_sequence_id INTEGER	
        ├── event_server_timestamp_offset INTEGER	
        ├── user_id STRING	
        ├── user_pseudo_id STRING	
        ├── privacy_info STRUCT	
        ├── user_properties ARRAY<STRUCT>	
        ├── user_first_touch_timestamp INTEGER	
        ├── user_ltv STRUCT	
        ├── device STRUCT	
        ├── geo STRUCT	
        ├── app_info STRUCT	
        ├── traffic_source STRUCT	
        ├── stream_id STRING	
        ├── platform STRING	
        ├── event_dimensions STRUCT	
        ├── ecommerce STRUCT	
        ├── items ARRAY<STRUCT>	
        ├── collected_traffic_source STRUCT	
        └── is_active_user BOOLEAN	
    ```

=== "Output `events` schema"
    ```text
    events
        ├── project_id STRING	
        ├── dataset_name STRING	
        ├── analytics_property_id STRING	
        ├── event_id STRING	
        ├── session_id STRING	
        ├── event_date DATE	
        ├── event_timestamp TIMESTAMP	
        ├── event_name STRING	
        ├── event_previous_timestamp TIMESTAMP	
        ├── event_value_in_usd FLOAT	
        ├── event_bundle_sequence_id INTEGER	
        ├── event_server_timestamp_offset TIMESTAMP	
        ├── user_id STRING	
        ├── user_pseudo_id STRING	
        ├── privacy_info STRUCT	
        ├── user_first_touch_timestamp TIMESTAMP	
        ├── user_ltv STRUCT	
        ├── device STRUCT	
        ├── geo STRUCT	
        ├── app_info STRUCT	
        ├── traffic_source STRUCT	
        ├── stream_id STRING	
        ├── platform STRING	
        ├── event_dimensions STRUCT	
        ├── collected_traffic_source STRUCT	
        ├── is_active_user BOOLEAN
        ├── local STRUCT	
        ├── count STRUCT
        ├── parameter STRUCT
        └── property STRUCT	    
    ```

### New Columns (Metadata)
The following metadata columns are added to the output `events` table.

| COLUMN NAME | DATA TYPE | DESCRIPTION
| --- | --- | ---
| project_id | STRING | `project_id` containing the `ga4_dataset_id`
| dataset_name | STRING | `dataset_name` of the GA4 dataset
| analytics_property_id | STRING | `analytics_property_id` of the GA4 dataset
| event_id | STRING | Unique ID per event
| session_id | STRING | Unique ID per session

### Existing Columns
The following columns are passed through directly from the raw `events_YYYYMMDD`, with no changes to structure or data type:

| COLUMN NAME| DATA TYPE |
| --- | --- |
| event_name | STRING |
| event_value_in_usd | FLOAT |
| event_bundle_sequence_id | INTEGER |
| event_server_timestamp_offset | INTEGER |
| user_id | STRING |
| user_pseudo_id | STRING |
| privacy_info | STRUCT |
| user_ltv | STRUCT |
| device | STRUCT |
| geo | STRUCT |
| app_info | STRUCT |
| traffic_source | STRUCT |
| stream_id | STRING |
| platform | STRING |
| event_dimensions | STRUCT |
| collected_traffic_source | STRUCT |
| is_active_user | BOOLEAN |
| privacy_info | STRUCT |
| user_ltv | STRUCT |
| device | STRUCT |
| geo | STRUCT |
| app_info | STRUCT |
| traffic_source | STRUCT |
| event_dimensions | STRUCT |
| collected_traffic_source | STRUCT |

### Data Type Changes
The following columns are included in the `events` output table, with appropriate data type changes.

| COLUMN NAME | FROM DATA TYPE | TO DATA TYPE | DESCRIPTION
| --- | --- | --- | --- |
| event_date | STRING | DATE | Inbound shard date 
| event_timestamp | INTEGER | TIMESTAMP | UTC timestamp of event occurrence
| event_previous_timestamp | INTEGER | TIMESTAMP | UTC timestamp of previous event occurrence
| user_first_touch_timestamp | INTEGER | TIMESTAMP | UTC timestamp of user first touch

### New Columns (Augmented)
The following augmentation columns are added to the output `events` table:

```txt
local STRUCT	
    ├── timezone_id STRING	
    ├── timezone_name STRING	
    ├── country_code STRING	
    ├── timezone_source STRING	
    ├── latitude STRING	
    ├── longitude STRING	
    ├── date DATE	
    ├── timestamp DATETIME	
    ├── time TIME	
    ├── hour INTEGER	
    ├── hour_decimal FLOAT	
    ├── previous_timestamp DATETIME	
    └── first_touch_timestamp DATETIME	
```

| COLUMN NAME | DATA TYPE | DESCRIPTION |
| --- | --- | --- |
| local | STRUCT | NEW local timezone and geolocation STRUCT |
| local.timezone_id | STRING | NEW timezone id |
| local.timezone_name | STRING | NEW timezone name |
| local.country_code | STRING | NEW ISO country code |
| local.timezone_source | STRING | NEW source of timezone allocation (city/region/country) |
| local.latitude | STRING | NEW latitude |
| local.longitude | STRING | NEW longitude |
| local.date | DATE | NEW geo-adjusted `event_date` |
| local.timestamp | DATETIME | NEW geo-adjusted `event_timestamp` |
| local.time | TIME | NEW geo-adjusted time |
| local.hour | INTEGER | NEW geo-adjusted hour |
| local.hour_decimal | FLOAT | NEW geo-adjusted decimal hour |
| local.previous_timestamp | DATETIME | NEW geo-adjusted `previous_timestamp` |
| local.first_touch_timestamp | DATETIME | NEW geo-adjusted `first_touch_timestamp` |

### New Columns (Restructured)
The following flat `STRUCT` columns are added, replacing the nested source columns which are excluded from the output `events` table. The `event_name` column is still included in the output `events` table, but is supplemented with the `count` metrics `STRUCT`.

| SOURCE COLUMN NAME | SOURCE DATA TYPE | TRANSFORMATION
| --- | --- | --- 
| event_name | STRING | Transformed to `count` metric STRUCT
| event_params | ARRAY | Transformed to `parameter` STRUCT
| user_properties | ARRAY | Transformed to `property` STRUCT

#### Count Schema
The following metric sub-columns are included as standard in the `count` `STRUCT`, in addition to any other `event_name` values detected in the data.

```text
count STRUCT	
    ├── events INTEGER	
    ├── conversions INTEGER	
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

#### Parameter Schema
The following value sub-columns are included as standard in the `parameter` `STRUCT`, in addition to any other `event_parameter` values detected in the data.
```text
    parameter STRUCT	
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

#### Property Schema
There are no standard values for `user_properties`, so sub-columns will only be included in the property `STRUCT` if they are detected in the data.
