# Decoders
Decoders restructure and augment inbound data to optimise for query and analysis.  They are installed between Google-managed inbound BigQuery data transfers and subsequent transformation, analysis or automation activities. 

## What?
Decoders:

- Restructure inbound data to optimise for query and analysis
- Optionally augment data from external sources
- Efficiently and reponsively curate downstream data tables

They are configurable upon deployment, with pre-configured installation functions for specific business intelligence tools (e.g. Looker, Looker Studio, Tableau, Power BI, Evidence).  They are responsive to newly arriving data in a cost-optimal manner and require minimal ongoing mainenance or monitoring.

## Why?

- Inbound data not optimised for modelling, query or analytics
- Optimised for 
    - schema stability w/ extensibility
    - storage
    - table singularity
- if you find yourself doing anything difficult and annoying over and over again 

## How?

- native BigQuery/GCP tools
- SQL-based data profiling
- nested-content-derived schema generation
- efficient automation 
- extensibility
- built on the foundations of the bqtools functional library for BigQuery

## Where?

- installed in the inbound BigQuery dataset or optionally an alternative.



