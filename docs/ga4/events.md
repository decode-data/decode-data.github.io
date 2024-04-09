# Events
## Overview
The `events` table is a flattened transformation of the raw inbound `events_YYYYMMDD` table, with the following structural and semantic changes:

- **Simplified flat output structure** for optimal query syntax
- **Simple event count metrics** for simple comparison and calculations
- **Standard and custom event parameters** in property-specific flat schema
- **Custom user properties** in property-specific flat schema
- **Geo-based local timestamps** for time-of-day analyses
- **Intelligent data type coalescence** to prevent type-based historic data loss
- **Session and event ids** to support robust data quality assurance processes

One row represents one single discrete event.

## Schema
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
| privacy_info | RECORD |
| user_ltv | RECORD |
| device | RECORD |
| geo | RECORD |
| app_info | RECORD |
| traffic_source | RECORD |
| stream_id | STRING |
| platform | STRING |
| event_dimensions | RECORD |
| collected_traffic_source | RECORD |
| is_active_user | BOOLEAN |
| privacy_info | STRING |
| user_ltv | FLOAT |
| device | STRING |
| geo | STRING |
| app_info | STRING |
| traffic_source | STRING |
| event_dimensions | STRING |
| collected_traffic_source | STRING |

### Data Type Changes
The following columns are included in the `events` output table, with appropriate data type changes.

| COLUMN NAME | FROM DATA TYPE | TO DATA TYPE |
| --- | --- | --- |
| event_date | STRING | DATE |
| event_timestamp | INTEGER | TIMESTAMP |
| event_previous_timestamp | INTEGER | TIMESTAMP |
| user_first_touch_timestamp | INTEGER | TIMESTAMP |

### New Columns (Augmentation)
The following augmentation columns are added to the output `events` table:

| COLUMN NAME | DATA TYPE | DESCRIPTION |
| --- | --- | --- |
| local | RECORD | NEW local timezone and geolocation STRUCT |
| local.timezone_id | STRING | NEW local timezone STRING column |
| local.timezone_name | STRING | NEW local timezone STRING column |
| local.country_code | STRING | NEW local timezone STRING column |
| local.timezone_source | STRING | NEW local timezone STRING column |
| local.latitude | STRING | NEW geolocation STRING column |
| local.longitude | STRING | NEW geolocation STRING column |
| local.date | DATE | NEW geo-adjusted DATE column |
| local.timestamp | DATETIME | NEW geo-adjusted DATETIME column |
| local.time | TIME | NEW geo-adjusted TIME column |
| local.hour | INTEGER | NEW geo-adjusted INTEGER column |
| local.hour_decimal | FLOAT | NEW geo-adjusted FLOAT column |
| local.previous_timestamp | DATETIME | NEW geo-adjusted DATETIME column |
| local.first_touch_timestamp | DATETIME | NEW geo-adjusted DATETIME column |

### New Columns (Restructuring)
The following flat `STRUCT` columns are added, replacing the nested source columns which are excluded from the output `events` table. The `event_name` column is still included in the output `events` table, but is supplemented with the `count` metrics `STRUCT`.

| SOURCE COLUMN NAME | SOURCE DATA TYPE | TRANSFORMATION
| --- | --- | --- 
| event_name | STRING | Transformed to `count` metric STRUCT
| event_params | ARRAY | Transformed to `parameter` STRUCT
| user_properties | ARRAY | Transformed to `property` STRUCT


