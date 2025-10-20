# Primary db 생성

**bash_profile 수정**

```bash
vi .bash_profile

----------------------------------------------------------------------------
# .bash_profile
# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/.local/bin:$HOME/bin

export PATH
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=/u01/app/oracle/product/19.3.0/dbhome_1
export ORA_INVENTORY=/u01/app/oraInventory
export ORACLE_SID=PROD
#export TNS_ADMIN=/u01/app/oracle/product/19.3.0/dbhome_1/network/admin
export PATH=$PATH:$HOME/bin:$ORACLE_HOME/bin
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib
export CLASSPATH=$ORACLE_HOME/JRE:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib
export NLS_LANG=KOREAN_KOREA.AL32UTF8

alias sys='export ORACLE_SID=PROD; sqlplus sys/oracle_4U as sysdba'

NLS_LANG=american_america.we8iso8859p15
NLS_DATE_FORMAT='RRRR/MM/DD:HH24:MI:SS'
export NLS_LANG
export NLS_DATE_FORMAT
----------------------------------------------------------------------------

source .bash_profile
```

**데이터베이스를 생성할 디렉토리 생성**

```bash
mkdir -p /u01/app/oracle/oradata/PROD

cd /u01/app/oracle/oradata/PROD

mkdir disk1 disk2 disk3 disk4 disk5

ls
--> disk1  disk2  disk3  disk4  disk5
```

**초기화 파라미터 파일 생성**

```bash
cd $ORACLE_HOME/dbs
pwd

/u01/app/oracle/product/19.3.0/dbhome_1/dbs

vi initPROD.ora
----------------------------------------------------------------------------

# 아래 내용을 수정해 작성
db_name=PROD
compatible=19.3.0                   
memory_target=1G
processes=300                       
undo_management=AUTO
undo_tablespace=UNDOTBS1
remote_login_passwordfile=EXCLUSIVE
db_unique_name=PROD
control_files=('/u01/app/oracle/oradata/PROD/disk1/ctrl1.ctl',      
               '/u01/app/oracle/oradata/PROD/disk2/ctrl2.ctl',
               '/u01/app/oracle/oradata/PROD/disk3/ctrl3.ctl');
```

**db nomount**

위에서 생성한 pfile로 nomount

```bash
source /home/oracle/.bash_profile

echo $ORACLE_SID
--> PROD

sqlplus / as sysdba

startup nomount pfile=$ORACLE_HOME/dbs/initPROD.ora
```

**create database 스크립트 수행**

- database 생성 시 datafile, controlfile, redo log file, parameter file, archive log file, password file이 생성된다
- 수동 생성 시 datafile, controlfile, redo log file 3가지가 생성된다

```sql
create database PROD
user sys identified by oracle
user system identified by oracle
datafile '/u01/app/oracle/oradata/PROD/disk1/system01.dbf' 
size 100M autoextend on maxsize unlimited extent management local
sysaux 
datafile '/u01/app/oracle/oradata/PROD/disk2/sysaux01.dbf' 
size 50M autoextend on maxsize unlimited
default temporary tablespace temp
tempfile '/u01/app/oracle/oradata/PROD/disk3/temp01.dbf' 
size 50M autoextend on maxsize unlimited
undo tablespace undotbs1
datafile '/u01/app/oracle/oradata/PROD/disk4/undotbs01.dbf' 
size 50M autoextend on maxsize unlimited
logfile 
group 1 ('/u01/app/oracle/oradata/PROD/disk4/redoG1M1.rdo',
         '/u01/app/oracle/oradata/PROD/disk5/redoG1M2.rdo') size 100M,
group 2 ('/u01/app/oracle/oradata/PROD/disk4/redoG2M1.rdo',
         '/u01/app/oracle/oradata/PROD/disk5/redoG2M2.rdo') size 100M,
group 3 ('/u01/app/oracle/oradata/PROD/disk4/redoG3M1.rdo',
         '/u01/app/oracle/oradata/PROD/disk5/redoG3M2.rdo') size 100M,
group 4 ('/u01/app/oracle/oradata/PROD/disk4/redoG4M1.rdo',
         '/u01/app/oracle/oradata/PROD/disk5/redoG4M2.rdo') size 100M,
group 5 ('/u01/app/oracle/oradata/PROD/disk4/redoG5M1.rdo',
         '/u01/app/oracle/oradata/PROD/disk5/redoG5M2.rdo') size 100M;
      
   
select status from v$instance;

STATUS
------------
OPEN        
```

**data dictionary 생성 스크립트 수행**

```sql
@$ORACLE_HOME/rdbms/admin/catalog.sql
@$ORACLE_HOME/rdbms/admin/catproc.sql

connect  system/oracle
@$ORACLE_HOME/sqlplus/admin/pupbld.sql
```

**유저 생성**

```sql
create user scott identified by tiger;
grant dba to scott;

connect scott/tiger

SCOTT @ PROD > @/home/oracle/demo.sql
```

**리스너 설정**

listener.ora 파일과 tnsnames.ora 파일을 수정

```sql
cd /u01/app/oracle/product/19.3.0/dbhome_1/network/admin
vi listener.ora

------------------------------------------------------------------------
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.13.198)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )
SID_LIST_LISTENER=
  (SID_LIST =
    (SID_DESC =
      (ORACLE_HOME=/u01/app/oracle/product/19.3.0/dbhome_1)
      (SID_NAME=PROD)
     )
   )
------------------------------------------------------------------------

# tnsnames.ora
PROD =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.13.198)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = PROD)
    )
  )
------------------------------------------------------------------------

# 리스너 재시작
lsnrctl stop
lsnrctl start

# 접속 확인
sqlplus scott/tiger@PROD
```

**패스워드 파일 생성**

```sql
cd $ORACLE_HOME/dbs

orapwd file=**orapwPROD** password=oracle format=12 force=y

ls -l orapwPROD
-rw-r-----. 1 oracle oinstall 2048 10월  2 14:18 orapwPROD

# 접속 테스트
sqlplus sys/oracle@PROD as sysdba
```