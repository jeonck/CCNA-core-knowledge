# RIP (Routing Information Protocol)

## 📝 개요
RIP는 가장 오래된 거리 벡터(Distance Vector) 라우팅 프로토콜 중 하나로, 홉 카운트를 메트릭으로 사용하는 간단한 IGP(Interior Gateway Protocol)입니다.

## 🎯 RIP의 특징

### 기본 특성
- **프로토콜 타입**: Distance Vector
- **메트릭**: 홉 카운트 (Hop Count)
- **최대 홉 카운트**: 15 (16은 무한대로 간주)
- **업데이트 주기**: 30초마다 정기 업데이트
- **AD (Administrative Distance)**: 120

### RIP 버전 비교
| 특징 | RIPv1 | RIPv2 |
|------|-------|-------|
| **RFC** | RFC 1058 | RFC 2453 |
| **클래스** | Classful | Classless |
| **VLSM 지원** | 미지원 | 지원 |
| **인증** | 미지원 | 지원 |
| **멀티캐스트** | 브로드캐스트 | 224.0.0.9 |
| **라우트 태깅** | 미지원 | 지원 |

## 🔄 RIP 동작 원리

### Distance Vector 알고리즘
```
Bellman-Ford 알고리즘 기반:
1. 초기화: 직접 연결된 네트워크만 알고 있음
2. 정보 교환: 인접 라우터와 라우팅 정보 교환
3. 경로 계산: 더 나은 경로 발견 시 업데이트
4. 수렴: 모든 라우터가 일관된 라우팅 정보 유지
```

### 업데이트 과정
```
1. 정기 업데이트 (30초)
   - 전체 라우팅 테이블 전송
   - UDP 포트 520 사용

2. 트리거 업데이트
   - 토폴로지 변화 시 즉시 전송
   - 수렴 시간 단축

3. 업데이트 처리
   - Split Horizon 적용
   - Poison Reverse 처리
   - 홉 카운트 증가
```

## ⏰ RIP 타이머

### 기본 타이머
| 타이머 | 기본값 | 설명 |
|--------|--------|------|
| **Update Timer** | 30초 | 정기 업데이트 간격 |
| **Invalid Timer** | 180초 | 업데이트 없으면 invalid 상태 |
| **Holddown Timer** | 180초 | 새 정보 수용 대기 시간 |
| **Flush Timer** | 240초 | 라우팅 테이블에서 완전 삭제 |

### 타이머 설정
```cisco
Router(config)# router rip
Router(config-router)# timers basic update invalid holddown flush
Router(config-router)# timers basic 30 180 180 240  # 기본값
```

## 🚫 루프 방지 메커니즘

### Split Horizon
```
원리: 정보를 받은 인터페이스로 같은 정보를 다시 보내지 않음

예시:
R1 → R2: "네트워크 A에 도달하려면 나를 거쳐라"
R2는 이 정보를 R1으로 다시 보내지 않음
```

### Poison Reverse
```
원리: 도달 불가능한 네트워크를 홉 카운트 16으로 광고

예시:
네트워크 A 장애 시:
R1 → R2: "네트워크 A의 홉 카운트는 16 (무한대)"
```

### Hold-down Timer
```
원리: 네트워크가 down된 후 일정 시간 동안 새로운 정보 무시

과정:
1. 네트워크 장애 감지
2. Hold-down 타이머 시작 (180초)
3. 타이머 만료 전까지 더 나쁜 정보 무시
4. 타이머 만료 후 새로운 정보 수용
```

## ⚙️ RIP 설정

### RIPv1 설정
```cisco
# RIP 활성화
Router(config)# router rip
Router(config-router)# version 1

# 네트워크 선언 (클래스풀 주소만)
Router(config-router)# network 192.168.1.0
Router(config-router)# network 10.0.0.0

# 수동 요약 (자동 요약 기본 활성화)
Router(config-router)# auto-summary
```

### RIPv2 설정
```cisco
# RIPv2 활성화
Router(config)# router rip
Router(config-router)# version 2

# 네트워크 선언
Router(config-router)# network 192.168.1.0
Router(config-router)# network 10.0.0.0

# 자동 요약 비활성화 (VLSM 지원)
Router(config-router)# no auto-summary
```

### 고급 설정 옵션
```cisco
# 수동 요약
Router(config)# interface serial0/0/0
Router(config-if)# ip summary-address rip 192.168.0.0 255.255.252.0

# RIP 비활성화 (특정 인터페이스)
Router(config-router)# passive-interface fastethernet0/0

# 유니캐스트 neighbor
Router(config-router)# neighbor 10.1.1.2

# 기본 경로 전파
Router(config-router)# default-information originate

# 최대 경로 수 설정
Router(config-router)# maximum-paths 4
```

## 🔐 RIPv2 인증

### 평문 인증 (Plain Text)
```cisco
# 키 체인 생성
Router(config)# key chain RIP_KEYS
Router(config-keychain)# key 1
Router(config-keychain-key)# key-string cisco123

# 인터페이스 적용
Router(config)# interface serial0/0/0
Router(config-if)# ip rip authentication mode text
Router(config-if)# ip rip authentication key-chain RIP_KEYS
```

### MD5 인증
```cisco
# 키 체인 생성
Router(config)# key chain RIP_KEYS
Router(config-keychain)# key 1
Router(config-keychain-key)# key-string cisco123

# 인터페이스 적용
Router(config)# interface serial0/0/0
Router(config-if)# ip rip authentication mode md5
Router(config-if)# ip rip authentication key-chain RIP_KEYS
```

## 📊 RIP 확인 및 모니터링

### 기본 확인 명령어
```cisco
# RIP 구성 정보
Router# show ip protocols

# RIP 데이터베이스
Router# show ip rip database

# 라우팅 테이블 (RIP 경로만)
Router# show ip route rip

# RIP 인터페이스 정보
Router# show ip interface brief
```

### 상세 정보 확인
```cisco
# 상세 RIP 정보
Router# show ip protocols

Output:
Routing Protocol is "rip"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Sending updates every 30 seconds, next due in 22 seconds
  Invalid after 180 seconds, hold down 180, flushed after 240
  Redistributing: rip
  Default version control: send version 2, receive version 2
    Interface             Send  Recv  Triggered RIP  Key-chain
    Serial0/0/0           2     2                     
    FastEthernet0/0       2     2                     
  Automatic network summarization is not in effect
  Maximum path: 4
  Routing for Networks:
    10.0.0.0
    192.168.1.0
  Routing Information Sources:
    Gateway         Distance      Last Update
    10.1.1.2            120      00:00:24
  Distance: (default is 120)
```

## 🐛 디버깅과 문제해결

### 디버깅 명령어
```cisco
# RIP 이벤트 디버그
Router# debug ip rip events

# RIP 패킷 디버그
Router# debug ip rip

# RIP 데이터베이스 디버그
Router# debug ip rip database

# 디버그 종료
Router# undebug all
```

### 일반적인 문제들

#### 1. 이웃 관계 형성 실패
```
문제: RIP 업데이트 수신 안됨
원인:
- 네트워크 명령어 누락
- 버전 불일치
- 인증 설정 오류

해결:
1. show ip protocols로 설정 확인
2. debug ip rip로 패킷 확인
3. 네트워크 명령어 재설정
```

#### 2. 경로 학습 실패
```
문제: 원격 네트워크 학습 안됨
원인:
- 자동 요약 문제
- Split Horizon 문제
- 필터링 설정

해결:
1. no auto-summary 설정
2. 인터페이스별 RIP 설정 확인
3. 접근 목록 확인
```

#### 3. 느린 수렴
```
문제: 토폴로지 변화 시 오랜 수렴 시간
원인: RIP의 기본적인 한계

해결:
1. 타이머 조정 (신중하게)
2. 다른 라우팅 프로토콜 고려
3. 네트워크 설계 개선
```

## 🚀 RIP 최적화

### 성능 개선 방법
```cisco
# 타이머 최적화 (신중하게 적용)
Router(config-router)# timers basic 15 90 90 120

# 불필요한 업데이트 방지
Router(config-router)# passive-interface default
Router(config-router)# no passive-interface serial0/0/0

# 수동 요약으로 라우팅 테이블 크기 축소
Router(config-if)# ip summary-address rip 192.168.0.0 255.255.248.0
```

### 라우트 필터링
```cisco
# 배포 목록 (distribute-list)
Router(config)# access-list 10 deny 192.168.10.0 0.0.0.255
Router(config)# access-list 10 permit any

Router(config)# router rip
Router(config-router)# distribute-list 10 out serial0/0/0

# 라우트 맵 사용
Router(config)# route-map FILTER_ROUTES permit 10
Router(config-route-map)# match ip address 10

Router(config)# router rip
Router(config-router)# distribute-list route-map FILTER_ROUTES out
```

## 🌐 RIP과 다른 프로토콜 비교

### 프로토콜 비교표
| 특징 | RIP | OSPF | EIGRP |
|------|-----|------|-------|
| **타입** | Distance Vector | Link State | Advanced Distance Vector |
| **메트릭** | 홉 카운트 | Cost | Composite |
| **수렴 속도** | 느림 | 빠름 | 빠름 |
| **확장성** | 제한적 | 높음 | 높음 |
| **CPU 사용** | 낮음 | 높음 | 중간 |
| **메모리 사용** | 낮음 | 높음 | 중간 |

### 사용 사례
```
RIP 적합한 경우:
✅ 소규모 네트워크 (< 15 홉)
✅ 단순한 토폴로지
✅ 제한된 하드웨어 리소스
✅ 학습/테스트 목적

RIP 부적합한 경우:
❌ 대규모 네트워크
❌ 복잡한 토폴로지
❌ 빠른 수렴 요구
❌ 정밀한 경로 제어 필요
```

## 📋 설정 예시

### 기본 네트워크 설정
```
토폴로지:
R1 [192.168.1.0/24] --- [10.1.1.0/30] --- R2 [192.168.2.0/24]
```

#### R1 설정
```cisco
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# interface fastethernet0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown

R1(config)# interface serial0/0/0
R1(config-if)# ip address 10.1.1.1 255.255.255.252
R1(config-if)# no shutdown

R1(config)# router rip
R1(config-router)# version 2
R1(config-router)# no auto-summary
R1(config-router)# network 192.168.1.0
R1(config-router)# network 10.0.0.0
```

#### R2 설정
```cisco
Router> enable
Router# configure terminal
Router(config)# hostname R2
R2(config)# interface fastethernet0/0
R2(config-if)# ip address 192.168.2.1 255.255.255.0
R2(config-if)# no shutdown

R2(config)# interface serial0/0/0
R2(config-if)# ip address 10.1.1.2 255.255.255.252
R2(config-if)# no shutdown

R2(config)# router rip
R2(config-router)# version 2
R2(config-router)# no auto-summary
R2(config-router)# network 192.168.2.0
R2(config-router)# network 10.0.0.0
```

## 🔗 관련 주제
- [[라우팅 기초|라우팅]] - 라우팅의 기본 개념
- [[OSPF|OSPF]] - 링크 상태 프로토콜
- [[EIGRP|EIGRP]] - Cisco 고급 프로토콜
- [[정적 라우팅|정적 라우팅]] - 수동 라우팅과 비교

## 🏷️ 태그
#ccna #rip #라우팅프로토콜 #distance-vector #igp #동적라우팅 #홉카운트