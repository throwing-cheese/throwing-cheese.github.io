# Extract Data about Oracle Fusion Scheduled Requests

## Release 12 Scheduled Request SQL

In the old Release 12 world it was possible to extract the schedule information about a scheduled requests, such as how often it runs, on which days of the week, on which hour etc. etc.

You can see more details about how that was done via this SQL query:

[oracle-e-business-suite-sql-scripts > sa > sa-concurrent-requests-scheduled.sql](https://github.com/throwing-cheese/oracle-e-business-suite-sql-scripts/blob/main/sa/sa-concurrent-requests-scheduled.sql)

## Oracle Fusion Scheduled Request Column

However, extracting that type of information from Oracle Fusion was something I'd not found out how to do until recently.

In one of my jobs someone circulated a zip file containing some useful SQL queries which could be used to extract various bits of info about concurrent requests in Fusion.

The main fusion `request_history` table contains all sorts of useful information about a concurrent request in Oracle Fusion.

### The 'adhocschedule' Column

There's a column in the `request_history` table called `adhocschedule`, however, you can't just include it in your query in the format in which it's stored in the database.

#### utl_raw.cast_to_varchar2

Instead, you have to do use `utl_raw.cast_to_varchar2(adhocschedule)`

Read more info about [utl_raw.cast_to_varchar2](https://docs.oracle.com/database/timesten-18.1/TTPLP/u_raw.htm#TTPLP71506)

##### Extracting XML from 'adhocschedule'

Once you have done that. you still end up with something hard to use - e.g.

`¿<¿?¿x¿m¿l¿ ¿v¿e¿r¿s¿i¿o¿n¿ ¿=¿ ¿'¿1¿.¿0¿'¿ ¿e¿n¿c¿o¿d¿i¿n¿g¿ ¿=¿ ¿'¿U¿T¿F¿-¿8¿'¿?¿>¿ ¿<¿s¿c¿h¿e¿d¿u¿l¿e¿ ¿x¿m¿l¿n¿s¿:¿x¿s¿i¿=¿"¿h¿t¿t¿p¿:¿/¿/¿w¿w¿w¿.¿w¿3¿.¿o¿r¿g¿/¿2¿0¿0¿1¿/¿X¿M¿L¿S¿c¿h¿e¿m¿a¿-¿i¿n¿s¿t¿a¿n¿c¿e¿"¿ ¿n¿a¿m¿e¿=¿"¿j¿o¿b¿4¿H¿o¿u¿r¿l¿y¿_¿s¿c¿h¿e¿d¿u¿l¿e¿"¿ ¿x¿m¿l¿n¿s¿=¿"¿h¿t¿t¿p¿:¿/¿/¿x¿m¿l¿n¿s¿.¿o¿r¿a¿c¿l¿e¿.¿c¿o¿m¿/¿s¿c¿h¿e¿d¿u¿l¿e¿r¿"¿>¿<¿d¿e¿s¿c¿r¿i¿p¿t¿i¿o¿n¿>¿S¿c¿h¿e¿d¿u¿l¿e¿ ¿f¿o¿r¿ ¿j¿o¿b¿4¿H¿o¿u¿r¿l¿y¿<¿/¿d¿e¿s¿c¿r¿i¿p¿t¿i¿o¿n¿>¿<¿d¿i¿s¿p¿l¿a¿y¿-¿n¿a¿m¿e¿>¿j¿o¿b¿4¿H¿o¿u¿r¿l¿y¿_¿s¿c¿h¿e¿d¿u¿l¿e¿<¿/¿d¿i¿s¿p¿l¿a¿y¿-¿n¿a¿m¿e¿>¿<¿r¿e¿c¿u¿r¿r¿e¿n¿c¿e¿>¿<¿c¿o¿u¿n¿t¿>¿0¿<¿/¿c¿o¿u¿n¿t¿>¿<¿i¿c¿a¿l¿-¿e¿x¿p¿r¿e¿s¿s¿i¿o¿n¿>¿F¿R¿E¿Q¿=¿H¿O¿U¿R¿L¿Y¿;¿B¿Y¿M¿I¿N¿U¿T¿E¿=¿3¿0¿;¿I¿N¿T¿E¿R¿V¿A¿L¿=¿1¿;¿<¿/¿i¿c¿a¿l¿-¿e¿x¿p¿r¿e¿s¿s¿i¿o¿n¿>¿<¿/¿r¿e¿c¿u¿r¿r¿e¿n¿c¿e¿>¿<¿/¿s¿c¿h¿e¿d¿u¿l¿e¿>`

This will make the data more usable:

```sql
replace(replace(replace(utl_raw.cast_to_varchar2(rh.adhocschedule),chr(0),''),chr(10),''),chr(13),'') adhocschedule_cast
```

Result:

```xml
<?xml version = '1.0' encoding = 'UTF-8'?><schedule xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" name="job4Hourly_schedule" xmlns="http://xmlns.oracle.com/scheduler"><description>Schedule for job4Hourly</description><display-name>job4Hourly_schedule</display-name><recurrence><count>0</count><ical-expression>FREQ=HOURLY;BYMINUTE=30;INTERVAL=1;</ical-expression></recurrence></schedule>
```

From there, by chopping up the data, you can then extract useful information about the scheduled job, as the info is held in various XML tags.

## Oracle Fusion Scheduled Requests SQL

See the **REQUEST HISTORY - SCHEDULED** heading in this SQL file:

[oracle-fusion-cloud-sql-scripts/sa/sa-requests-history.sql](https://github.com/throwing-cheese/oracle-fusion-cloud-sql-scripts/blob/main/sa/sa-requests-history.sql)

## Note about the 'ical-expression' tag

The above query contains this line:

`and instr(replace(utl_raw.cast_to_varchar2(adhocschedule),chr(0),''),'<ical-expression>') = 0 -- exclude jobs run by system %APPID accounts, plus the "EssBipJob" BI Publiser job`

Where the adhocschedule contains the `<ical-expression>`XM tag, it is usually the case that the job was run by a user account ending in "APPID" e.g. FUSION_APPS_HCM_ESS_APPID

As per this Cloud Customer Connect post
https://community.oracle.com/customerconnect/discussion/comment/888413#Comment_888413

Quoting an Oracle employee:

- %APPID - These are internal application users, their purpose is to authorize an application to exchange/access another application. 
- These users do not actually have functional access to your data (eg you cannot login with any of these users to access business flows).
- The user is an internal account for Fusion Applications functionality.
- These accounts are not visible in Security Console.
- These accounts are owned by Oracle software.
- Most those users are used by program, no human know their password.

Someone on the post noted that the `created_by` column in the `per_users` table for usernames ending in APPID sometimes contained the usernames of non system accounts. More feedback from another Oracle employee:

- The create_by user in the table is the user who brought the data from ldap server into per_users table
- not the user who create the APPID user in ldap server.
- the create_by user in ldap server will be an system user, but cloud customer does not have access to ldap server
- for example, the user who run retrieve latest ldap changes process, will bring data into per_users table, and that user can be the create_by user in the table

Other `<ical-expression>` jobs are sometimes submitted by the "FAAdmin" user, such as:

FndExtMgrDigestPurgeJob - Job for purging Extension Manager old Digests
FndIDCSSyncNotifyServiceJob - Job for FA IDCS Users Sync
FndOSCSBulkIngestJob - Job for processing errored rows to Oracle Search Cloud Service
FndOSCSAvailabilityJob - Job to check Search Cloud Service availability
FndOSCSAttachmentIngestJob - Job for ingesting attachments to Oracle Search Cloud Service
BPMSMCAttachmentUploadServiceJob - Job to Upload Attachments of BPM Archive Tasks.
BPMSMCTranslationServiceJob - Job to Process Translation of BPM Archive Tasks.
BPMSMCDataExtractServiceJob - Job to Extract Workflow Tasks for Archive.

Also, some jobs are submitted by "regular" non system users where the `<ical-expression>` XML tag is populated in the adhocschedule column. The main job seems to be the "EssBipJob" under the "BI Publisher" Product name, so these are scheduled BI Publisher jobs.

Therefore, to exclude `<ical-expression>` jobs, this line is included

and instr(replace(utl_raw.cast_to_varchar2(adhocschedule),chr(0),''),'<ical-expression>') = 0 -- exclude jobs run by system %APPID accounts, plus the "EssBipJob" BI Publiser job
