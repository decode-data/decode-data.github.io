## Installation
To install and run the decoder in the GA4 dataset, with default options, execute the following code in your BigQuery console, replacing `project_id.analytics_##########` with your GA4 `dataset_id`. 

Note that decoder functions will only add new functions, tables and views to your GA4 dataset, and never delete anything.

=== "Default installation"
    ```SQL
      CALL decodedata.us.install_ga4_decoder('project_id.analytics_##########', 'project_id.analytics_##########', NULL);
    ```

## Execution
Upon installation, to execute an incremental update on your data execute the following code in your BigQuery console, replacing `project_id.analytics_##########` with your GA4 `dataset_id`.  This will only write new date partitions to your outbound tables when new source data is detected:

=== "Default execution"
    ```SQL
      CALL `project_id.analytics_##########`.RUN_DECODER(NULL)
    ```

This is a metadata query, so you will not be charged for querying the underlying data. 