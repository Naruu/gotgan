# 실무로 배우는 빅데이터 기술

## 01. 빅데이터 이해하기

### 6V

- 크기(Volume)
- 다양성(Varity): 정형(DB) + 비정형(SNS, 동영상, 사진, 음성, 텍스트 등)
- 속도(Velocity): 실시간 생산, 빠른 속도로 처리/분석
- 진실성(Veracity): 데이터 품질과 신뢰성 확보
- 시각화(Visualization)
- 가치(Value)

### 빅데이터 프로젝트의 유형

- 플랫폼 구축형 프로젝트
- 빅데이터 분석 프로젝트
- 빅데이터 운영 프로젝트

### 기술

- 수집
  - 여러 소스, 다양한 인터페이스로부터 정형/비정형형 데이터 수집
  - 크롤링 등
  - CEP(Complex Event Processing), ESP(Event Stream Processing)
  - Flume, Fluented, Scribe, Logstash, Chukswa, Nifi, Embulk
- 적재
  - 분산 스토리지에 영구/임시 보관. 전처리를 포함하기도 함
  - 유형
    - 대용량 파일의 영구 저장: HDFS
    - 메시징 데이터의 영구 저장: NoSQL(HBase, MongoDB, Casandra 등)
    - 메시징 데이터의 일부 저장: 인메모리 캐시(Redis, Memcached, Infinispan)
    - 메시징 데이터 버퍼링 처리: Message Oriented Middleware(Kafka, RabbitMQ, ActiveMQ)
- 처리,탐색
  - 데이터 정형화 및 정규화
  - 탐색적 분석: SQL on Hadoop, Ad-hoc query
- 분석, 응용
  - Impala, Zeppelin, Mahout, R, Tensorflow, Sqoop, ...

### 보안

- 접근 제어 보안

  - apache knox

    - hadoop ecosystem에 대한 접근 제어. 중간 게이트웨이 역할
    - LDAP(Lightweight Directory Access Protocol), KDC(Key Distribution Center)로 접근 인증

  - apahce centry, ranger
    - 하둡 파일시스템에 상세한 접근 제어
    - 하둡 데이터에 접근하려는 클라이언트는 반드시 센트리 에이전트를 설치하여, 에이전트를 통해 센트리 서버와 통신하여 권한을 획득한다.
    - centry: cloudera, ranger: hortonworks
  - keberos
    - KDC(Key Distribution Center)
    - AS(Authentication Service, 인증 서버) \
      TGS(Ticket Granting Service, 티켓 발행 서버)
    - 클라이언트는 AS에서 최초 인증 -> TGS로부터 파일 시스템 접근 허용 티켓을 발행받음 \
      이후부터는 티켓으로 인증 없이 접근

## 03. 수집

- Flume

  - 통신 프로토콜, 메시지 포맷, 발생 주기, 데이터 크기 등에 대한 고민을 해결하는 기능 제공
  - component
    - agent: source -> (interceptor) -> channel -> sink 순으로 구성도니 작업 단위.
    - source: 여러 소스의 데이터를 channel로 전달
    - sink: 수집한 데이터를 channel로부터 전달받아 최종 목적지에 저자아기 위한 기능. HDFS, Hive, Logger, Avro, ElasticSearch, Thrift
    - channel: source와 sink를 연결
    - interceptor: source와 channel 사이에서 데이터 필터링 및 가공하는 component. timestamp, host, regex filtering 등 제공

- Kafka
  - 비동기 처리를 위한 message queue
  - provider가 데이터를 빠르게 보내는 경우, consumer에 부하가 갈 수 있음. \
    속도 완충, 임시 저장소 역할
  - component
    - broker: 서비스 인스턴스. 다수의 broker를 클러스터로 구성하고 topic이 생성되는 물리적 서버
    - topic: 데이터 발생/소비 처리를 위한 저장소

## 04. 적재 - 대용량 로그 파일

- hadoop

  - component(v2)
    - namenode: metadata. active/standby
    - datanode: store data
    - resource manager: 자원 관리. find the best data node
    - node manager: datanode마다 존재 conainer를 실행시키고 라이프사이클 관리
    - container: datanode의 사용 가능 리소스를 container 단위로 할당하여 구성
    - application master: nodemanager에게 애플리케이션이 실행될 container를 요청하고, 실제 실행 및 관리
    - journalnode: editlog를 각 노드에 복제 관리, active - editslog쓰기만, standby - 읽기만

- zookeeper
  - coordinator system: 분산락, 순서 제어, 네임서비스
  - 3대 이상의 홀수 서버
  - 1대만 리더, 나머지는 팔로워 서버
  - 팔로워 서버의 Znode정보는 리더 서버로 전달되고, 리더 서버가 전체 서버에 broadcast
  - component
    - znode: 주커피 서버에 생성되는 파일시스템의 디렉터리 개념.
    - ensemble: 3대 이상의 주키버 서버를 하나의 클러스터로 구성한 HA 아키텍처

## 05. 적재 - 실시간 로그 파일

### HBase

- hadoop based column-oriented NoSQL
- schema 변경이 자유로움
- component
  - HTable: column-based table. 공통점이 있는 칼룸 그룹인 패밀리와 row를 식별하기 위한 row key로 구성
  - HMaster: HRegion 서버 관리, HRegion 서버의 메터 정보 관리
  - HRegion: HTable의 크기에 따라 자동으로 수평 분할 발생, 분할된 블록을 HRegion 단위로 지정
  - HRegionServer: 분산 노드별 HRegionServer가 구성되며, 하나의 HRegionServer에는 다수의 HRegion 서버가 생성되어 관리함.
  - Store: 하나의 Store에는 칼럼 패밀리가 저장,관리. Memstore, HFile로 구성
    - MemStore: Store의 데이터를 인메모리에 저장/관리하는 데이터 캐시 영역
    - HFile: Store의 데이터를 스토리지 저장/관리하는 영구 저장 영역
- 쓰기
  - 클라이언트는 zookeeper와 통신하여 HTable의 기본 정보와 HRegion의 위치 정보를 알아낸다.
  - 클라이언트가 직접 HRegionServer로 연결되어 MemStore에 쓴다
  - 특정 시점이 되면 MemStore -> HFile로 플러시 된다.(Major/Minor Compaction)
- 읽기
  - zookeeper를 통해 rowkey에 해당하는 데이터의 위치 정보를 알아온다.
  - 클라이언트가 직접 HRegionServer로 연결되어 MemStore에서 읽는다. \
    MemStore에 없으면 HFile로부터 가져온다.f
  - HBase와 HDFS 사이의 데이터 스트림은 열려있으므로 레이턴시가 없다.

### Redis

- 분산 캐시, in-memory data grid
- key,value data + 다양한 형식의 데이터 지원
- snapshot 기능: 인메모리 데이터를 영구 저장
- AOF(Append Only File): 데이터 유실에 정합성 보장
- sharding, replication 지원

### Storm

- speed data 처리 \
  (speed data: 클릭, 터치, 위치, IOT 등의 이벤트. 작지만 동시다발적)
- component
  - spout: 데이터를 받고 -> 가공 -> 튜플 생성 -> bolt에 전송
  - bolt: 튜플을 받아 -> 분산 작업. filtering, aggregation, join 등
  - topology: spout-bolt의 데이터 처리 흐름
  - nimbus: topology를 supervisor에 배포 및 작업 할당. supervisor 모니터링 + failover 처리
  - supervisor: topology를 worker에 할당, 관리
  - worker: supervisor 상의 자바 프로세스. spout, bolt 실행
  - executor: supervisor 상의 자바 프로세스
  - tasker: spout, bolt 객체가 할당

### Esper

- CEP(Complex Event Processing): 복잡한 데이터 간의 관계를 복합적으로 찾는 것.
- component
  - event: 실시간 스트림으로 발생하는 데이터의 흐름 또는 패턴 정의
  - EPL: 유사 SQL 기반의 이벤트 데이터 처리 스크립트 언어
  - input adapter
  - output adpater
  - window

## 06. 탐색

### Hive

- hive에서 작성한 QL(Query Language)은 map reduce로 변환된다
- SQL parser가 hive QL을 mapreduce program으로 변환하고 hadoop cluster에 전송된다.
- component
  - metastore: hive의 table schema 저장/관리
  - cli
  - jdbc/odbc drviber
  - query engine
- 특징
  - 대화형 온라인 처리에 부적합: 대용량 배치 처리에 적합.
  - 부분 수정 불가
  - transaction 관리 기능 없음

### Spark

- component
  - spark driver: task 구성, 할당, 계획
  - spark executor: task 실행, 관리
  - spark rdd, sql, streaming, mlib, graphX

### Oozie

- DAG로 워크플로우를 정의하여 관리
- component
  - oozie workflow
  - oozie client
  - oozie server
  - control node: 워크플로우의 흐름을 제어하기 위한 start, end, decision 노드
  - action node: task 수행 노드. hive, pig, mapreduce 등의 action
  - coordinator

### Hue

- hadoop ecosystem를 web ui로 통합 제공

## 07. 분석

### 분석의 유형

- 기술 분석: 데이터 파악을 위한 선택, 집계, 요약 등
- 탐색 분석: 상관관계나 연관관계 파악
- 추론 분석: 전통적인 통계 분석 기법. 가설을 세우고 샘플링으로 입증
- 인과 분석: 문제해결을 위한 원인과 결과 변수 도출, 변수의 영향도 분석
- 예측 분석

### impala

- in-memory based sql on hadoop

### Zeppelin

- spark based, analysis, and visualization tool
- web ui's notebook에서 spark 작업 작성
- hadoop cluster에 작업 요청
- web ui에서 시각화

### Mahout

- machine learning based datamining tool
- component
  - recommendation
  - classification
  - clustering
  - supervised learning
  - unsupervised learning

### Sqpoop

- data import/export tool between hdfs and RDBMS
