# OSPF (Open Shortest Path First)

## 개요
OSPF는 링크 상태(Link State) 라우팅 프로토콜로, 다익스트라 알고리즘을 사용하여 최단 경로를 계산합니다. [[라우팅 기초]]를 기반으로 하며, 대규모 네트워크에서 빠른 컨버전스와 효율적인 라우팅을 제공합니다.

## OSPF 특징

### 장점
- **빠른 컨버전스**: 토폴로지 변화에 즉시 대응
- **무한 루프 방지**: 링크 상태 기반으로 루프 없음
- **확장성**: 대규모 네트워크 지원
- **VLSM 지원**: 가변 길이 서브넷 마스크
- **인증 지원**: 보안 강화 가능
- **멀티패스**: Equal Cost Load Balancing

### 단점
- **복잡성**: 설정 및 관리 복잡
- **리소스 사용**: CPU 및 메모리 사용량 높음
- **수렴 시간**: 초기 LSA 교환 시간 필요

## OSPF 동작 원리

### 3단계 프로세스
```
1. Neighbor Discovery (인접 라우터 발견)
   ↓
2. Database Exchange (링크 상태 데이터베이스 교환)
   ↓  
3. Route Calculation (SPF 알고리즘으로 경로 계산)
```

### LSA (Link State Advertisement)
```
목적: 링크 상태 정보 전파
내용: 
- 라우터 ID
- 연결된 네트워크
- 링크 비용 (Cost)
- 이웃 정보

전파: 플러딩(Flooding) 방식으로 모든 라우터에 전달
```

### LSDB (Link State Database)
```
구성: 모든 LSA의 집합
특징: 
- Area 내 모든 라우터가 동일한 LSDB 유지
- 토폴로지 전체 그림 제공
- SPF 계산의 기초 데이터
```

## OSPF 메트릭

### Cost 계산
```
기본 공식: Cost = 100,000,000 / Bandwidth (bps)

인터페이스별 기본 Cost:
- Serial 64K: 1562
- Serial 128K: 781  
- Serial T1 (1.544M): 64
- Ethernet 10M: 10
- Fast Ethernet 100M: 1
- Gigabit Ethernet: 1
```

### Cost 수동 설정
```cisco
# 인터페이스별 Cost 설정
Router(config)# interface fastethernet 0/0
Router(config-if)# ip ospf cost 50

# 참조 대역폭 변경 (모든 라우터에서 동일하게)
Router(config)# router ospf 1
Router(config-router)# auto-cost reference-bandwidth 1000
```

## OSPF Area 개념

### 계층적 설계
```
목적:
- 링크 상태 정보 범위 제한
- SPF 계산 부하 분산
- 확장성 향상
- 관리 편의성

구조:
Backbone Area (Area 0)
├── Regular Area (Area 1)
├── Regular Area (Area 2)  
└── Regular Area (Area 3)
```

### Area 유형
```
Backbone Area (Area 0):
- 모든 Area의 중심
- 모든 ABR이 연결되어야 함
- Inter-area 트래픽 전달

Regular Area:
- Area 0에 연결된 일반 영역
- Internal LSA만 생성
- 다른 Area 정보는 Summary LSA로 수신

Stub Area:
- External LSA 수신 안함
- 기본 경로로 외부 도달
- ABR에서 자동으로 기본 경로 주입

Totally Stub Area (Cisco 독점):
- External과 Summary LSA 모두 차단
- 기본 경로만 사용

NSSA (Not-So-Stubby Area):
- 제한적인 외부 경로 허용
- Type 7 LSA 사용
```

## OSPF 라우터 유형

### 라우터 역할
```
Internal Router:
- 하나의 Area에만 속함
- 같은 Area 내 라우터와만 인접

ABR (Area Border Router):
- 두 개 이상의 Area에 연결
- Area 0에 반드시 연결되어야 함
- Summary LSA 생성

ASBR (Autonomous System Boundary Router):
- 외부 AS와 연결
- 외부 경로를 OSPF로 재분배
- External LSA 생성

Backbone Router:
- Area 0에 연결된 모든 라우터
- ABR도 Backbone Router임
```

## OSPF 패킷 유형

### 5가지 패킷
```
1. Hello Packet:
   - 이웃 발견 및 유지
   - 10초 간격 (기본값)
   - Dead Time: 40초

2. DBD (Database Description):
   - LSA 헤더 정보 교환
   - 동기화 시작

3. LSR (Link State Request):
   - 특정 LSA 요청

4. LSU (Link State Update):
   - 실제 LSA 데이터 전송

5. LSAck (Link State Acknowledgment):
   - LSA 수신 확인
```

### Hello 패킷 매개변수
```
일치해야 하는 항목:
- Area ID
- Authentication  
- Hello Interval
- Dead Interval
- Network Mask (Point-to-Point 제외)
- Stub Area Flag
```

## OSPF 인접 관계 (Adjacency)

### Neighbor 상태 변화
```
Down → Init → 2-Way → ExStart → Exchange → Loading → Full

Down: 이웃 정보 없음
Init: Hello 패킷 수신
2-Way: 양방향 통신 확인
ExStart: Master/Slave 결정
Exchange: DBD 패킷 교환
Loading: LSR/LSU 교환
Full: 완전한 인접 관계
```

### DR/BDR 선출
```
목적: 멀티액세스 네트워크에서 LSA 교환 최적화

선출 기준:
1. OSPF Priority (높을수록 우선)
2. Router ID (높을수록 우선)

Priority 값:
- 0: DR/BDR 되지 않음
- 1-255: 참여 (기본값: 1)

예시:
Router(config-if)# ip ospf priority 100
```

## OSPF 기본 설정

### 기본 설정
```cisco
# OSPF 프로세스 시작
Router(config)# router ospf 1

# Router ID 설정 (권장)
Router(config-router)# router-id 1.1.1.1

# 네트워크 선언
Router(config-router)# network 192.168.1.0 0.0.0.255 area 0
Router(config-router)# network 10.1.1.0 0.0.0.3 area 1

# 인터페이스별 설정
Router(config)# interface fastethernet 0/0
Router(config-if)# ip ospf 1 area 0
```

### 고급 설정
```cisco
# 수동 이웃 설정 (NBMA)
Router(config-router)# neighbor 10.1.1.2

# 가상 링크 (Virtual Link)
Router(config-router)# area 1 virtual-link 2.2.2.2

# 인증 설정
Router(config-router)# area 0 authentication message-digest
Router(config-if)# ip ospf message-digest-key 1 md5 cisco123

# Stub Area 설정
Router(config-router)# area 1 stub
Router(config-router)# area 1 stub no-summary  # Totally Stub

# 기본 경로 주입
Router(config-router)# default-information originate
```

## OSPF 확인 명령어

### 기본 확인
```cisco
# OSPF 프로세스 정보
Router# show ip ospf

# 이웃 라우터 확인
Router# show ip ospf neighbor

# 데이터베이스 확인
Router# show ip ospf database

# 인터페이스 정보
Router# show ip ospf interface

# 라우팅 테이블에서 OSPF 경로
Router# show ip route ospf
```

### 상세 확인
```cisco
# 특정 이웃 상세 정보
Router# show ip ospf neighbor detail

# LSA 유형별 확인
Router# show ip ospf database router
Router# show ip ospf database network
Router# show ip ospf database summary

# 특정 Area 정보
Router# show ip ospf database area 1

# 라우터 ID 및 프로세스 정보
Router# show ip ospf | include Router ID
```

## OSPF 문제해결

### 일반적인 문제
```
1. 인접 관계 형성 실패
   원인: Hello 매개변수 불일치
   확인: show ip ospf interface
   해결: 매개변수 통일

2. 라우팅 테이블에 경로 없음
   원인: Area 연결 문제, LSA 전파 실패
   확인: show ip ospf database
   해결: Area 설정, 네트워크 선언 확인

3. 느린 컨버전스
   원인: Timer 설정, 네트워크 크기
   확인: show ip ospf neighbor
   해결: Timer 조정, Area 분할

4. DR/BDR 문제
   원인: Priority 설정, 선출 과정
   확인: show ip ospf neighbor
   해결: Priority 조정, 프로세스 재시작
```

### 디버깅
```cisco
# OSPF 인접 관계 디버그
Router# debug ip ospf adj

# Hello 패킷 디버그
Router# debug ip ospf hello

# LSA 디버그
Router# debug ip ospf lsa-generation

# SPF 계산 디버그
Router# debug ip ospf spf
```

## OSPF 최적화

### 성능 튜닝
```cisco
# Hello/Dead Timer 조정
Router(config-if)# ip ospf hello-interval 5
Router(config-if)# ip ospf dead-interval 20

# SPF 계산 지연 조정
Router(config-router)# timers throttle spf 5 10 40

# LSA 그룹 페이싱
Router(config-router)# timers lsa-group-pacing 60

# 최대 경로 수 증가
Router(config-router)# maximum-paths 8
```

### 메모리 최적화
```cisco
# LSA 플러딩 제한
Router(config-if)# ip ospf flood-reduction

# 불필요한 LSA 필터링
Router(config-router)# area 1 filter-list prefix FILTER_LIST in
```

## OSPF 보안

### 인증 설정
```cisco
# Area 단위 인증
Router(config-router)# area 0 authentication
Router(config-if)# ip ospf authentication-key cisco123

# MD5 인증 (권장)
Router(config-router)# area 0 authentication message-digest
Router(config-if)# ip ospf message-digest-key 1 md5 cisco123

# 인터페이스별 인증
Router(config-if)# ip ospf authentication message-digest
Router(config-if)# ip ospf message-digest-key 1 md5 secretkey
```

### 보안 모범 사례
```
1. 모든 Area에서 인증 사용
2. 강력한 패스워드 사용
3. 정기적인 키 교체
4. 불필요한 인접 관계 차단
5. LSA 필터링 적용
```

## Multi-Area OSPF 설계

### 설계 원칙
```
1. Backbone Area (Area 0) 필수
2. 모든 Regular Area는 Area 0에 연결
3. Area당 50개 이하 라우터 권장
4. 계층적 주소 할당
5. Summarization 활용
```

### 주소 요약 (Summarization)
```cisco
# ABR에서 Inter-Area 요약
Router(config-router)# area 1 range 192.168.0.0 255.255.252.0

# ASBR에서 External 요약  
Router(config-router)# summary-address 10.0.0.0 255.0.0.0
```

### Virtual Link
```cisco
# Area 0 연결이 불가능한 경우
Router(config-router)# area 1 virtual-link 2.2.2.2

# Virtual Link를 통한 LSA 전달
# Transit Area는 Regular Area여야 함
```

## 관련 주제
- [[라우팅 기초]] - 라우팅 프로토콜 기초
- [[EIGRP]] - 다른 고급 라우팅 프로토콜
- [[정적 라우팅]] - 정적 경로와의 비교
- [[문제해결 방법론]] - OSPF 문제해결

## 태그
#ccna #ospf #라우팅프로토콜 #링크상태 #동적라우팅
