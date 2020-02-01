# 3장: MySQL Architecture

## MySQL전체구조
- MySQL Server = MySQL Engine + Storage Engine
- MySQL Engine = Connection Handler + SQL Interface + SQL Parser + SQL Optimizer + Cache & Buffer
    - MySQL Engine은 클라이어트로부터의 좁속 및 쿼리 요청을 처리하는 커넥션 핸들러와 SQL Parser 및 전처리기, 그리고 쿼리의 최적화된 샐행을 위한 옵티마이저가 중심을 이룬다. 성능 향상을 위해 MyISAM 키 캐시나 InnoDB의 버퍼풀과 같은 보조 저장소가 포함돼있다.
    - MySqL Engine은 SQL 문장을 분석하거나 최적화
    - MySql Engine은 Brain이다.
- Storage Engine = InnoDB, MyISAM, Memory, Federated
    - 디스크 스토리지에 정장하고 디스크 스토리지로부터 데이터를 읽어오는 부분은 Storage Engine이 전담한다
    - MySQL 서버에서 MySQL Enigne은 하나지만 스토리지 엔진은 여러개이다
    ``` MySql
    CREATE TABLE test_table(fd1 INT, fd2 INT) ENGINE=INNODB;
    //test_table에 대한 INSERT, UPDATE, DELETE, SELECT 작업은 모두 INNODB스토리지 엔진이 처리를 담당
    ```
- 핸들러 API
    - MySQL 엔진의 쿼리 실행기에서 데이터를 쓰거나 읽어야 할 때는 각 스토리지 엔진에게 쓰기 또는 읽기를 요청하는데, 이러한 요청을 핸들러(Handler) 요청이라고 하고, 여기서 사용되는 API를 핸들러 API라고 한다. INNODB 스토리지 엔진 또한 이 핸들러 API를 이용해 MYSQL엔진과 데이터를 주고 받는다.

## MySQL 스레딩 구조
- MySQL 서버는 프로세스 가반이 아니라 스레드 기반으로 작동하며. Foregroud Thread, Backgroudn Thread가 있다

### Forground Thread
- 포그라운드 스레드(클라이언트 스레드)는 최소한 MySQL 서버에 접속된 클라이언트의 수만큼 존재하며, 주로 각 클라이언트 사용
자가 요청하는 쿼리 문장을 처리하는 것이 임무다. 
- Thread Cache = Thread Pool
- 스레드 캐시 일정하고 유지 설정: thread_cache_size
- 포그라운드 스레드는 데이터를 MySQL의 데이터 버퍼나 캐시로부터 가져오며, 버퍼나 개시에 없는 경우에는 직접 디스크의 데이터나 
인덱스 파일로부터 데이터를 읽어와서 작업을 처리한다. 

### MyIsam vs InnoDB(Storage Engine, Table를 의미함)
- 전자는 디스크 쓰기 작업까지 포그라운드 스레드가 처리하지만
- 후자는 데이터 버퍼나 캐시까지만 포그라운드이고 나머지 버퍼로부터 디스크까지 기록하는 작업은 백그라운드 스레드가 담다

#### Background Thread
- INNODB Table 또느 INNODB Storage ENGINE에만 해당
- 인서트 버퍼 병합, 로그를 디스크로 기록, INNODB 버퍼풀에 데이터를 디스크에 기록, 데이터를 버퍼로 읽어들이기, 잠금이나 데드락 모니터링, 총괄하는 메인 스레드
- 가장 중요한 쓰레드: Log Thread, Write Thread

### Write Thread
- 버퍼의 데이터를 디스크로 내려쓰는 작업을 처리하는 쓰레드

### SQL 지연쓰기 vs 지연읽기
- 쓰기 작업은 버퍼링 작업할 수 있지만 읽기 작업은 버퍼링이 안된다
- 사용자가 읽기 쿼리를 날렸는데 잠깐만 십분있다가 줄게 하는 꼴
- InnoDB에서는 Insert, Update, Delete쿼리로 데이터가 변경되는 경우 데이터가 디스크의 데이터 파일로 완전히 저장될 때 까지 기다리지 않아도 된다(왜? 버퍼링 되니까...)

## 메모리 할당 구조
- MySQL서버는 글로벌 메모리 영역과 로컬 메모리 영역으로 나뉜다. (JVM의 Heap, Stack과 비슷한 개념인 것 같다)
- MySQL 서버 = 글로벌 메모리 영역(MyISAM의 키캐시, InnoDB의 버퍼 풀, 쿼리 캐시, 테이블 캐시, 로그 버퍼) + 로컬(세션) 메모리 영역(커넥션 버퍼, Result 버퍼, Read 버퍼, Join 버퍼, 랜덤 Read 버퍼, 정렬 버퍼, 바이너리 로그 버퍼)
- 글로벌 메모리 영역의 모든 메모리 공간은 MySQL서버가 시작되면서 무조건 운영체제로부터 할당받는다

### 글로벌 메모리 vs 로컬 메모리
- 차이는 MySQL서버 내에 존재하는 많은 스레드가 공유해서 사용하는 공간인지 아닌지에 따라 구분된다
- 글로벌 메모리 영역: 모든 스레드가 공유
- 세션 메모리 영역: 클라이언트 스레드가 쿼리를 처리하는데 사용하는 메모리 영역이다. 
- 로컬 메모리는 각 클라이언트가 스레드별로 독립적으로 할당되며 절대 공유되어 사용되지 않는다.
- 로컬 메모리 공간은 커넥션이 열려있는 동안 계속 할당된 상태로 남아 있는 공간(커넥션 버퍼, 결과 버퍼)도 있고 그렇지 않고 쿼리를
실행하는 순간에만 할당했다가 다시 해제하는 공간도 있다(커넥션 버퍼, 결과 버퍼)

## 플러그인 스토리지 엔진 모델
- MySQL의 독특한 구조 중 대표적인 것인 바로 플로그인 모델
- 이 세상의 수많은 사용자의 요구조건을 만족시키기 위해 => 스코리지 엔징 커스터 마이징
- *대부분의 작업이 MySQL엔진에서 처리되고, 마지막 데이터 읽기/쓰기 작업만 스토리지 엔진에서 처리된다.*
- MySQL Engine의 역할: 1 SQL Parsing 2 SQL Optimizing 3 SQL 실행
- Stroage Engine의 역할: 데이터 읽기/쓰기
- 디스크 스토리지: 데이터를 담기
- MySQL서버에서는 MySQL엔진은 사람 역할을 하고, 각 스토리지 엔진은 자동차 역할을 하게 되는데, MySQL엔진을 조정하기 위해 핸들러를 사용 (아마 핸들러가 Storage Engine를 갖고 있는 형태 일듯요?)
- 복잡한 Group By나 Order By 등 많은 복잡한 처리는 MySQL 엔진에(쿼리 실행기)에서 실행한다.
- Parsing -> Optimizing -> Executing!

## 쿼리 실행 구조
- Parser: 쿼리 문장을 MySQL이 인식할 수 있는 최소 단위의 어휘나 기호로 분리해 트리 행태의 구조로 나타냄. 기본 문법 발견
- 전처리기: 만들어진 파서 트리를 기반으로 문장에 구조적인 문제점이 있는지 확인. 토큰을 테이블 이름, 칼럼 이름, 내장 함수에 매핑. 있는지 없는지 확인, 접근권한 확인
- Optimizer: 이 책의 핵심! 사용자의 요청으로 들어온 쿼리 문장을 저렴한 비용으로 가장 빠르게 처리할지 결정하는 역할을 담당, 두뇌. 어떻게 하면 더 나은 선택을 할 수 있을까?
- 실행엔진


