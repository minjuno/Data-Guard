# Read-Only Standby & Active Data Guard

### Read-Only Standby

Standby db를 read only모드로 열어서 reporting 작업을 Standby에서 수행해 Primary db부하를 줄일 수 있다.

예를 들어 사용자가 데이터를 분석하거나 통계, 집계 쿼리를 수행할 때 Primary db 대신 Standby에서 수행하면 Primary의 자원을 트랜잭션 처리에 집중시킬 수 있다.

하지만, read only 모드일 때도 아카이브 로그는 계속 전송되고 복구 프로세스는 중지되기 때문에 데이터는 최신 상태에서 계속 뒤쳐지게 된다.

**Standby database를 read only 모드로 전환**

```sql
shutdown immediate
startup mount

alter database open read only;
```

**복구를 다시 적용하려면 아래를 수행한다**

```sql
shutdown immediate
startup mount

alter database recover managed standby database disconnect from session;
```

### Active Data Guard

11g부터 ADG 기능을 도입했다.

Standby db를 read only모드로 열어두면서 Real-Time Apply를 수행할 수 있다.

Real-Time apply는 Standby db가 아카이브 로그가 아니라 Standby Redo log에서 직접 읽어 즉시 복구를 적용한다

즉, 아카이브가 완료되기를 기다리지 않아 적용 지연이 크게 감소한다.

**ADG 적용**

```sql
shutdown immediate
startup mount

alter database open read only;
alter database recover managed standby database using current logfile disconnect;
```

- using current logfile : Real-Time apply
- Disconnect  : 백그라운드에서 apply 수행

### 테스트

**Standby db에 real-time apply를 적용시킨다**

```sql
sqlplus / as sysdba

SYS @ SBDB > shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.

SYS @ SBDB > startup mount

ORACLE instance started.

Total System Global Area 1073737800 bytes
Fixed Size                  8904776 bytes
Variable Size             641728512 bytes
Database Buffers          415236096 bytes
Redo Buffers                7868416 bytes
Database mounted.

SYS @ SBDB > alter database open read only;

Database altered.

SYS @ SBDB > ALTER DATABASE RECOVER MANAGED STANDBY DATABASE USING CURRENT LOGFILE DISCONNECT;

Database altered.
```

**실시간 동기화 확인**

```sql
# Primary db
connect scott/tiger 

SCOTT @ PROD > update emp
               set sal = 0; 
  
14 rows updated.

SCOTT @ PROD > commit;

Commit complete.

------------------------------------------

# Standby db
SCOTT @ SBDB > select ename, sal from emp;

ENAME             SAL
---------- ----------
KING                0
BLAKE               0
CLARK               0         
...               
```