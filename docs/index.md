---
hide:
  - navigation
---

# Overview
## What do we do?
We build Google BigQuery data automation capabilities, using native BigQuery functionality and adjacent Google Cloud products. In order to transform and augment inbound data to support subsequent analytics, automation and AI use-cases, we build and deploy source-specific data decoders for our clients and partners, and license them to 3rd parties.

## What is a decoder?
A decoder restructures and augments inbound data to optimise the structure and contents for query and analysis. It is essentially an _upgrade_ to a standard inbound data source, removing complexity from downstream operations in order to speed up (and level-up) subsequent data activities.

It is installed between standard Google-managed inbound BigQuery data sources and subsequent transformation, augmentation, analysis or automation activities.

=== "Decoder Logical Flow"

    ```mermaid
    flowchart LR
     source_data(["Source Data"]) --> inbound_data["Inbound<br>Data Transfer"] --> decoder{{"Decoder"}} --> outbound_data["Outbound<br>Data Assets"] --> downstream_activities_gc(["Downstream<br>Data Activities"])

     style decoder fill:#ffffff,stroke:#333,stroke-width:2px
    ```

It efficiently and reponsively builds downstream data assets and is optionally configurable upon deployment, with pre-configured installation functions for connection to Looker, Looker Studio, Tableau, Power BI, other business intelligence tools or even cross-cloud storage buckets.

Downstream tables can also be connected directly to data transformation-specific tools like [Dataform](https://cloud.google.com/dataform/docs/) or [DBT](https://docs.getdbt.com/), to simplify subsequent data modelling activities.

By using a metadata-driven approach, it is responsive to newly arriving data in a cost-optimal manner and requires minimal ongoing maintenance or monitoring (although metadata-based monitoring functions are provided).

## Why would you need one?
Inbound data structures are typically designed for optimal storage and schema stability, not for simplicity of modelling, query or analytics. Specifically, nested data structures pose challenges as the structure of the nested data needs to be known ahead of time in order to extract meaningful data. This also applies to JSON data.

Historically this has meant that transforming this data required painfully hand-crafting queries with a verbose, unintuitive and uncommon query syntax, slowing down subsequent data-related activities.

Many Google-managed inbound data transfers also do not arrive at a consistent time every day, which can be challenging to control for efficiently and reliably.

## How does it work?
Decoders are installed by executing a BigQuery function with installation-specific configuration arguments. Installation involves the following automated stages:

1. Profile data structure, contents and types
2. Build source-specific decoder functions based on configuration and data profile
3. Build output assets (typically date-partitioned tables)
4. Deploy automation, monitoring and installation functions

=== "Decoder Installation"

    ```mermaid
    flowchart LR
     profile_data["Profile<br>Data"] --> build_decoder_functions["Build<br>Decoder<br>Functions"] --> build_output_assets["Build<br>Output<br>Assets"] --> deploy_automation_function["Deploy<br>Management<br>Functions"] 
    ```

Once a decoder is installed, the output data assets can then be:

- queried directly;
- connected directly to business intelligence tools;
- used as an input to simplified transformation models; and 
- extended with SQL to customise subsequent data transformation.

Decoders are built using native (but extended) BigQuery functionality, so do not require external platforms or API calls. Decoders can be deployed on whitelisted datasets, by users with the appropriate permissions to call the specific installation functions.

A simple automation function is deployed in the decoder dataset, which checks and compares inbound shard and outbound partition metadata. New date partitions are identified, transformation/augmentation logic is applied to the newly-arrived data and the output data tables are updated incrementally.

This is triggered on a regular schedule (as short as every 5 minutes, but typically hourly), and can also be called manually to refesh specific partition subsets of the outbound tables. Since this is built on metadata queries (and does not query the underlying data) it is a simple mechanism to execute data transformations upon arrival of both predictably and unpredictably timed inbound data.

Access to installation and automation functions is granted to permitted users on whitelisted datasets.

## Where is it installed?
A decoder is installed in a configurable dataset (within the same region as the inbound data), but typically the same dataset as the inbound data. 

Decoders can deployed programmatically across hundreds of data sources in just a few lines of code. No configuration is _required_, but decoders _can_ be configured in granular ways depending on the use-case and data.

## What Decoders are available?
The [Google Analytics 4 Decoder](ga4/index.md) is now generally available.  Signup to one of our plans [here](signup.md), for a single GA4 property, enterprise or agency plans. All plans feature a free trial period for evaluation.

Additional decoders are also in development and are currently being used on a variety of internal and client projects.
