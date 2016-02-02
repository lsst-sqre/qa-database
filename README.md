# SQuaRE QA database

## Introduction
  
  SQuaRE is designing a QA database to store metrics and summary information 
to be used in the context of the verification datasets. The current schema supports single visit processing and eventually  will be extended to co-add processing. 
  
  There are three sets of tables for QA metrics, summary information, and for process execution. The motivation for that is the characterization of each single visit at diferent levels of information. The QA metrics are computed by the "QA pipeline" and only the results are stored. For metrics that failed one can look at aggregated information at the cdd or visit levels stored in the database. The ccd summary information is computed from the qa_source table, which store properties of high S/N point sources (not decided yet if we want to keep this table in the database, it depends on  its size). If the full image and source catalog are required for futher inspection they  can be retieved from the process output_dir using the butler (or throught the webserv API). 
  
  This must be understood as a 'common model' for the different camera supported by the stack. A clear advantage of that is to compare metrics and results among datasets still at the database level (i.e using SQL). The mechisms for translating camera-specific metadata is being discussed.
  
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
- For run=xxxx, give me the fraction of ccd failures
```sql
SELECT rv.n_failed/rv.n_processed as failure_fraction 
FROM run_visit rv
INNER JOIN run r ON rv.run_id=r.run_id  
WHERE run_id = 'xxxx';
```
- For run=xxxx, give me the list of visits with failures
```sql
SELECT visit 
FROM visit v 
INNER JOIN run_visit rv ON v.visit_id=rv.visit_id
INNER JOIN run r ON rv.run_id=r.run_id
WHERE r.run_id='xxxx';
```

- Give me the footprint or run xxxx (i.e. corners in sky coordinates of all processed ccds) 
```sql
SELECT c.llra, 
       c.lldec, 
       c.urra, 
       c.urdec 
FROM ccd c 
INNER JOIN visit v ON c.visit_id=v.visit_id
INNER JOIN run_visit rv ON v.visit_id =rv.visit_id
INNER JOIN run r ON rv.run_id = r.run_id where run_id = 'xxxx';
```

- Give me filter, exptime, zd, airmass, ha, and the median fwhm, ellipticity, sky_bkg, ra_scatter, dec_scatter of all failed ccds in visit yyyy  

```sql
SELECT v.filter, 
       v.exptime, 
       v.zd, 
       v.airmass, 
       v.ha, 
       c.median_fwhm, 
       1.0-c.median_minor_axis/c.median_major_axis as median_ellipticity,
       c.median_sky_bkg,
       c.ra_scatter,
       c.dec_scatter
FROM visit v, 
     ccd c
WHERE v.visit_id = c.visit_id 
AND v.visit = 'yyyy';
```

TODO: aggregate these values per visit (make use of scisql_median() function to compute median in the database, avg() could be used as an alternative)

- Give me the process ccd logs of failed ccds in visit yyyy
```sql
SELECT log
FROM visit v, 
     ccd c
WHERE v.visit_id = c.visit_id 
AND v.visit = 'yyyy';
```

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
