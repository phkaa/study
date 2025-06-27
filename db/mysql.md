### MySQL Troubleshooting
- API 혹은 어플레키에션이 운영 중일때 쿼리 로그를 확인하는 방법?
  - Log 에는 General log(모든 Query log 수집), Slow log(SQL 질의 요청을 했는데 응답이 오래 걸리는 log 를 수집), Error log 가 있는데 그 중에서 General log 수집 방법을 사용 한다.
  - 설정 방법
    1. MySQL 에 접속을 한다.
    2. 아래의 쿼리를 실행 후 general_log 값이 ON 이 되어 있는지 확인한다.

    ```
    query> SHOW variables LIKE 'general%';
    ```

    3. general_log 가 OFF 라면 아래의 쿼리를 실행 후 다시 2번 쿼리를 실행하여 general_log 값이 ON 이 되었는지 확인한다.

    ```
    query> SET GLOBAL general_log = 'on';
    ```

    4. 정상적으로 실행되었다면 이제 로그를 테이블로 보기 위해 LOG_OUTPUT 을 table 로 설정한다.

    ```
    query> SET GLOBAL LOG_OUTPUT = 'table';
    ```

    5. 설정 후 정상적으로 LOG_OUTPUT 값이 table 로 되어있는지 확인한다.

    ```
    query> SHOW variables LIKE 'log_output%';
    ```

    6. 설정이 완료되면 아래의 쿼리를 실행하면 MySQL 서버에 쌓이는 로그를 확인 할 수 있다.

    ```
    query> SELECT * FROM mysql.general_log;
    ```

    7. 확인이 끝났으면 general_log 값을 off 로 변경한다. off 로 변경하는 이유는 많은 쿼리가 MySQL 서버로 들어오게 되면 성능상의 이슈가 생길 수 있기 때문이다.

    ```
    query> SET GLOBAL general_log = 'off';
    ```
- 실행계획
  - 성능 진단의 가장 첫걸음은 실행한 SQL 이 DB에서 어떻게처 처리 되는지 파악하는 것이다.
  - 실행계획이란 DB가 데이터를 찾아가는 일련의 과정을 사람이 알아보기 쉽게 DB 결과 셋으로 보여주는 것이다.
  - 이를 활용하여 기존의 쿼리를 튜닝 할 수 있고 성능 분석, 인덱스 전략 수집 등과 같이 최적화를 진행 할 수 있다.
  - 사용 방법

  ```
  -- 실행할 쿼리 앞에 EXPLAIN EXTENDED 를 붙여서 사용한다.
  query> EXPLAIN EXTENDED SELECT * FROM CLASS_ROOM AS c LEFT JOIN STUDENT AS s	ON c.`key` = s.class_key;

  -- 위의 쿼리를 실행 후 아래 쿼리를 실행하면 쿼리 옵티마이저에 의해 최적화 된 쿼리를 확인 할 수 있다.
  query> SHOW WARNINGS;
  ```

  - EXPLAIN EXTENDED 결과에서 각 항목의 의미
    - id: SELECT 아이디로 SELECT를 구분하는 번호
    - table: 참조하는 테이블
    - select_type: SELECT에 대한 타입
    - type: 조인 혹은 조회 타입
    - possible_keys: 데이터를 조회할 때 DB에서 사용할 수 있는 인덱스 리스트
    - key: 실제로 사용 할 인덱스
    - key_len: 실제로 사용 할 인덱스의 길이
    - ref: KEY 안의 인덱스와 비교하는 컬럼(상수)
    - rows: 쿼리 실행 시 조사하는 행 수립
    - extra: 추가 정보
  - [EXPLAIN EXTENDED 의 자세한 목록 참조 사이트](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html)

- 테이블 정보조회
  - 아래의 쿼리를 실행하면 테이블에 대한 Field, Type, Null, Key, Default, Extra 를 확인 할 수 있다.

  ```
  query> DESC `Table이름`;
  ```
---
- 쿼리가 느려지는 문제 해결 하는 세가지 방버
  - 쿼리 자체 튜닝
  - 테이블의 튜닝
  - 서버 튜닝