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
- TDB: data insgestion configuration should be responsible for the mapping of camera specific header key values to (table, property) in the schema

## Database Model

https://github.com/lsst-sqre/qa-database/blob/master/sqa.pdf

## Sample queries

- Give me all processed datasets, run numbers, date, duration, status and who processed

```sql 
SELECT d.name, 
       r.run_id, 
       r.start, 
       r.end - r.start as duration, 
       r.status, 
       u.username
FROM run r 
INNER JOIN dataset d ON r.dataset_id=d.dataset_id
INNER JOIN user u ON r.user_id=u.user_id;
```
- For run=xxxx, give me the fraction of processed ccd failures

- For run=xxxx, give me the footprint (i.e. corners in sky coordinates of all processed ccds) 

- For run=xxxx, give me the list of visits with failures

- Give me filter, exptime, zd, airmass, ha, median of fwhm, ellipticity, sky_bkg, ra_scatter, dec_scatter of all failed ccds 
 
- Give me all the process ccd logs of failed ccds in visit=yyyy

- Give me src catalog and image files for ccd=1, visit=yyyy procesed by run=xxxx 
(cannot be done in sql, but we can return the output_dir and then use the butler to get files giving the ccd and visit) 

- Give me distributions of the median scatter in ra and dec of all runs that processed dataset=zzz, the version of the stack, the configuration file used, from date=yyyy-mm-dd, and with at least 1000 ccds processed

- Recover DECam image quality history from date=yyyy-mm-dd and date=yyyy-mm-dd looking at the most recent runs 
 

## References
  - LSE-63 Data Quality Assurrance Plan
  - LSST Database Schema, baseline version (https://lsst-web.ncsa.illinois.edu/schema/index.php?sVer=baseline)
  - pipeQA
  - HSC Database schema v1.0 
  - DES Quick Reduce and DES operations database
