# 1장. 사용자 수에 따른 규모 확장성

# 단일 서버

모든 컴포넌트가 단 한대의 서버에서 실행되는 간단한 시스템(웹 앱, 데이터베이스, 캐시 등이 전부 서버 한 대에서 실행)

- 웹 애플리케이션
    - 비즈니스 로직, 데이터 저장 등을 처리하기 위해서는 서버 구현용 언어(자바, 파이썬 등)를 사용하고, 프레젠테이션용으로는 클라이언트 구현용 언어(HTML, 자바스크립트 등)를 사용한다.
- 모바일 앱
    - 모바일 앱과 웹 서버간 통신을 위해서 HTTP 프로토콜을 이용한다.
    - 반환될 응답 데이터 포맷으로 JSON을 널리 사용한다.

# 데이터베이스

사용자가 늘면 서버 하나로는 충분하지 않아서 여러 서버를 두어야 하는데, 웹/모바일 트래픽 처리 서버(웹 계층)와 데이터베이스 서버(데이터 계층)를 분리하여 각각 독립적으로 확장해갈 수 있다.

## 어떤 데이터 베이스를 사용할 것인가?

- 관계형 데이터베이스(RDBMS)
    - 자료를 테이블, 열, 컬럼으로 표현
    - SQL을 사용해서 여러 테이블에 있는 데이터를 그 관계에 따라 조인(join)하여 합칠 수 있다.
    - MySQL, 오라클 데이터베이스, PostgreSQL..
- 비-관계형 데이터베이스(NoSQL)
    - Amazon DynamoDB, CouchDB, Cassandra, HBase..
    - 종류
        - key-value store
        - graph store
        - column store
        - document store
    - 적합한 케이스
        - 아주 낮은 응답 지연시간(leatency)가 요구됨
        - 다루는 데이터가 비정형(unstructured)이라 관계형 데이터가 아님
        - 데이터(JSON, YAML, XML 등)를 직렬화하거나 역직렬화 할 수 있기만 하면 된다.
        - 아주 많은 양의 데이터를 저장할 필요가 있다.

# 수직적 규모 확장 vs 수평적 규모 확장

- 수직적 규모 확장(Scale Up)
    - 서버에 고사양 자원(더 좋은 CPU, 더 많은 RAM 등)을 추가하는 행위
    - 서버로 유입되는 트래픽의 양의 적을 때 좋은 선택
    - 장점
        - 단순함
    - 단점
        - 한 대의 서버에 늘릴 수 있는 CPU나 메모리는 한계가 존재함
        - 장애에 대한 자동 복구(failover) 방안이나 다중화(re-dundancy) 방안을 제시하지 않아 장애 발생시 중단된다.
- 수평적 규모 확장(Scale Out)
    - 더 많은 서버를 추가하여 성능을 개선하는 행위
    - 대규모 애플리케이션에 적합하다.

## 로드밸런서

- 부하 분산 집합에 속한 웹 서버들에게 트래픽 부하를 고르게 분산하는 역할을 한다.
- 이미지
- 사용자는 로드밸런서의 공개 IP 주소(Public IP address)로 접속한다.
- 서버간 통신에는 사설 IP 주소(private IP address)가 사용된다.
    - 사설 IP 주소는 같은 네트워크에 속한 서버 사이의 통신에만 쓰일 수 있는 IP 주소
- 부하 분산 집합에 또 하나의 웹 서버를 추가하고 나면 장애를 자동 복구하지 못하는 문제가 해소되면, 웹 계층의 가용성은 향상된다.
    - 서버 1이 다운되면 모든 트래픽은 서버 2로 전송
    - 트래픽이 가파르게 증가하여 두 개의 서버로 감당되지 않을 경우, 더 많은 서버를 추가하고 로드밸런서가 자동적으로 트래픽을 분산시킨다.

## 데이터베이스 다중화

- 많은 데이터베이스 관리 시스템은 다중화를 지원하는데, 서버 사이에 주(master)-부(slave) 관계를 설정하고 데이터 원본은 주 서버에, 사본은 부 서버에 저장하는 방식이다.
- 쓰기 연산(write operation)은 마스터에서만 지원한다.
    - 부 데이터베이스는 주 데이터베이스로부터 그 사본을 전달받으며, 읽기 연산(read operation)만을 지원한다.
    - 데이터 베이스를 변경하는 명령어(insert, update, delete..)들은 주 데이터베이스로만 전달되어야 한다.
    - 대부분의 애플리케이션은 읽기 연산의 비중이 쓰기 연산보다 훨씬 높다.
    - 통상 부 데이터베이스의 수가 주 데이터베이스의 수보다 많다.
    
- 장점
    - 더 나은 성능
        - 주-부 다중화 모델에서 모든 데이터 변경은 주 데이터베이스 서버로만 전달되고, 읽기는 부 데이터베이스 서버로 분산 되어 병렬로 처리될 수 있는 질의의 수가 늘어나 성능이 좋아진다.
    - 안정성
        - 데이터를 지역적으로 떨어진 여러 장소에 다중화시킬 경우 자연재해등의 이유로 서버가운데 일부가 파괴되어도 데이터 보존이 가능하다.
    - 가용성
        - 데이터를 여러 지역에 복제해둠으로써, 하나의 데이터베이스 서버에 장애가 발생하더라도 다른 서버에 있는 데이터를 가져와 계속 서비스할 수 있게 된다.

로드밸런서와 데이터베이스 다중화를 고려한 설계안

# 캐시

값비싼 연산 결과 또는 자주 참조되는 데이터를 메모리 안에 두고 뒤이은 요청이 보다 빨리 처리될 수 있도록 하는 저장소.

애플리케이션의 성능은 데이터베이스를 얼마나 자주 호출하느냐에 크게 좌우되는데, 캐시는 그런 문제를 완화할 수 있다.

## 캐시 계층

데이터가 잠시 보관되는 곳으로 데이터베이스보다 훨씬 빠르다. 

별도의 캐시 계층을 두면 성능개선, 데이터베이스의 부하 감소, 캐시 계층의 규모의 독립적 확장이 가능

### 읽기 주도형 캐시 전략(read-through caching strategy)

요청을 받은 웹서버는 캐시에 응답이 저장되어 있는지를 본 후, 저장되어 있다면 해당 데이터를 사용하고 그렇지 않다면 데이터베이스 질의를 통해 데이터를 찾아 캐시에 저장한 후 클라이언트에 반환한다.

### 캐시 사용시 고려할 점

- 캐시 서버에 저장할 데이터의 종류
    - 데이터 갱신은 자주 일어나지 않지만 참조는 빈번하게 일어나는 경우
    - 캐시 서버는 재시작시 캐시 내의 모든 데이터는 사라지므로, 영속적으로 보관할 데이터는 캐시에 두지 않는 것이 바람직하며 중요한 데이터는 여전히 지속적 저장소(persistent data store)에 두어야 한다.
- 캐시에 보관된 데이터의 적정한 만료 기한 정책
- 캐시와 데이터 저장소 원본간의 일관성 유지
- 캐시 서버의 분리
    - 장애 발생시 단일 장애 지점(Single Point of Failure, SPOF)이 되지 않도록 여러 지역에 걸친 캐시서버의 분리가 필요
- 캐시 메모리의 크기
- 데이터 방출 정책
    - LRU(Least Recently Used) : 마지막으로 사용된 지점이 가장 오래된 데이터를 내보내는 정책
    - LFU(Least Frequently Used) : 사용된 빈도가 가장 낮은 데이터를 내보내는 정책
    - FIFO(First In First Out) : 가장 먼저 캐시에 들어온 데이터를 가장 먼저 내보내는 정책

# 콘텐츠 전송 네트워크(CDN)

정적 콘텐츠를 전송하는 데 쓰이는, 지리적으로 분산된 서버의 네트워크이며 이미지, 비디오, CSS, JavaScript 파일 등을 캐시할 수 있다.

사용자가 웹 사이트에 접속시 사용자에게 가장 가까운 CDN 서버가 정적 콘텐츠를 전달한다.

### CDN 사용 시 고려해야 할 사항

- 비용 : CDN은 보통 제3 사업자(third-party providers)에 의해 운영되어 데이터 전송양에 따라 요금을 지불
- 적절한 만료 시한 설정
- CDN 장애에 대한 대처 방안
    - CDN 자체가 죽었을 경우를 고려하여 웹사이트/애플리케이션이 동작하는 방식 구성
- 콘텐츠 무효화(invalidation) 방법 : 아직 만료되지 않은 콘텐츠를 CDN에서 제거하는 방법

# 무상태(stateless) 웹 계층

웹 계층을 수평적으로 확장하기 위해 웹 계층에서 상태 정보(사용자 세션 데이터와 같은)를 제거해야한다.

무상태 웹 계층은 상태 정보를 관계형 데이터베이스나 NoSQL 같은 지속성 저장소에 보관하고, 필요할 때 가져오도록 구성한다.

- 무상태 아키텍쳐에서는 사용자로부터의 HTTP 요청은 어떤 웹 서버로도 전달될 수 있으며, 웹 서버는 상태 정보가 필요할 경우 공유 저장소(shared storage)로부터 데이터를 가져온다.
- 무상태 아키텍쳐는 단순하고, 안정적이며 규모 확장이 쉽다는 장점이 있다.

# 데이터 센터

- 지리적 라우팅(geoDNS-routing or geo-routing)
    - 장애가 없는 상황에서 사용자는 가장 가까운 데이터 센터로 안내된다.
    - 지리적 라우팅에서의 geoDNS는 사용자의 위치에 따라 도메인 이름을 어떤 IP 주소로 변환할지 결정할 수 있도록 해주는 DNS 서비스이다.
- 이미지
- 데이터 센터 중 하나에 심각한 장애가 발생하면 모든 트래픽은 장애가 없는 데이터 센터로 전송된다.

### 다중 데이터 센터 아키텍처의 구성을 위해 해결해야하는 기술적 난제

- 트래픽 우회
    - 올바른 데이터 센터로 트래픽을 보내는 효과적인 방법을 찾아야 한다.
    - GeoDNS는 사용자에게서 가장 가까운 데이터센터로 트래픽을 보낼 수 있도록 해준다.
- 데이터 동기화
    - 데이터 센터마다 별도의 데이터베이스를 사용하고 있는 상황이라면, 장애가 자동으로 복구되어(failover)도 다른 데이터센터로 우회되었다고 하여도 해당 데이터센터에 찾는 데이터가 없을 수 있다.
    - 데이터를 여러 데이터센터에 걸쳐 다중화한다.
- 테스트와 배포(deployment)
    - 웹 사이트 또는 애플리케이션의 원활한 테스트를 위해서 자동화된 배포 도구를 사용하는 방법도 있다.

# 메시지 큐

- 메시지 큐는 메시지의 무손실(durability, 즉 메시지 큐에 일단 보관된 메시지는 소비자가 꺼낼 때까지 안전히 보관된다는 특성)을 보장하는, 비동기 통신을 지원하는 컴포넌트
- 생산자 또는 발행자(producer/publisher)로 불리는 입력 서비스가 메시지를 만들어 메시지 큐에 발행(publish)하고, 소비자 혹은 구독자(consumer/subsciber)라 불리는 서비스 혹은 서버가 메시지를 받아 그에 맞는 동작을 수행하는 역할을 한다.
- 메시지 큐를 이용하면 서비스 또는 서버간 결합이 느슨해져서, 규모 확장성이 보장되어야 하는 안정적 애플리케이션을 구성하기 좋다.

# 로그, 메트릭 그리고 자동화

소규모 웹 사이트를 만들때는 로그, 메트릭, 자동화 구성이 필수가 아니지만 규모가 커지게 되면 필수 구성요소가 된다.

- 로그
    - 시스템의 오류와 문제들을 쉽게 발견하기 위해 에러 로그를 모니터링 하는 것은 중요하다.
- 메트릭
    - 호스트 단위 메트릭 : CPU, 메모리, 디스크 I/O에 관한 메트릭
    - 종합(aggregated) 메트릭 : 데이터베이스 계층의 성능, 캐시 계층의 성능
    - 핵심 비즈니스 메트릭 : 일별 능동 사용(daily active user), 수익(revenue), 재방문(retention)
- 자동화

# 데이터베이스의 규모 확장

### 수직적 규모 확장(Scale Up)

고성능의 자원(CPU, RAM, 디스크 등)을 증설하는 방법

- 단점
    - 데이터베이스 서버 하드웨어는 한계가 존재한다.
    - SPOF(Single Point of Failure)로 인한 위험성
    - 높은 비용

### 수평적 확장

- 샤딩(sharding)
    - 대규모 데이터베이스를 샤드(shard)라고 부르는 작은 단위로 분할하는 기술
    - 모든 샤드는 같은 스키마를 쓰지만 샤드에 보관되는 데이터 사이에는 중복이 없다.
    - 샤딩 전략 구현시 가장 중요한 점은 샤딩 키(sharding key)를 어떻게 정하느냐 하는 것
        - 샤딩 키 : 데이터가 어떻게 분산될지 정하는 하나 이상의 칼럼
    - 샤딩 도입시 시스템이 풀어야 할 문제
        - 데이터의 재 샤딩 : 데이터의 재샤딩 해야하는 필요가 있을 때, 샤드 키를 계산하는 함수를 변경하고 데이터를 재배치 하여하 한다.
        - 유명인사 문제 : 핫스팟 키(hotspot key) 문제라고도 부르는데, 특정 샤드가 질의에 집중되어 서버가 과부하에 걸리는 문제
        - 조인과 비정규화 문제 : 여러 샤드에 걸친 데이터를 조인하기가 힘들어진다. 이를 해결하기 위해 데이터베이스를 비정규화 하여 하나의 테이블에서 질의가 수행될 수 있도록 한다.

# 시스템 규모 확장을 위한 기법 정리

- 웹 계층은 무상태 계층으로 구성
- 모든 계층에 다중화 도입
- 가능한 한 많은 데이터를 캐시한다.
- 여러 데이터 센터를 지원한다.
- 정적 콘텐츠는 CDN을 이용해 서비스한다.
- 데이터 계층은 샤딩을 통해 그 규모를 확장한다.
- 각 계층은 독립적 서비스로 분할한다.
- 시스템을 지속적으로 모니터링 하고, 자동화 도구들을 활용한다.