# SQuaRE QA database

## Introduction
  
  SQuaRE is designing a QA database to store summary information 
  and QA metrics to be used in the context of the verification datasets. The current schema supports single visit processing and eventually  will be extended to co-add processing. 
  
  There are three sets of tables, for QA summary information, metrics, and for process execution.
  
  Changes can be made in MySQLWorkbench (http://dev.mysql.com/downloads/workbench/) - it installs easily in several platforms. Clone this repo, open the sqa.mwb file, make your changes and export to SQL. Do not change the SQL directly.

## General guidelines
  
  - Store summary information and QA metrics
  - QA metrics must easily extended 
  - Must be optimized for fast and interactive QA visualization
  - It has to be camera agnostic, should support at least SDSS, CFHT and DECam images processed by the LSST stack and QA tasks that will compute the summary information and QA metrics 
  - TDB: data insgestion configuration should be responsible for the mapping of camera specific  
    header key values to (table, property) in the schema

## Database Model

https://github.com/lsst-sqre/qa-database/blob/master/sqa.pdf

## Sample queries

Give me all datasets processed, run numbers, process date, status and who processed

For run=xxxx, give me the fraction of processed ccd failures

For run=xxxx, give me the list of visits with failures

Give me filter, exptime, zd, airmass, ha, median of fwhm, ellipticity, sky_bkg, ra_scatter, dec_scatter of all failed visits

Give me all the process ccd logs of failed ccds in visit=yyyy
 

## References
  - LSE-63 Data Quality Assurrance Plan
  - LSST Database Schema, baseline version (https://lsst-web.ncsa.illinois.edu/schema/index.php?sVer=baseline)
  - pipeQA
  - HSC Database schema v1.0 
  - DES Quick Reduce and DES operations database
