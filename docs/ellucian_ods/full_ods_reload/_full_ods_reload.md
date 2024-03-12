## Overview

UO periodically clones Banner PROD data to Banner TEST, giving developers and users refreshed 'testing' data.  Historically, this occurs roughly every 8 months.

After a Banner PROD-to-TEST clone, ODS environments fed by Banner TEST (IOED & IOET) require a Full ODS Reload.  This is a fairly complicated process which involves a reload of replicated Banner data, followed by a reload of replication-derived transformed data.  

Many of the steps outlined can take hours, require attentive monitoring, orchestration, and troubleshooting, resulting in the overall process lasting up to 2 weeks.

This process has historically been performed by DBAs, but starting 2/2024 is managed by DDS Data Engineers.

## Preliminary Actions

These steps outline actions taken by DDS DEs to set/validate temporary configurations required for `Restaging Materialized Views` and `Reloading Reporting Tables`.

1. **Verify removal of scheduled IA_Admin Jobs**:(1)
{ .annotate }
    1. IOED/T needs all activity stopped before the clone process begins; DBAs manually remove them.            
    
        These frequency-scheduled jobs will be re-instutited after a success full reload, emulating their schedule/configuration from IOEP.


    ``` sql
    select 'GOOD TO GO'
    from dual 
    where not exists (select * 
				      from sys.dba_jobs 
				      where sys.dba_jobs.job not in (select job 
                                                     from sys.dba_jobs_running));	
    ```

2. **Verify DDS DEs granted 'DBA' role**:(1)
{ .annotate }
    1. Temporarily granted by DBA staff to DDS DEs to perform subsequent steps of the full reload process

    ``` sql
       select 'GOOD TO GO'
       from dba_role_privs
       where grantee       = user
         and granted_role  = 'DBA'
    ```



3. **Change default namespaces to MVLOG for Banner schemas**:(1)
{ .annotate }
    1. `MVLOG` is an Oracle tablespace with additional storage compared to the normal module-specific Banner tablespaces.  
    
        While normally MView logs are purged on an hourly/daily frequency, the full reload process may take up to 1.5 weeks resulting in larger-than-normal MView logs accumulated, impacting Oracle FLASH memory allocation to the point of failure.  
        
        After a full reload is complete, this change is reverted.

    ``` sql
    alter user FIMSMGR default tablespace MVLOG;
    alter user GENERAL default tablespace MVLOG;
    alter user TAISMGR default tablespace MVLOG;
    alter user ALUMNI default tablespace MVLOG;
    alter user SATURN default tablespace MVLOG;
    alter user FAISMGR default tablespace MVLOG;
    alter user PAYROLL default tablespace MVLOG;
    alter user POSNCTL default tablespace MVLOG;
    alter user HOUSMGR default tablespace MVLOG;   
    ```

4. **Restage SPRIDEN**(1):
{ .annotate }
    1. This serves as a 'smoke-test' to confirm correct configurations; validated in subseqent steps.

    In [IA_Admin](https://confluence.uoregon.edu/display/mid/ODS-EDW+administrative+web+interface){:target="_blank"}, restage the `SATURN.SPRIDEN` ODS Materialized View:
    
    ??? note annotate "Steps to Restage Single Materialized View(s)" 
        <iframe src="https://scribehow.com/embed/Safari_and_Microsoft_Teams_work_or_school_Workflow__VGlDBNbBQX6Zqzj0R4R8KQ?skipIntro=true&removeLogo=true" width="100%" height="640" allowfullscreen frameborder="0"></iframe>

5. **Confirm Configuration**(1):
{ .annotate }
    1. In the Banner PROD-to-TEST clone process, many configurations from the Banner-side are altered and re-instated.  
    To ensure proper configurations for Banner-to-IOEx DB Links, Snapshots and MViews, reviewing results from the previous step are neccessary

    Run the following in Banner TEST to confirm the Snapshots configuration is registered correctly to the environment for the previous step (IOED or IOET):

    ``` sql
    select l.snapshot_id,
           owner,
           name,
           substr(snapshot_site, 1, 30) snapshot_site,
           to_char(current_snapshots, 'mm/dd/yyyy hh24:mi:ss') current_snapshots
    from dba_registered_snapshots r, dba_snapshot_logs l
    where r.snapshot_id = l.snapshot_id(+)
      and name = 'SPRIDEN';
    ```

    The results should look like below. `CURRENT_SNAPSHOTS` should reflect Restaging SPRIDEN in previous step:

    |SNAPSHOT_ID|OWNER  |NAME   |SNAPSHOT_SITE    |CURRENT_SNAPSHOTS  |
    |-----------|-------|-------|-----------------|-------------------|
    |4108       |SATURN |SPRIDEN|IOED.UOREGON.EDU |03/11/2024 08:32:23|

    Next, run the following in (in Banner TEST) to confirm the registered views are properly configured:

    ``` sql
    select t.owner, 
       t.name,
       t.mview_site,
       t.can_use_log
    from dba_registered_mviews t 
    where t.name = 'SPRIDEN';
    ```
    The results should look like below. `MVIEW_SITE` should reflect the IOEx environment SPRIDEN was restaged in:

    |OWNER  |NAME    |MVIEW_SITE       |CAN_USE_LOG |
    |-------|--------|-----------------|------------|
    |SATURN |SPRIDEN |IOED.UOREGON.EDU |YES         |


## Replication Reload

6. **Restage by Schemas**(1):
{ .annotate }
    1. There is a specific order different schemas need to be restaged.  Upon the completion of a restaged schema, there are often errors in the Control Reports that can be ignored, and others that cannot.  These errors need to be addressed before moving on to the next schema.

    Restage the following schemas, ***in order***.  
    
    Before progressing to the next schema, review the Control Reports for errors:

