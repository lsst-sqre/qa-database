# SQuaRE QA database

## Introduction
  
  SQuaRE is designing a QA database to store summary information 
  and QA metrics,  to be used in the context of the verification datasets. The current schema supports single visit processing and eventually  will be extended to co-add processing. 
  
  Changes can be done in MySQLWorkbench (http://dev.mysql.com/downloads/workbench/), which installs easily in several platforms. Clone this repo, open the sqa.mwb file, make your changes and export to SQL. 
  Do not change the SQL directly.

## General guidelines
  
  - Store summary information and QA metrics
  - QA metrics must be easily extended  
  - Optimized for fast and interactive QA visualization
  - It has to be camera agnostic to be used in the context of the verification datasets
  - Support SDSS, CFHT and DECam images processed by the LSST stack, eventually a simplified
  pipeline called QA pipeline (TBD) wich computes the summary information and QA metrics 
  - TDB: Ingestion Task configuration is responsible for the mapping of camera specific  
    header key values to (table, property) in the schema

## Database Model

https://github.com/lsst-sqre/qa-database/blob/master/sqa.pdf

## References
  - LSE-63 Data Quality Assurrance Plan
  - LSST Database Schema, baseline version (https://lsst-web.ncsa.illinois.edu/schema/index.php?sVer=baseline)
  - HSC Database schema v1.0 
  - DES Quick Reduce and DES operations database
