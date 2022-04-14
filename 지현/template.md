# Template

# 06. DML 튜닝 
<br/>

### 6.2.2 Direct Path Insert  
### 일반적인 INSERT 가 느린 이유  
1. 데이터를 입력할 수 있는 블록을 Freelist에서 찾는다.  
2. Freelist에서 할당받은 블록을 버퍼캐시에서 찾는다.  
3. 버퍼캐시에 없으면, 데이터파일에서 읽어 버퍼캐시에 적재한다.  
4. INSERT 내용을 Undo 세그먼트에 기록한다.  
5. INSERT 내용을 Redo 로그에 기록한다.  
  
#### 🥝 HWM(High Water Mark) :  
1) 저장공간을 갖는 세그먼트 영역에서 사용한 적이 있는 Block과 사용한 적이 없는 Block 의 경계점을 의미한다.  
2) Block은 위에서 부터 채워진다.  
3) 데이타파일은 HWM을 가지지 않으며, 세그먼트만이 HWM를 가진다.  
  
#### 🍒 Freelist :  
- 테이블 HWM(High Water Mark) 아래쪽에 있는 블록 중 데이터 입력이 가능한 블록을 목록으로 관리하는데, 이를 'Freelist' 라고 함  
<br/>  
  
  
### Direct Path Insert 방식이 빠른 이유  
1. Freelist를 참조하지 않고 HWM 바깥 영역에 데이터를 순차적으로 입력한다.  
2. 블록을 버퍼캐시에서 탐색하지 않는다.  
3. 버퍼캐시에 적재하지 않고, 데이터파일에 직접 기록한다.  
4. Undo 로깅을 안 한다.  
5. Redo 로깅을 안 하게 할 수 있다.  
- 테이블을 아래와 같이 nologgin 모드로 전환한 상태에서 Direct Path Insert 를 하면 된다.  
```sql
-- t는 Alias
ALTER TABLE t NOLOGGING;
```  
<br/>  

### Array Processing도 Direct Path Insert 방식으로 처리할 수 있다.  
append_values 힌트를 사용하면 된다.  

```sql
DECLARE
  CURSOR c IS
    SELECT * 
	  FROM SOURCE;
  TYPE typ_source IS TABLE OF c%ROWTYPE;
  l_source typ_source;
  
BEGIN
  OPEN c;   -- 커서 오픈
  
  LOOP
    FETCH c BULK COLLECT INTO l_source LIMIT 10000; -- 커서 실행
    
    FORALL i IN l_source.first..l_source.last
    
    INSERT /*+ append_values */ INTO TARGET VALUES l_source(i);
    
    EXIT WHEN c%NOTFOUND;
  END LOOP;
  
  CLOSE c;  -- 커서 닫기
  
  COMMIT;
END;
```  
<br/>  
