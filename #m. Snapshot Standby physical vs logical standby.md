# Snapshot Standby/physical vs. logical standby

### Snapshot Standby

Snapshot Standby는 일시적으로 Standby를 일반 DB처럼 read-write 모드로 전환하는 기능이다.

Standby 모드로 다시 전환될 때 read-write모드 동안 발생한 변경 사항은 모두 사라진다.

**RAC환경이라면 하나의 인스턴스를 제외하고 모두 내린다**

```sql
shutdown immediate
startup mount

# 복구 적용 중지
alter database recover managed standby database cancel;

# snapshot standby로 전환
alter database convert to snapshot standby;
alter database open;

select flashback_on from v$database;

FLASHBACK_ON
--------------------
RESTORE POINT ONLY
```

**다시 physical standby로 전환**

```sql
shutdown immediate
startup mount

alter database convert to physical standby;

shutdown immediste
startup mount

alter database recover managed standby database disconnect;
```

### Physical standby vs. Logical standby

**Physical standby**

Primary db의 Redo log를 그대로 받아 Block 단위로 복제하는 방식

datafile 구조, SCN, DB 구조가 Primary와 완전히 동일하다

Redo apply 방식으로 복구 수행

Failover시 즉시 전환이 가능하다    

**Logical standby**

Primary db의 데이터를 sql 문장으로 변환해 Standby에 적용

데이터 내용은 동일하지만 물리적 구조는 다를 수 있다

Active Data Guard 지원 x