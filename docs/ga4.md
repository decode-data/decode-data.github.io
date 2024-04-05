# Google Analytics 4
## Installation
The GA4 decoder can be deployed in the same dataset as the inbound GA4 dataset (`ga4_dataset_id`), or a different dataset if desired.  Note that the installation function needs to be called in the same region in which the GA4 dataset is located (in the example below, for the `us` multi-region).

=== "Installation in GA4 dataset"

    ```SQL
    --DECLARATION
        DECLARE ga4_dataset_id, decoder_dataset_id STRING;

    --CONFIGURATION
        SET ga4_dataset_id = 'project_id.analytics_##########';
        SET decoder_dataset_id = ga4_dataset_id;

    --EXECUTION  
        CALL decodedata.us.install_ga4_decoder(ga4_dataset_id, decoder_dataset_id);
    ```

=== "Installation in different dataset"

    ```SQL
    --DECLARATION
        DECLARE ga4_dataset_id, decoder_dataset_id STRING;

    --CONFIGURATION
        SET ga4_dataset_id = 'project_id.analytics_##########';
        SET decoder_dataset_id = 'another_project_id.analytics_##########';

    --EXECUTION  
        CALL decodedata.us.install_ga4_decoder(ga4_dataset_id, decoder_dataset_id);
    ```

=== "Simplified syntax"

    ```SQL
    CALL decodedata.us.install_ga4_decoder('project_id.analytics_##########', 'project_id.analytics_##########');
    ```

## Output Resources
Following default installation, these output resources are built in the `decoder_dataset_id` dataset:

Resource Name | Resource Type | Partitioning Column | Row Granularity
--- | --- | --- | ---
**`events`** | `TABLE` | `event_date` | One row per event
**`sessions`** | `TABLE` | `event_date` | One row per session

These will be built from the latest data upon installation, but you will need to execute the `decoder_dataset_id.RUN_FLOW(start_date, end_date)` function to refresh output table with the transformed latest arriving data.

## Automation
In order to efficiently automate the decoder, a simple BigQuery scheduled query can be used to call the `RUN_FLOW` function periodically (typically every hour).  Since it is querying metadata to identify newly arriving date partitions, the underlying data is never queried, meaning that the compute required for the automation is predicatable and low (see the Compute Estimate section).

Since Google indicates that inbound data can change up to 72 hours after arrival, we typically refresh the past 4 days of data on each identified arrival of new data.

The scheduled query required to execute this is then simply:

=== "Overwrite past 4 days of data"

    ```SQL
    CALL `decoder_dataset_id.us.RUN_FLOW`(CURRENT_DATE - 4, NULL);
    ```

=== "Add query label"
    
    ```SQL
    SET @@query_label = "scheduled_query_id:decoder_dataset_id"; 
    
    CALL `decoder_dataset_id.us.RUN_FLOW`(CURRENT_DATE - 4, NULL)
    ```

Note that it is good practice to add a unique `scheduled_query_id` query label to the scheduled query.  This will enable job-based cost tracking across your GA4 properties to support robust cost management processes.

### Compute Estimate
The compute automation component to run the GA4 Decoder hourly is approximately $0.135 per month. This will not increase over time, regardless of inbound data volumes.

Component | Logic | Value | Unit
--- | --- | --- | ---
Execution Processing | 3 x 10MB Metadata Queries | 30 | MB
Monthly Executions | 30 days * 24hrs | 720 | Executions
Monthly Compute | 720 Executions * 30MB | 21.6 | GB
Compute Unit Cost | 1TB On-Demand Compute = $6.25 | 0.00625 | $/GB
 | | | 
**Total Automation<br>Monthly Compute** | **21.6GB * $0.00625** | **0.135** | **$**
