# Flashback Database (플래시백을 이용한 복구)

### Failover 후 원본 Primary를 재활용할 때 필요한 이유

switchover 또는 switchback 외 failover은 Primary를 완전히 교체하는 작업이므로 기존 Primary 데이터베이스는 Standby로 바로 전환할 수 없게 된다

Flashback Database 기능이 활성화되어 있지 않으면 기존 Primary는 더 이상 사용할 수 없고, 새로 Standby로 재구성해야 한다(Manual / rman duplicate)

반대로 Primary에 Flashback database를 미리 활성화해두면 failover 발생 시 기존 Primary를 failover 직전 시점으로 flashback 시켜 빠르게 새로운 Standby로 전환할 수 있다.

또한 Standby db에도 flashback을 켜둘 수 있다. switchover 후 역할이 바뀌어도 항상 두 DB가 Flashback 복구 가능 상태로 유지된다.

### Flashback 테스트

**Primary db에서 수행**

```sql
SCOTT @ PROD > select systimestamp from dual;

SYSTIMESTAMP
---------------------------------------------------------------------------
24-OCT-25 11.41.15.302491 PM +09:00

SCOTT @ PROD > delete from emp;
SCOTT @ PROD > commit;

SCOTT @ PROD > alter system switch logfile;

System altered.
```

**Standby에서 확인**

```sql
SCOTT @ SBDB > select * from emp;

no rows selected
```

**Primary에서 테이블을 flashback 가능한 상태로 변경**

```sql
SCOTT @ PROD > alter table emp enable row movement;

Table altered.

SCOTT @ PROD > flashback table emp to timestamp
               ( systimestamp - interval '4' minute );
               
Flashback complete.

SCOTT @ PROD > select count(*) from emp;

  COUNT(*)
----------
        14
        
SCOTT @ PROD > alter system switch logfile;

System altered.
```

**다시 Standby에서 확인**

```sql
SCOTT @ SBDB > select count(*) from emp;

  COUNT(*)
----------
        14
```

**Flashback 정보는 아래 글 참고**

[https://junobaboda.tistory.com/9](https://junobaboda.tistory.com/9)