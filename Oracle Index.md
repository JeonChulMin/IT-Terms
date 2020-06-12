# Oracle Index

- 인덱스는 DB 테이블에 있는 데이터를 빨리 조회하기 위한 용도로 DB 객체, 조회 기술이다.

- DB 테이블에 index를 생성하게 되면 index 테이블을 생성해 관리한다.

- index는 테이블에 있는 하나 이상의 컬럼으로 만들 수 있다.

- 가장 일반적인 B-tree 인덱스는 인덱스 키(인덱스로 지정할 컬럼 값)와 이 키에 해당하는 컬럼 값을 가진 테이블의 row가 저장된 주소 값으로 구성된다.

- index는 조회 성능을 높이기 위해 만든 객체인데 무분별하게 많이 만들면 CUD 작업 시에 부하가 발생해 성능을 저하시킨다.

- 인덱스는 기본키나 유일키와 같은 제약 조건을 지정하면 따로 인덱스를 생성하지 않아도 자동으로 생성된다.
- 인덱스 객체에 대한 정보를 확인하는 방법
  - USER_INDEXES, USER_IND_COLUMNS 데이터 딕셔너리 뷰에서 확인 가능



## 인덱스 생성

```SQL
CREATE INDEX 인덱스명 ON 테이블명(컬럼1, 컬럼2, ....);
CREATE UNIQUE 인덱스명 ON 테이블명(컬럼1, 컬럼2, ....); -- 컬럼의 중복값을 허용하지 않음
```

## 인덱스 조회

```SQL
SELECT * FROM 인덱스명 WHERE TABLE_NAME = '테이블명';
SELECT * FROM 인덱스명 WHERE TABLE_NAME = '인덱스명';
```

## 인덱스 삭제

```SQL
DROP INDEX 인덱스명;
```



## 인덱스 구조

 일반적인 DB 테이블과 비교해서보면 테이블은 컬럼이 여러 개이고, 데이터가 정렬되지 않고 입력된 순서로 들어간다. 반면 인덱스는 컬럼이 Key 컬럼(사용자가 지정한 컬럼)과 ROWID 컬럼, 두 개의 컬럼으로 이루어져 있다.

### ROWID (위치 주소) 구조

- 인덱스는 (데이터, 위치 주소) 쌍으로 저장되어 진다.
- Row 위치의 주소 구조는 아래와 같다.
  - 데이터 오브젝트 번호 / 파일 번호 / 블록 번호 / ROW 번호



## B-tree 인덱스 작동 원리

 예를 들어 아래와 같은 SQL문을 수행한다고 해보자.

```SQL
SELECT * FROM EMP WHERE EMPNO = 7369;
```

데이터 파일 안 블록이 100만개가 있다고 가정하고 SQL문을 수행

1) 서버 프로세스가 SQL 파싱 과정을 처리한 후 DB Buffer 캐시에 EMPNO가 7369인 정보가 있는지 먼저 확인한다.

2) 해당 정보가 캐시에 없다면 디스크 파일에서 7369를 가진 블록을 찾아서 DB Buffer 캐시로 가져온 뒤 정보를 제공한다.

#### INDEX를 사용하지 않는 경우

- 7369 정보가 어느 디스크 블록이 있는 지 알 수 없으므로 100만개 데이터 전부를 DB Buffer 캐시로 복사한 후 Full Scan으로 찾는다.
- (FULL SCAN: DB 테이블에 포함된 데이터를 처음부터 끝까지 검사하며 원하는 데이터를 찾아 추출하는 것을 의미)

#### INDEX를 사용한 경우

- WHERE절의 조건으로 준 컬럼이 Index의 Key로 생성되어 있는지 확인한 후, Index에 먼저 가서 7369가 어떤 ROWID를 가지고 있는지 확인한 뒤 찾아가서 정보를 DB Buffer 캐시에 복사한다.



## 인덱스 종류

### B-tree 인덱스

- OLTP(Online Transaction Processing), 실시간 트랜잭션 처리
- 실시간으로 데이터 입력과 수정이 일어나는 환경에 많이 사용

### BITMAP 인덱스

- OLAP(Online Analytical Processing), 온라인 분석 처리
- 대량의 데이터를 한꺼번에 입력한 뒤 주로 분석이나 통계 정보를 출력할 때 많이 사용
- 데이터 값의 종류가 적고 동일한 데이터가 많을 경우 사용
- 데이터가 어디에 있다는 지도정보(MAP)를 bit로 표기, 데이터가 존재하는 곳은 1, 없는 곳은 0으로 표기

| 구분                                   | 설명                                                     |
| -------------------------------------- | -------------------------------------------------------- |
| 고유 인덱스(Unique Index)              | 유일한 값을 갖는 컬럼에 대해서만 인덱스를 설정할 수 있음 |
| 비고유 인덱스(NonUnique Index)         | 중복된 데이터를 갖는 컬럼에 대해서 생성하는 인덱스       |
| 단일 인덱스(Single Index)              | 한 개의 컬럼으로 구성된 인덱스                           |
| 결합 인덱스(Composite Index)           | 두 개 이상의 컬럼으로 구성된 인덱스                      |
| 함수 기반 인덱스(Function Based Index) | 수식이나 함수를 적용하여 만든 인덱스                     |



## Index Rebulid

 인덱스 파일은 생성 후에 CUD와 같은 작업들을 반복하다보면 성능이 저하된다. 인덱스는 트리 구조를 가지기 때문에 CUD 작업이 많이 일어나다보면 트리의 한 쪽이 무거워져 전체적으로 트리의 깊이가 깊어진다. 이러한 현상으로 인덱스를 이용한 조회 성능이 떨어지므로 주기적으로 리빌딩 작업을 해주어야 한다.



```SQL
ALTER INDEX 인덱스명 REBUILD;
```



```SQL
SELECT 'ALTER INDEX '||INDEX_NAME||' REBUILE; ' FROM USER_INDEXES;
```

- 일일이 트리의 깊이가 깊은 인덱스를 찾아서 리빌딩하기 번거롭다면 USER_INDEXES에 있는 인덱스를 조회하여 한번에 리빌딩하면 된다.



## 주의 사항

 개발 진행시에 보통 개발 서버와 운영 서버를 나누어서 관리한다. 대부분 개발 서버에서 개발할 때는 비교적 적은 양의 데이터를 가지고 로직 검사를 하며 검사를 통과한 코드들이 운영서버에 업데이터 된다. 운영서버에 업데이트 후에 통과했던 로직들이 많은 양의 데이터를 처리하다보면 성능 문제가 많이 발생한다. 문제의 주요 원인은 DB에 있다. DB 관리자는 성능 문제가 발생하면 먼저 생각하는 해결책이 인덱스 추가, 생성이다. 인덱스를 생성하여 해결했다하고 그 다음 문제가 생겨 또 인덱스를 만들었다고 하면 인덱스가 쌓여 전체적인 DB 성능 부하를 초래한다. 그렇기 때문에 인덱스를 무작정 생성하기 보다는 SQL문을 효율적으로 짜는 방향으로 나가는 것이 좋다.



### INSERT 작업시 주의사항

- index split 현상: 인덱스의 블록이 1개에서 2개로 나누어지는 현상
  - 인덱스는 데이터가 순서대로 정렬되어 저장되는데, 기존 블럭에 여유 공간이 없는 상황에 기존 블록에 새로운 데이터가 입력되는 경우 Oracle은 기존 블록의 내용 중 일부를 새 블록에다가 기록한 다음 기존 블록에 빈 공간을 만들어 새 데이터를 추가한다.
  - 따라서 성능 측면에서 좋지 않다.
    - index split은 새로운 블록을 할당받고 key를 옮기는 복잡한 작업을 수행
    - index split 작업 동안에 해당 블록의 키 값이 변경되면 안되므로 DML 블로킹



### DELETE 작업시 주의사항

 테이블에서 데이터 삭제시 해당 데이터가 삭제되고 그 공간을 사용할 수 있다. 하지만 index에서 데이터를 delete 할 경우 데이터를 지워지지 않고 사용하지 않는다는 의미 표시를 해둔다. 예를 들면 테이블에 10만개의 데이터 중 5만개를 지워도 index에는 10만개 데이터가 존재한다.



### UPDATE 작업시 주의사항

 위에 UPDATE라고 썼지만 index에는 UPDATE 작업이 존재하지 않고 수정하려면 기존의 데이터를 DELETE한 다음 새로운 값을 INSERT하는 2번의 과정을 거쳐서 수행한다. 그렇기 때문에 더 큰 부하를 데이터베이스에게 주게 된다.



## 인덱스를 사용해야 하는 경우

- 테이블의 행 개수가 많은 경우
- 인덱스를 적용한 컬럼이 WHRER 절에서 많이 사용되는 경우
- 해당 컬럼이 NULL을 포함하는 경우
- 원본 테이블의 CUD 작업이 빈번한 경우 사용 X



## 인덱스의 장단점

### 장점

- 검색 속도가 빨라진다.
- 시스템 부하를 줄여서 시스템 전체 성능을 향상시킨다.

### 단점

- 인덱스를 생성하는 데 시간과 추가적인 공간이 필요하다.
- INSERT, UPDATE, DELETE 작업이 빈번하게 일어날 경우 성능이 저하된다.





# 인덱스 예제

## 인덱스 사용하지 않는 경우

1) 사원 테이블을 복사해서 새로운 테이블을 생성

```SQL
CREATE TABLE EMP08 AS SELECT * FROM EMP;
```

:question: 테이블 복사하기

```SQL
CREATE TABLE 새로만들테이블명 AS SELECT * FROM 복사하고자하는테이블명;
```



2) EMP와 EMP08 테이블에 인덱스가 설정되어 있는지 확인

```SQL
COL TABLE_NAME FORMAT A15;
COL COLUMN_NAME FORMAT A15;
SELECT TABLE_NAME, INDEX_NAME, COLUMN_NAME FROM USER_IND_COLUMNS WHERE TABLE_NAME IN('EMP', 'EMP08');
```

:question: COLUMN FORMAT: 컬럼의 출력 형식을 변경할 때 사용

```SQL
COL 컬럼명 FORMAT 출력형식;
COL 컬럼명 FORMAT A숫자; -- 문자 형식 길이 설정, A 뒤에 숫자가 컬럼의 길이를 뜻함
COL 컬럼명 FORMAT 9,999,999 -- 숫자 형식 길이 설정, 3자리마다 , 표시해줌
COL 컬럼명 FORMAT 0000000 -- 숫자 형식 길이 설정, 자리수 선정, 데이터 길이가 설정한 값보다 작을 때 빈자리에 0을 채워준다.
```

- 위 명령어는 SQLPLUS를 이용해야 사용가능한 것 같다. 지금 실행하면 invalid SQL statement 뜬다. 지금은 무시하고 진행하겠음



- 결과

| TABLE_NAME | INDEX_NAME  | COLUMN_NAME |
| ---------- | ----------- | ----------- |
| EMP        | SYS_C008244 | EMPNO       |

- EMP 테이블에는 인덱스가 존재하지만 EMP08에는 존재하지 않는다. 그 이유는 테이블 복사시에 제약조건은 복사되지 않기 때문이다.



3) 서브 쿼리문으로 INSERT문을 여러 번 반복한다.

```SQL
INSERT INTO EMP08 SELECT * FROM EMP08;
--917504 row(s) inserted.
```



4) 검색용으로 사용할 행을 새롭게 하나 추가한다.

```SQL
INSERT INTO EMP08 (EMPNO, ENAME) VALUES(1111,'MJS');
```



5)  시간을 체크하기 위해 다음과 같은 명령을 입력

```mssql
SET TIMING ON
```

- 실행 안됨, SQLPLUS 관련 문제인 거 같은데 안되서 PASS



6) 사원 이름이 'MJS'인 행을 검색

```SQL
SELECT DISTINCT EMPNO, ENAME FROM EMP08 WHERE ENAME='MJS';
```

| EMPNO | ENAME |
| ----- | ----- |
| 1111  | MJS   |

1 rows returned in 0.27 seconds



## 인덱스를 사용하여 조회하는 경우

1) 테이블 EMP08의 컬럼 중에서 ENAME에 대해서 인덱스를 생성한다.

```SQL
CREATE INDEX IDX_EMP08_ENAME ON EMP08(ENAME);
```



2) 인덱스 생성 유무를 확인한다.

```SQL
SELECT TABLE_NAME, INDEX_NAME FROM USER_INDEXES WHERE TABLE_NAME IN('EMP08');
```

| TABLE_NAME | INDEX_NAME      |
| ---------- | --------------- |
| EMP08      | IDX_EMP08_ENAME |



```SQL
SELECT TABLE_NAME, INDEX_NAME FROM USER_IND_COLUMNS WHERE TABLE_NAME IN('EMP08');
```

| TABLE_NAME | INDEX_NAME      |
| ---------- | --------------- |
| EMP08      | IDX_EMP08_ENAME |



3) 사원 이름이 'MJS'인 행을 검색한다.

```SQL
SELECT DISTINCT EMPNO, ENAME FROM EMP08 WHERE ENAME='MJS';
```

| EMPNO | ENAME |
| ----- | ----- |
| 1111  | MJS   |

1 rows returned in 0.00 seconds



- 위에서는 0.27, 인덱스를 사용하니 0.00 속도가 짧아졌음을 알 수 있다.



### 사용한 인덱스 제거

```SQL
DROP INDEX IDX_EMP08_ENAME;
```

- 인덱스가 제거되었는지 확인

```SQL
SELECT TABLE_NAME, INDEX_NAME FROM USER_INDEXES WHERE TABLE_NAME IN ('EMP08');
```

no data found



# 인덱스 사용여부 판단 및 재생성하기

 인덱스를 사용하면 검색 시간을 줄인다고 하여 무조건 인덱스를 사용한다고 해서 검색 속도가 빨라지는 것은 아니다. 계획성없이 너무 많은 인덱스를 지정하면, 오히려 성능을 저하시킬 수 있다.

| 인덱스를 사용해야 하는 경우                                  | 인덱스를 사용하지 말아야 하는 경우                    |
| ------------------------------------------------------------ | ----------------------------------------------------- |
| 테이블에 행의 수가 많을 때                                   | 테이블에 행의 수가 적을 때                            |
| WHERE 문에 해당 컬럼이 많이 사용될 때                        | WHERE 문에 해당 컬럼이 자주 사용되지 않을 때          |
| 검색 결과가 전체 데이터의 2~4% 정도일 때                     | 검색 결과가 전체 데이터의 10~15% 정도일 때            |
| JOIN에 자주 사용되는 컬럼이나 NULL을 포함하는 컬럼이 많은 경우 | 테이블에 DML 작업이 많은 경우(INSERT, UPDATE, DELETE) |

- 인덱스가 생성된 후에 새로운 행이 추가, 삭제, 변경되는 경우
  - 본 테이블에서 추가, 삭제, 갱신이 일어날 때, 해당 테이블에 걸린 인덱스의 내용도 함께 수정되어야 한다.
  - 인덱스가 없을 때보다 DML 작업이 무거워진다.
  - 인덱스는 가끔 한번씩 재생성을 해주어야만 빠른 효율을 누릴 수 있다.



## Invisible Index

- 인덱스를 실제 삭제하기 전에 `사용 안 함` 상태로 만들어서 테스트해 볼 수 있는 기능을 제공하는 인덱스
- 인덱스가 많은 경우 DML 문장에 나쁜 영향을 주기 때문에 사용하지 않는 인덱스는 삭제해주어야 한다.
- 문제는 해당 인덱스를 삭제하려고 했을 때, 정말 사용하는지 사용하지 않는 것인지를 정확하게 알아야 한다.
- 모니터링 기간이 잘못 되었다든지 해서 인덱스를 삭제했는데, 나중에 문제가 발생할 수 있기 때문에 사용한다.



```SQL
CREATE INDEX IDX_EMP01_ENAME ON EMP01(ENAME);
SELECT TABLE_NAME, INDEX_NAME, VISIBILITY FROM USER_INDEXES WHERE TABLE_NAME = 'EMP01';
ALTER INDEX IDX_EMP01_ENAME INVISIBLE;
SELECT TABLE_NAME, INDEX_NAME, VISIBILITY FROM USER_INDEXES WHERE TABLE_NAME = 'EMP01';
```

- INVISIBLE 설정해도 지워진 것은 아니므로 DML 작업시 인덱스 내용은 계속 반영, 따라서 INVISIBLE로 설정한 후 점건하여 다른 SQL 문장에 영향을 주는 것이 없는 지 확인되면 해당 인덱스를 지우면 된다.



# References

- https://coding-factory.tistory.com/419
- https://rongscodinghistory.tistory.com/113
- http://blog.naver.com/sunbeatz/140104507086
- https://wikidocs.net/4180
- https://wikidocs.net/4181
- https://wikidocs.net/4182
- https://wikidocs.net/4183