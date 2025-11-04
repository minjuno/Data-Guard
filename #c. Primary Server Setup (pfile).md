# Primary Server Setup

### Logging

**primary db가 archive mode인지 확인**

```sql
select log_mode from v$database;

LOG_MODE
------------
NOARCHIVELOG
```

**noarchivelog mode일 때** 

```sql
shutdown immediate
startup mount

alter database archivelog;
alter database open;

# 로그 스위치를 발생해 아카이브 로그 생성 확인
alter system switch logfile;

select name from v$archived_log;

NAME
--------------------------------------------------------------------------------
/u01/app/oracle/product/19.3.0/dbhome_1/dbs/arch1_15_1214763347.dbf
```

**Force Logging 활성화**

```sql
alter database force logging;

alter database add supplemental log data
   (primary key, unique index) columns;
   
# force logging 활성화
select force_logging from v$database;

FORCE_LOGGING
---------------------------------------
YES
```

### 파라미터 설정

**DB_NAME, DB_UNIQUE_NAME 파라미터 확인**

```bash
show parameter db_name

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
db_name                              string      PROD

show parameter db_unique_name
NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
db_unique_name                       string      PROD
```

**initPROD.ora 파일 수정**

```bash
cd $ORACLE_HOME/dbs
vi initPROD/ora

# 아래 내용 추가
------------------------------------------------------------------------

standby_file_management=auto 
db_file_name_convert='/u01/app/oracle/oradata/SBDB', '/u01/app/oracle/oradata/PROD'
log_file_name_convert='/u01/app/oracle/oradata/SBDB', '/u01/app/oracle/oradata/PROD'
log_archive_dest_1='location=/home/oracle/PROD/arch valid_for=(all_logfiles,all_roles)'
log_archive_dest_2='service=SBDB LGWR SYNC AFFIRM valid_for=(online_logfiles,primary_role)'

fal_server=SBDB
fal_client=PROD
```

**파라미터 설명**

```bash
db_unique_name : standby에서 primary db 이름을 식별하기 위한 파라미터
standby_file_management=auto : Primary쪽에서 테이블스페이스 생성 시 Standby쪽에도 생성
db_file_name_convert : 순서대로 Standby쪽, Primary쪽 데이터파일의 위치
log_file_name_convert : Standby, Primary log file 위치
log_archive_dest_1 : Primary 아카이브 로그 파일 생성 위치
log_archive_dest_2 : Standby에 생성할 아카이브 로그 파일 위치
```

**spfile 생성, 파라미터 관련 디렉토리 생성**

```sql
mkdir -p /home/oracle/PROD/arch   
mkdir -p /home/oracle/PROD/flash

create spfile from pfile;
```

### Service Setup

primary, standby db의 엔트리가 각각의 tnsnames.ora 파일에 작성되어야 한다.

이 단계에서는 primary db의 tnsnames.ora 파일만 수정한다

```bash
cd $ORACLE_HOME/network/admin
vi tnsnames.ora

------------------------------------------------------------------------
PROD =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.13.198)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = PROD)
    )
  )

SBDB =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.13.199)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = SBDB)
    )
  )
```

### standby pfile 생성

**standby db의 파라미터 파일을 생성한다**

```bash
cd /tmp
vi initSBDB.ora

------------------------------------------------------------------------
db_name=PROD
compatible=19.3.0                   
memory_target=1G
global_names=true
db_block_size=8192
service_names=SBDB
job_queue_processes=10
processes=300                       
undo_management=AUTO
undo_tablespace=UNDOTBS1
remote_login_passwordfile=EXCLUSIVE
db_unique_name=SBDB
db_recovery_file_dest_size=4G
db_recovery_file_dest=/home/oracle/SBDB/flash
control_files = (/u01/app/oracle/oradata/SBDB/disk1/ctrl1.ctl ,
                 /u01/app/oracle/oradata/SBDB/disk2/ctrl2.ctl ,
                 /u01/app/oracle/oradata/SBDB/disk3/ctrl3.ctl )

standby_file_management=auto 
db_file_name_convert='/u01/app/oracle/oradata/PROD','/u01/app/oracle/oradata/SBDB'
log_file_name_convert='/u01/app/oracle/oradata/PROD','/u01/app/oracle/oradata/SBDB'
log_archive_dest_1='location=/home/oracle/SBDB/arch valid_for=(all_logfiles,all_roles)'
log_archive_dest_2='service=PROD LGWR SYNC AFFIRM valid_for=(online_logfiles,primary_role)'

fal_server=PROD
fal_client=SBDB
```
