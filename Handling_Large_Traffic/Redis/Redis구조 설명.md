# 아키텍처별 차이점 설명(Cluster, Sentinel)

## Redis Replication

- 고가용성을 제공하기 위한 기본적인 Master-Slave 구조
    - Master: 데이터 읽기/쓰기
    - Slave: Master의 데이터를 복사하여 읽기만 제공
- Master에 장애가 발생하면 쓰기 요청은 모두 실패
    - Slave는 계속 Master에 읽기 요청을 하지만 실패

아키텍처별 차이점 설명(Cluster, Sentinel)

## Redis Sentinel

- Master-Slave 구조에서 Master 장애 시 자동 장애 조치(auto failover)를 수행
    - Sentinel 노드 과반수 이상이 Master 장애를 감지하면 Slave 중 하나를 Master로 승격
- 비교적 설정이 쉽고 기존 Redis 구조에 Sentinel만 추가하면 됨
    - Redis 서버를 지속적으로 모니터링
    - Client가 올바른 노드를 찾을 수 있도록 도움
    - 최소 3개 이상의 노드를 권장
    - Redis와 동일한 노드에 구성해도 되고 별도로 구성해도 됨

## Redis Sentinel 문제점

- 일반적으로 단일 Master 로 구성함
- Master 하나가 모든 쓰기 작업을 처리해야함
    - 대규모 쓰기 작업시 병목 발생
    - Slave에 복제되는 Latency 증가

## Redis Cluster

- 수평 확장을 통해 데이터 분산(shard) 및 고가용성(auto failover)을 제공
    - 최소 3개의 Master Node 필요
    - 각 Master Node마다 최소 하나 이상의 Slave Node 필요
- 설정과 관리가 매우 복잡하고 특정 작업(예: 다중 키 작업)은 일부 제한이 있을 수 있음
    - 데이터는 16,384개 슬롯으로 나뉘어 특정 Master에 할당됨
    - 모든 데이터는 Master 단위로 sharding되고 Slave 단위로 복제됨
    - 즉, 슬롯으로 sharding함

## Redis Cluster 문제점

- 설정과 관리가 매우 복잡함
- 특정 작업은 일부 제한이 있을 수 있음
    - 다중 키 작업(MGET, MSET)
- 새로운 노드를 추가하거나 제거할 때 슬롯 리밸런싱이 필요
- 네트워크 파티션 발생 시 일부 노드끼리 연결되지 않을 경우 장애 발생
- 인메모리 DB 특성상 충분한 메모리 확보가 필

# 내부 구조 설명(SingleThread)

## Redis Single Thread 방식

- Redis는 단일 스레드(Single Thread)에서 모든 클라이언트 요청을 처리
- Redis 서버 자체는 CPU의 코어 수와 관계없이 하나의 CPU 코어만 사용

## Single Thread 방식의 장단점

### 장점

- 모든 명령어가 순차적으로 처리되어 동기화 문제 처리를 고려할 필요 없음(일관성 유지)
- context switching이 필요 없기 때문에 명령어가 빠르게 처리 가능
- 많은 클라이언트 요청을 동시에 받을 수 있음(비동기 I/O)

### 단점

- scale out을 위해서는 Redis 서버를 여러 개 사용해야 함

## Redis Flow

- 클라이언트 요청(네트워크 I/O)
- Task Queue: 클라이언트 요청을 저장
- Event Loop: Task Queue에서 요청(작업)을 꺼냄
- Event Dispatcher: 작업을 적절한 Event Processor에게 전달
- Event Processor: 실제 작업을 처리하는 프로세서
- 결과 반환

### 그럼 CPU 코어 1개인 서버를 쓰는 게 가장 효율적인가?

- CPU가 여러 개라면 Redis 인스턴스를 여러 개 띄워서 코어를 모두 활용할 수 있음
    - CPU 바인딩을 통해서 각 인스턴스가 특정 코어를 바라보게 사용 가능
- Redis가 실행되는 OS도 CPU를 사용하기 때문에 CPU 코어가 최소 2개 이상으로 구성하는 것을 추천

# Redis에서 제공되는 다양한 기능(명령어)

## 자주 사용하는 Redis 명령어