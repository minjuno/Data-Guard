# 복구 적용 프로세스(Apply Process)

### **Recover**

db에서 발생한 변경 내역을 redo log를 적용해 데이터파일에 적용해 최신 상태로 만드는 것

### **Standby db에서 복구 시작 방법**

Standby db는Primary에서 보낸 Archive Redo Log를 읽어 데이터 파일에 반영(MRP 프로세스)

**MRP(Media Recovery Process)가 올라왔는지 확인**

```sql
select process, status from v$managed_standby;
PROCESS   STATUS
--------- ------------
ARCH      CONNECTED
DGRD      ALLOCATED
DGRD      ALLOCATED
ARCH      CONNECTED
ARCH      CONNECTED
ARCH      CONNECTED
**MRP0      WAIT_FOR_LOG**
DGRD      ALLOCATED
```

- **Foreground redo 적용 방식**
    - 세션이 복구가 끝날 때까지 점유된다

```sql
alter database recover managed standby database;
```

- **Background redo 적용 방식**
    - 세션을 즉시 반환하고 복구를 백그라운드 프로세스(MRP0)로 실행한다

```sql
alter database recover managed standby database disconnect;
```

### **복구 적용 중지**

```sql
alter database recover managed standby database cancel;
```

### **복구 지연 적용**

**아카이브 로그와 Standby 서버에 적용되는 시간 간격을 설정할 수 있다.**

```sql
alter database recover managed standby database cancel;
alter database recover managed standby delay 30 disconnect from session;

alter database recover managed standby nodelay[default] disconnect from session;
```

### **Real-Time apply**

Standby redo 로그를 구성했으면 아래 명령어를 통해 real-time apply 기능을 사용할 수 있다.

```sql
alter database read only;
alter database recover managed standby database using current logfile disconnect;
```
