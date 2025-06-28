# EtherChannel

## 📝 개요
EtherChannel은 여러 개의 물리적 이더넷 링크를 하나의 논리적 링크로 결합하여 대역폭을 증가시키고 중복성을 제공하는 기술입니다.

## 🎯 EtherChannel의 장점

### 대역폭 증가
- **2개 링크**: 2배 대역폭
- **4개 링크**: 4배 대역폭  
- **8개 링크**: 8배 대역폭 (최대)

### 중복성 (Redundancy)
- 하나의 링크 장애 시 자동 복구
- 무중단 서비스 제공
- **STP 차단 없음**: 모든 링크 활성 상태

### 로드 밸런싱
- 트래픽을 여러 링크에 분산
- 효율적인 대역폭 활용
- 성능 최적화

## 🔧 EtherChannel 프로토콜

### PAgP (Port Aggregation Protocol)
- **벤더**: Cisco 독점
- **IEEE 표준**: 아님
- **기본 모드**: Desirable

#### PAgP 모드
| 모드 | 설명 | 동작 |
|------|------|------|
| **On** | 강제로 채널 형성 | 협상 없이 바로 형성 |
| **Auto** | 수동적 협상 | 상대방 요청 시 응답 |
| **Desirable** | 능동적 협상 | 적극적으로 채널 형성 시도 |

#### PAgP 설정
```cisco
# PAgP 설정
Switch(config)# interface range gi0/1-2
Switch(config-if-range)# channel-protocol pagp
Switch(config-if-range)# channel-group 1 mode desirable

# 또는
Switch(config-if-range)# channel-group 1 mode auto
```

### LACP (Link Aggregation Control Protocol)
- **표준**: IEEE 802.3ad
- **벤더**: 개방형 표준
- **권장**: 멀티벤더 환경에서 선호

#### LACP 모드
| 모드 | 설명 | 동작 |
|------|------|------|
| **On** | 강제로 채널 형성 | 협상 없이 바로 형성 |
| **Passive** | 수동적 협상 | 상대방 요청 시 응답 |
| **Active** | 능동적 협상 | 적극적으로 채널 형성 시도 |

#### LACP 설정
```cisco
# LACP 설정
Switch(config)# interface range gi0/1-2
Switch(config-if-range)# channel-protocol lacp
Switch(config-if-range)# channel-group 1 mode active

# 또는
Switch(config-if-range)# channel-group 1 mode passive
```

## 🔄 로드 밸런싱 방법

### 기본 로드 밸런싱
Cisco 스위치는 다양한 로드 밸런싱 방법을 지원합니다.

#### 로드 밸런싱 옵션
| 방법 | 설명 | 사용 사례 |
|------|------|-----------|
| **src-mac** | 소스 MAC 주소 기반 | 다양한 클라이언트 |
| **dst-mac** | 목적지 MAC 주소 기반 | 다양한 서버 |
| **src-dst-mac** | 소스+목적지 MAC | 일반적인 용도 |
| **src-ip** | 소스 IP 주소 기반 | Layer 3 환경 |
| **dst-ip** | 목적지 IP 주소 기반 | 서버 팜 |
| **src-dst-ip** | 소스+목적지 IP | Layer 3 일반 |
| **src-port** | 소스 포트 기반 | 애플리케이션별 |
| **dst-port** | 목적지 포트 기반 | 서비스별 |
| **src-dst-port** | 소스+목적지 포트 | 세밀한 분산 |

#### 로드 밸런싱 설정
```cisco
# 글로벌 로드 밸런싱 설정
Switch(config)# port-channel load-balance src-dst-ip

# 현재 설정 확인
Switch# show port-channel load-balance
```

## ⚙️ EtherChannel 설정

### 기본 설정 단계
1. **물리 인터페이스 설정 통일**
2. **채널 그룹 생성**
3. **포트-채널 인터페이스 설정**

### Layer 2 EtherChannel
```cisco
# 물리 인터페이스 설정
Switch(config)# interface range gi0/1-4
Switch(config-if-range)# switchport mode trunk
Switch(config-if-range)# switchport trunk allowed vlan 10,20,30
Switch(config-if-range)# switchport trunk native vlan 99

# LACP EtherChannel 생성
Switch(config-if-range)# channel-protocol lacp
Switch(config-if-range)# channel-group 1 mode active

# 포트-채널 인터페이스 설정
Switch(config)# interface port-channel 1
Switch(config-if)# description "Link to Core Switch"
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20,30
Switch(config-if)# switchport trunk native vlan 99
```

### Layer 3 EtherChannel
```cisco
# 물리 인터페이스를 Layer 3로 설정
Switch(config)# interface range gi0/1-2
Switch(config-if-range)# no switchport
Switch(config-if-range)# channel-group 1 mode active

# 포트-채널 인터페이스에 IP 할당
Switch(config)# interface port-channel 1
Switch(config-if)# ip address 10.1.1.1 255.255.255.252
Switch(config-if)# no shutdown
```

## 📊 EtherChannel 확인 및 모니터링

### 기본 확인 명령어
```cisco
# EtherChannel 요약 정보
Switch# show etherchannel summary

Output:
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-------
1      Po1(SU)       LACP        Gi0/1(P)    Gi0/2(P)

# 상세 정보
Switch# show etherchannel 1 detail
Switch# show etherchannel port-channel

# 포트별 상태
Switch# show interfaces gi0/1 etherchannel
Switch# show interfaces port-channel 1

# LACP 정보
Switch# show lacp neighbor
Switch# show lacp sys-id
Switch# show lacp counters

# PAgP 정보 (PAgP 사용 시)
Switch# show pagp neighbor
Switch# show pagp counters
```

### 상태 코드 해석
#### 포트-채널 상태
| 코드 | 의미 |
|------|------|
| **SU** | Stand-alone, Up |
| **RU** | Routed, Up |
| **P** | Port is in port-channel |
| **D** | Port is down |
| **I** | Port is in independent mode |

#### 개별 포트 상태
| 코드 | 의미 |
|------|------|
| **P** | Port is bundled in port-channel |
| **I** | Port is in independent mode |
| **s** | Port is suspended |
| **H** | Port is hot-standby |
| **D** | Port is down |

## 🚨 문제해결

### 일반적인 문제들

#### 1. 설정 불일치
```
문제: 포트들이 번들되지 않음
원인: 인터페이스 설정이 다름

확인:
- VLAN 설정
- 트렁크 설정
- 속도/듀플렉스
- 액세스 모드
```

#### 2. 프로토콜 불일치
```
문제: 채널 형성 실패
원인: 양쪽 프로토콜이 다름

해결:
- 같은 프로토콜 사용 (PAgP 또는 LACP)
- 호환되는 모드 설정
```

#### 3. STP 문제
```
문제: 일부 포트가 차단됨
원인: EtherChannel 형성 전 STP 동작

해결:
- EtherChannel 우선 설정
- STP 설정 확인
```

### 문제해결 절차
```cisco
# 1. 현재 상태 확인
show etherchannel summary
show interfaces status

# 2. 설정 비교
show running-config interface gi0/1
show running-config interface gi0/2

# 3. 프로토콜 상태 확인
show lacp neighbor detail
show lacp sys-id

# 4. 에러 카운터 확인
show interfaces gi0/1 counters errors
show lacp counters

# 5. 로그 확인
show logging | include ETHCHNL
```

### 디버깅
```cisco
# EtherChannel 디버그
debug etherchannel events
debug etherchannel detail

# LACP 디버그
debug lacp
debug lacp events
debug lacp packets

# PAgP 디버그 (PAgP 사용 시)
debug pagp events
debug pagp packets
```

## ⚡ 고급 기능

### LACP 우선순위
```cisco
# 시스템 우선순위 설정 (낮을수록 높은 우선순위)
Switch(config)# lacp system-priority 100

# 포트 우선순위 설정
Switch(config)# interface gi0/1
Switch(config-if)# lacp port-priority 100
```

### Fast Switching
```cisco
# 빠른 스위칭 활성화
Switch(config)# interface port-channel 1
Switch(config-if)# port-channel min-links 2
```

### LACP Rate
```cisco
# LACP PDU 전송 간격 설정
Switch(config)# interface gi0/1
Switch(config-if)# lacp rate fast    # 1초 간격
# 또는
Switch(config-if)# lacp rate normal  # 30초 간격 (기본값)
```

## 🛡️ 보안 고려사항

### 보안 모범 사례
```cisco
# 1. 미사용 포트 비활성화
Switch(config)# interface range gi0/5-24
Switch(config-if-range)# shutdown

# 2. LACP 인증 (지원되는 경우)
Switch(config)# interface gi0/1
Switch(config-if)# lacp key 12345

# 3. 포트 보안과 함께 사용
Switch(config)# interface gi0/1
Switch(config-if)# switchport port-security
Switch(config-if)# switchport port-security maximum 1
```

## 🏗️ 설계 고려사항

### 최적 구성
```
권장사항:
1. 동일한 속도/듀플렉스 사용
2. 최대 8개 포트까지 번들
3. 같은 모듈/카드 사용 권장
4. 로드 밸런싱 방법 신중 선택
```

### 용량 계획
```
대역폭 계산:
- 2 x 1Gbps = 2Gbps 총 대역폭
- 4 x 1Gbps = 4Gbps 총 대역폭
- 하지만 실제 성능은 트래픽 패턴에 따라 달라짐
```

## 🔗 관련 주제
- [[스위치 동작원리|스위칭]] - EtherChannel의 기반
- [[STP|Spanning Tree]] - STP와 EtherChannel 상호작용
- [[VLAN 구성|VLAN]] - 트렁크 EtherChannel
- [[QoS|서비스 품질]] - EtherChannel에서의 QoS

## 🏷️ 태그
#ccna #etherchannel #lacp #pagp #로드밸런싱 #대역폭증가 #중복성 #스위칭