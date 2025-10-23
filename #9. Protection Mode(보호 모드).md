# Protection Mode(보호 모드)

### Primary database의 Protection Mode의 3가지 모드

**Maximum Protection**

- 데이터 손실 제로 보장 (Zero Data Loss)
- Primary DB의 트랜잭션은 최소 1개 이상의 Standby DB의 online redo 로그에 기록되기 전까지 커밋되지 않는다
- Standby 사용 불가 시 Primary도 shutdown된다

**Maximum Availability**

- Primary DB의 트랜잭션은 redo 정보가 최소 1개 이상의 Standby의 online redo 로그에 기록되기 전까지 커밋하지 않는다 (Maximum Protection과 동일)
- Standby가 사용 불가능하면 Standby가 복구되기 전까지 Maximum Performance로 운영된다

**Maximum Performance** 

- Primary DB의 트랜잭션은 redo 정보가 online redo 로그에 기록되고 바로 커밋한다
- Standby 서버로의 redo 정보는 비동기 방식으로 전송된다
- Standby의 장애가 Primary에 영향을 주지 않는다

**모드 스위칭**

새로 Standby db를 생성하면 Primary db는 기본값으로 maximum performance 모드이다.

```sql
select protection_mode from v$database;

PROTECTION_MODE
--------------------
MAXIMUM PERFORMANCE
```

```sql
# Maximum Availity
alter system set log_archive_dest_2='SERVICE=SBDB AFFIRM SYNC VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=SBDB';
alter database set standby database to maximize availability;

# Maximum Performance
alter system set log_archive_dest_2='SERVICE=SBDB NOAFFIRM ASYNC VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=SBDB';
alter database set standby database to maximize performance;

# Maximum Protection
alter system set log_archive_dest_2='SERVICE=SBDB AFFIRM SYNC VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=SBDB';
shutdown immediate
startup mount
alter database set standby database to maximize protection;
alter database open;
```
