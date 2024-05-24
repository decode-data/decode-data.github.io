# Google Analytics 4
## Objective
The Google Analytics 4 (GA4) decoder simplifies and augments the GA4 BigQuery export, making subsequent data operations simpler and quicker.

It enables automatic pre-modelling of the GA4 BigQuery `events_YYYYMMDD` export, providing flattened, date-partitioned `events` and `sessions` tables containing all standard and observed `event_params` and `user_properties`.  It also provides mechanisms to detect new `event_params` and `user_properties` values and to incrementally update the schema based on inbound data detection.

It can be installed using a one-line command in the BigQuery console by permitted users on registered datasets, and has no dependencies on any external platforms or API calls.

## Access
The Google Analytics 4 decoder is currently open to private alpha registration. Apply for access <a href="https://docs.google.com/forms/d/e/1FAIpQLSf1LVjV2PAVxOqnQMZrg43XMRwblpHPaooGGX2eCJ1Or52qwg/viewform?usp=sf_link" target="_blank">here</a>.

## Installation
The GA4 decoder can be deployed in the same dataset as the inbound GA4 dataset (`ga4_dataset_id`), or a different dataset if desired.  Note that the installation function needs to be called in the same region in which the GA4 dataset is located (in the examples below, for the `us` multi-region).

### Basic Installation
In the following installation examples, the `event_options` `JSON` variable is `NULL`, which means that default installation options are used. The `event_options` is the mechanism via which custom installation configurations are set.

=== "Basic installation in the GA4 dataset"

    ```SQL
      DECLARE ga4_dataset_id, decoder_dataset_id STRING;
      DECLARE event_options JSON DEFAULT JSON '{}';

      SET ga4_dataset_id = 'project_id.analytics_##########';
      SET decoder_dataset_id = ga4_dataset_id;

      CALL decodedata.us.install_ga4_decoder(ga4_dataset_id, decoder_dataset_id, event_options);
    ```

=== "Basic installation in a different dataset"

    ```SQL
      DECLARE ga4_dataset_id, decoder_dataset_id STRING;
      DECLARE event_options JSON DEFAULT JSON '{}';

      SET ga4_dataset_id = 'project_id.analytics_##########';
      SET decoder_dataset_id = 'project_id.decoder_dataset_name';

      CALL decodedata.us.install_ga4_decoder(ga4_dataset_id, decoder_dataset_id, event_options);
    ```

=== "Concise syntax"

    ```SQL
    CALL decodedata.us.install_ga4_decoder('project_id.analytics_##########', 'project_id.decoder_dataset_name' NULL);
    ```

## Output Resources
### Output Tables
Following installation, these output tables are built in the `decoder_dataset_id` dataset:

Resource Name | Resource Type | Partitioning Column | Row Granularity
--- | --- | --- | ---
**`events`** | `TABLE` | `event_date` | One row per event
**`sessions`** | `TABLE` | `event_date` | One row per session

Detailed information regarding the transformation, augmentation and schema is available in the [events](/events/index.md) and [sessions](/sessions.md) docs.

These date-partitioned tables will be built from the latest data upon installation.

### Functions
Following installation, these execution functions are also deployed in the `decoder_dataset_id` dataset:

Resource Name <div style="width:120px"></div>| Resource Type | Arguments <div style="width:180px"></div>| Action
--- | --- | --- | ---
**`INSTALL_DECODER`** | `PROCEDURE` | none | Reinstall decoder with original deployment options
**`RUN_DECODER`** | `PROCEDURE` | `execution_options JSON` | Execute the decoder and refresh output tables

The `decoder_dataset_id.INSTALL_DECODER()` function is executed to reinstall the decoder with the original installation configuration.  It also rebuilds output tables with the full historic data.

The `decoder_dataset_id.RUN_DECODER(execution_options)` function is executed to refresh output tables with the transformed latest arriving data.

## Automation
The `RUN_DECODER` function has two different execution modes: `incremental` and `full`. Execution mode is set using the `execution_options` `JSON` arguments.

In order to efficiently automate the decoder, a simple BigQuery scheduled query can be used to call the `RUN_DECODER` function periodically (typically every hour).  Since it is querying metadata to identify newly arriving date partitions, the underlying data is never queried, meaning that the compute required for the automation is predictable and low (see the Compute Estimate section).

### Incremental Refresh
The `incremental` execution mode is the optimal mode to keep the output tables up-to-date in a cost-efficient manner. It compares inbound and outbound table metadata to determine when new date partitions arrive, and runs on those partitions only (plus an additional `n` days, due to late arriving data in the GA4 export).  Note that since the source data can change up to 72 hours after the table is created, it is recommended to set this to 4 days.

=== "Incremental refresh"

    ```SQL
    DECLARE execution_options JSON DEFAULT JSON '{"mode": "incremental", "date_partitions_to_replace": 4}';

    CALL `decoder_dataset_id.us.RUN_DECODER`(execution_options);
    ```

=== "Concise syntax"
    ```SQL
    CALL `decoder_dataset_id.us.RUN_DECODER`(JSON '{"mode": "incremental", "date_partitions_to_replace": 4}');
    ```

### Full Refresh
To execute a full refresh of output tables between the first observed source shard data and the `CURRENT_DATE`, use the following `execution_options`:

=== "Full refresh"

    ```SQL
    DECLARE execution_options JSON DEFAULT JSON '{"mode": "full"}';

    CALL `decoder_dataset_id.us.RUN_DECODER`(execution_options);
    ```

=== "Concise syntax"
    ```SQL
    CALL `decoder_dataset_id.us.RUN_DECODER`(JSON '{"mode": "full"}');
    ```

### Default mode
If the `execution_options` `JSON` argument is `NULL`, the execution defaults to `mode = incremental` and `date_partitions_to_replace = 4`.

=== "Default incremental cxecution"

    ```SQL
    DECLARE execution_options JSON DEFAULT NULL;

    CALL `decoder_dataset_id.us.RUN_DECODER`(execution_options);
    ```

=== "Concise syntax"

    ```SQL
    CALL `decoder_dataset_id.us.RUN_DECODER`(NULL);
    ```


## Compute Estimate
The compute automation component to run the GA4 decoder hourly is approximately $0.135 per property per month. This consumption will not increase over time, regardless of inbound data volumes.

Component | Logic | Value | Unit
--- | --- | --- | ---
Execution Processing | 3 x 10MB Metadata Queries | 30 | MB
Monthly Executions | 30 days * 24hrs | 720 | Executions
Monthly Compute | 720 Executions * 30MB | 21.6 | GB
Compute Unit Cost | 1TB On-Demand Compute = $6.25 | 0.00625 | $/GB
 | | | 
**Total Automation<br>Monthly Compute** | **21.6GB * $0.00625** | **0.135** | **$**