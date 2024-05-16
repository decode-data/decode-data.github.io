# Event Names
## Overview
Events all have a non-null `event_name`, which is retained in the decoder output `event_name` column. These are also included as `INT64` sub-columns of the `count` `STRUCT`, enabling them to be used as metrics (and therefore based inputs for additional computations) in the decoder output tables.

## Standard Event Names
The following event names are included as standard in the GA4 decoder output tables, whether they are observed in the source data or not. 

| EVENT NAME |
| --- |
| click |
| file_download |
| first_visit |
| form_start |
| form_submit |
| page_view |
| purchase |
| scroll |
| search |
| session_start |
| user_engagement |
| video_complete |
| video_progress |
| video_start |
| view_search_results |

## Custom Event Names
When additional event names are detected in the source data, they will be preserved in the `event_name` column and will also be included in the `count` output `STRUCT`. These will be found _after_ the alphabetically listed standard event names, so they can be easily identified.  Recommended event parameters are included _if_ they are in the expected or observed custom event parameters.

### New Metric Sub-Columns 
The following sub-columns are added to the `count` metric STRUCT to enable simple subsequent computations:

| COLUMN NAME | DATA TYPE | DESCRIPTION
| --- | --- | --- 
| count.total_events | INTEGER | An integer flag to enable `total_events` to be used as an output metric
| count.total_conversions | INTEGER | An integer flag to enable `total_conversions` to be used as an output metric.

Note that `event_name` values to be classified as conversions are set in the installation configuration, but defaults to `purchase` as per the Google default.  These can be renamed in subsequent transformations or BI tools to retain consistency with whatever confusing Google name changes happen in future. 

## STRUCT Schema: `count`
The schema of the output `count` `STRUCT` is therefore:

```text
count STRUCT
├── total_events INT64
├── total_conversions INT64
├── click INT64
├── file_download INT64
├── first_visit INT64
├── form_start INT64
├── form_submit INT64
├── page_view INT64
├── purchase INT64
├── scroll INT64
├── search INT64
├── session_start INT64
├── user_engagement INT64
├── video_complete INT64
├── video_progress INT64
├── video_start INT64
├── view_search_results INT64
└── [custom event NAMES]
```
