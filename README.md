# SQuaRE QA database

## Changelog

v0.1 - Initial proposal
v0.2 - Using LSST database naming convention (in preparation)

## Introduction
  
  SQuaRE is designing a QA database to store metrics and summary information 
to be used in the context of the verification datasets. The current schema supports single visit processing and eventually  will be extended to co-add processing. 
  
  There are three sets of tables to gather the information at different levels:  1) QA metrics are computed by QA Tasks and only the results, conditions, thresholds and metric descriptions are stored in the database. For metrics that failed one can look at summary information for cdds and visits. 
  2) That summary information is computed from the properties of a subset of high S/N point sources also stored in the database.
  3) If the full image and source catalogs are required for futher inspection they can be retieved from the process output_dir using the butler. Configuration, code version and logs for each processed are also stored. 
  
  This must be understood as a 'common model' for the different camera supported by the stack. A clear advantage of that is to compare metrics and results accross different processes of the same dataset or accross different datasets. The mechanism for translating camera-specific metadata to this common model is still being discussed.
  
  Changes in the database model can be made using MySQLWorkbench (http://dev.mysql.com/downloads/workbench/) - it installs easily in several platforms. Clone this repo, open the sqa.mwb file, make your changes and export to SQL. Do not change the SQL directly.

## General guidelines
  
- Store QA metrics and summary information for ccds and visits
- QA metrics must easily extended (i.e without changing the schema) 
- Must be optimized for fast and interactive QA visualization
- It has to be camera agnostic, should support SDSS, CFHT, HSC and DECam images processed by the LSST stack
- TDB: data insgestion configuration should be responsible for the mapping of camera-specific header key values to (table, property) in the schema

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
- For run=xxxx, give me a list of visits with failures
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

- Give me filter, exptime, zd, airmass, ha, and the median fwhm, ellipticity, sky_bkg, ra_scatter, dec_scatter of all ccds in visit yyyy  
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

- Give me summary information for all visits processed by run xxxx (use scisql_median() function to aggregate values per visit)
```sql
SELECT v.visit,
       v.filter, 
       v.exptime, 
       v.zd, 
       v.airmass, 
       v.ha, 
       scisql_median(c.median_fwhm) as fwhm, 
       scisql_median(1.0-c.median_minor_axis/c.median_major_axis) as ellipticity,
       scisql_median(c.median_sky_bkg) as sky_bkg,
       scisql_median(c.ra_scatter) as ra_scatter,
       scisql_median(c.dec_scatter) as dec_scatter
FROM ccd c 
INNER JOIN visit ON c.visit_id = v.visit_id
INNER JOIN run_visit rv ON v.visit_id =rv.visit_id
INNER JOIN run r ON rv.run_id = r.run_id where run_id = 'xxxx'
GROUP BY v.visit,
         v.filter,
         v.exptime,
         v.zd,
         v.airmass,
         v.ha
ORDER BY v.visit;
```
- Give me the process ccd logs of failed ccds in visit yyyy
TODO: we need an status in ccd table

```sql
SELECT log
FROM visit v, 
     ccd c
WHERE v.visit_id = c.visit_id 
AND v.visit = 'yyyy'
AND c.status = 1;
```

- Give me src catalog and image files for ccd=1, visit=yyyy procesed by run=xxxx 
(cannot be done in sql, but if we return the output_dir then one can user the butler to get files giving the ccd and visit) 

- Give me median scatter in ra and dec for all visits in all runs that processed dataset=zzz, the version of the stack, the configuration file used, from date=yyyy-mm-dd with at least 1000 ccds processed in each visit
```sql
SELECT r.run_id as run,
       d.name as dataset,
       v.visit,
       scisql_median(c.ra_scatter) as ra_scatter,
       scisql_median(c.dec_scatter) as dec_scatter
FROM ccd c 
INNER JOIN visit ON c.visit_id = v.visit_id
INNER JOIN run_visit rv ON v.visit_id =rv.visit_id
INNER JOIN run r ON rv.run_id = r.run_id 
INNER JOIN dataset d ON r.dataset_id=d.dataset_id
WHERE d.name='zzzz'
GROUP BY v.visit
ORDER BY r.run_id;
```

- Recover DECam image quality history (e.g. fwhm and its scatter) from date=yyyy-mm-dd and date=yyyy-mm-dd looking at all runs that processed decam data 
```sql
SELECT r.run_id as run,
       d.name as dataset,
       v.visit,
       scisql_median(c.median_fwhm) as fwhm,
       scisql_median(c.mad_fwhm) as scatter
FROM ccd c 
INNER JOIN visit ON c.visit_id = v.visit_id
INNER JOIN run_visit rv ON v.visit_id =rv.visit_id
INNER JOIN run r ON rv.run_id = r.run_id 
INNER JOIN dataset d ON r.dataset_id=d.dataset_id
WHERE d.camera= 'decam'
GROUP BY v.visit
ORDER BY r.run_id;
```


## References
  - LSE-63 Data Quality Assurrance Plan
  - LSST Database Schema, baseline version (https://lsst-web.ncsa.illinois.edu/schema/index.php?sVer=baseline)
  - pipeQA
  - HSC Database schema v1.0 
  - DES Quick Reduce and DES operations database
