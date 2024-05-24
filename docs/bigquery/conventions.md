## Naming Conventions
Term <div style="width:110px"></div> | Data Type | Description 
--- | --- | --- 
`project_id` | `STRING` | The globally unique identifier for each [project](https://cloud.google.com/resource-manager/docs/creating-managing-projects)
`dataset_name` | `STRING` | The name for each [dataset](https://cloud.google.com/bigquery/docs/datasets), unique in each project
`dataset_id` | `STRING` | The globally unique identifier for each dataset, composed as `project_id.dataset_name`
`resource_name`  | `STRING` | The single name for each resource (`TABLE`, `VIEW`, `FUNCTION`, `TABLE FUNCTION` or `PROCEDURE`)
`resource_id` | `STRING` | The globally unique identifier for an individual `TABLE`, `VIEW`, `FUNCTION`, `TABLE FUNCTION` or `PROCEDURE`, composed as `project_id.dataset_name.resource_name`
`date_id` | `STRING` | A `STRING` representantion of a date in the format `YYYYMMDD`
`shard_id` | `STRING` |  A `STRING` representantion of a table shard date in the format `YYYYMMDD`
`shard_date` | `DATE` |  A `DATE` representantion of a table shard date
`partition_id` | `STRING` |  A `STRING` representantion of a table partition date in the format `YYYYMMDD`
`partition_date` | `DATE` |  A `DATE` representantion of a table partition date