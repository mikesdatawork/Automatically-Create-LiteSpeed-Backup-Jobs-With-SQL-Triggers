![MIKES DATA WORK GIT REPO](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_01.png "Mikes Data Work")        

# Automatically Create LiteSpeed Backup Jobs With SQL Triggers
**Post Date: February 7, 2008**        



## Contents    
- [About Process](##About-Process)  
- [SQL Logic](#SQL-Logic)  
- [Build Info](#Build-Info)  
- [Author](#Author)  
- [License](#License)       

## About-Process

<p>Automatically create a LiteSpeed backup job for each new database created just create a trigger on the master table (forbidden in sql 2005) which simply runs the following procedure upon updated with a new database name.</p>      


## SQL-Logic
```SQL
CREATE procedure [dbo].[sp_createbackupjob]
as 
/****** Object: Job [LITESPEED FULL BACKUP @newdb] Script Date: 04/14/2008 14:06:21******/
BEGIN TRANSACTION
DECLARE @ReturnCode INT
SELECT @ReturnCode = 0
/****** Object: JobCategory [[Uncategorized (Local)]]] Script Date: 04/14/2008 14:06:21******/
IF NOT EXISTS (SELECT name FROM msdb.dbo.syscategories WHERE name=N'[Uncategorized (Local)]' AND category_class=1)
BEGIN
EXEC @ReturnCode = msdb.dbo.sp_add_category @class=N'JOB', @type=N'LOCAL', @name=N'[Uncategorized (Local)]'
IF (@@ERROR 0 OR @ReturnCode 0) GOTO QuitWithRollback 
END 
declare @newdb varchar(50)
set @newdb = (select top 1 [name] from master..sysdatabases order by dbid desc)
declare @desc varchar(100)
set @desc = 'LITESPEED FULL BACKUP ' + @newdb
declare @mycommand varchar(300)
set @mycommand = 'exec master.dbo.xp_backup_database @database = ' + @newdb + ', @filename = N"E:\Backup\' + @newdb + ' LiteSpeed_Full.BKP", @backupname = N"' + @newdb + ' Backup", @init = 1, @with = N"SKIP", @with = N"STATS = 10"' 
DECLARE @jobId BINARY(16)
EXEC @ReturnCode = msdb.dbo.sp_add_job @job_name=@desc,
@enabled=1,
@notify_level_eventlog=0,
@notify_level_email=0,
@notify_level_netsend=0,
@notify_level_page=0,
@delete_level=0,
@description=@desc,
@category_name=N'[Uncategorized (Local)]',
@owner_login_name=N'sa', @job_id = @jobId OUTPUT
IF (@@ERROR 0 OR @ReturnCode 0) GOTO QuitWithRollback
/****** Object: Step [Full Backup] Script Date: 04/14/2008 14:06:22******/
EXEC @ReturnCode = msdb.dbo.sp_add_jobstep @job_id=@jobId, @step_name=N'Full Backup',
@step_id=1,
@cmdexec_success_code=0,
@on_success_action=1,
@on_success_step_id=0,
@on_fail_action=2,
@on_fail_step_id=0,
@retry_attempts=0,
@retry_interval=0,
@os_run_priority=0, @subsystem=N'TSQL',
@command=@MyCommand,
@database_name=@newdb,
@flags=2
IF (@@ERROR 0 OR @ReturnCode 0) GOTO QuitWithRollback
EXEC @ReturnCode = msdb.dbo.sp_update_job @job_id = @jobId, @start_step_id = 1
IF (@@ERROR 0 OR @ReturnCode 0) GOTO QuitWithRollback
EXEC @ReturnCode = msdb.dbo.sp_add_jobschedule @job_id=@jobId, @name=N'Backup',

@enabled=1,

@freq_type=4,
@freq_interval=1,
@freq_subday_type=1,
@freq_subday_interval=0,
@freq_relative_interval=0,
@freq_recurrence_factor=0,
@active_start_date=20050419,
@active_end_date=99991231,
@active_start_time=10000,
@active_end_time=235959
IF (@@ERROR 0 OR @ReturnCode 0) GOTO QuitWithRollback
EXEC @ReturnCode = msdb.dbo.sp_add_jobserver @job_id = @jobId, @server_name = N'(local)'
IF (@@ERROR 0 OR @ReturnCode 0) GOTO QuitWithRollback
COMMIT TRANSACTION
GOTO EndSave
QuitWithRollback:
IF (@@TRANCOUNT > 0) ROLLBACK TRANSACTION
EndSave: 
```



[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

[![Gist](https://img.shields.io/badge/Gist-MikesDataWork-<COLOR>.svg)](https://gist.github.com/mikesdatawork)
[![Twitter](https://img.shields.io/badge/Twitter-MikesDataWork-<COLOR>.svg)](https://twitter.com/mikesdatawork)
[![Wordpress](https://img.shields.io/badge/Wordpress-MikesDataWork-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

    
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Mikes Data Work](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_02.png "Mikes Data Work")

