# SNMP (Simple Network Management Protocol)

## 📝 개요
SNMP는 네트워크 장비를 중앙에서 관리하고 모니터링하기 위한 표준 프로토콜입니다. 네트워크 관리 시스템(NMS)이 라우터, 스위치, 서버 등의 장비 상태를 원격으로 감시하고 제어할 수 있게 합니다.

## 🏗️ SNMP 아키텍처

### 주요 구성 요소
```
NMS (Network Management Station)
    ↓ SNMP (UDP 161/162)
Agent (SNMP-enabled Device)
    ↓ 
MIB (Management Information Base)
```

#### NMS (Network Management Station)
- **역할**: 네트워크 관리 소프트웨어
- **기능**: 
  - 장비 모니터링
  - 성능 데이터 수집
  - 알람 수신
  - 설정 변경
- **예시**: SolarWinds, PRTG, Nagios, Zabbix

#### Agent
- **역할**: 관리 대상 장비에서 실행되는 소프트웨어
- **기능**:
  - MIB 정보 제공
  - NMS 요청 처리
  - Trap 메시지 전송
- **위치**: 라우터, 스위치, 서버, 프린터

#### MIB (Management Information Base)
- **역할**: 관리 가능한 객체들의 데이터베이스
- **구조**: 계층적 트리 구조
- **식별**: OID (Object Identifier)

## 🔢 SNMP 버전 비교

### SNMP v1
```
특징:
✅ 최초 버전 (RFC 1157)
✅ 단순한 구현
❌ 보안 기능 없음
❌ 커뮤니티 스트링 (평문)
❌ 32비트 카운터 제한

보안: 커뮤니티 기반 (평문)
```

### SNMP v2c
```
특징:
✅ 향상된 데이터 타입
✅ 64비트 카운터 지원
✅ GetBulk 연산 추가
✅ 향상된 오류 처리
❌ 여전히 평문 전송

보안: 커뮤니티 기반 (v1과 동일)
```

### SNMP v3
```
특징:
✅ 강력한 보안 기능
✅ 인증 (Authentication)
✅ 암호화 (Privacy)
✅ 사용자 기반 보안 모델
✅ 뷰 기반 접근 제어

보안: USM (User-based Security Model)
```

## 🔧 SNMP 동작 원리

### SNMP 메시지 유형
| 메시지 | 방향 | 포트 | 설명 |
|--------|------|------|------|
| **GET** | NMS → Agent | 161 | 단일 객체 값 요청 |
| **GET-NEXT** | NMS → Agent | 161 | 다음 객체 값 요청 |
| **GET-BULK** | NMS → Agent | 161 | 다중 객체 값 요청 (v2c+) |
| **SET** | NMS → Agent | 161 | 객체 값 설정 |
| **RESPONSE** | Agent → NMS | - | 요청에 대한 응답 |
| **TRAP** | Agent → NMS | 162 | 비동기 알림 |
| **INFORM** | Agent → NMS | 162 | 확인이 필요한 알림 (v2c+) |

### OID (Object Identifier) 구조
```
1.3.6.1.2.1.1.1.0 (sysDescr.0)
│ │ │ │ │ │ │ │ │
│ │ │ │ │ │ │ │ └─ Instance (0 = scalar)
│ │ │ │ │ │ │ └─── Object (1 = sysDescr)
│ │ │ │ │ │ └───── Group (1 = system)
│ │ │ │ │ └─────── MIB-2 (1)
│ │ │ │ └───────── Management (2)
│ │ │ └─────────── Internet (1)
│ │ └───────────── DOD (6)
│ └─────────────── ISO identified-org (3)
└───────────────── ISO (1)
```

### 주요 MIB-II 객체
| OID | 이름 | 설명 |
|-----|------|------|
| 1.3.6.1.2.1.1.1.0 | sysDescr | 시스템 설명 |
| 1.3.6.1.2.1.1.3.0 | sysUpTime | 시스템 가동 시간 |
| 1.3.6.1.2.1.1.5.0 | sysName | 시스템 이름 |
| 1.3.6.1.2.1.1.6.0 | sysLocation | 시스템 위치 |
| 1.3.6.1.2.1.2.2.1.10 | ifInOctets | 인터페이스 수신 바이트 |
| 1.3.6.1.2.1.2.2.1.16 | ifOutOctets | 인터페이스 송신 바이트 |

## ⚙️ Cisco 장비에서 SNMP 설정

### SNMP v1/v2c 설정
```cisco
# 기본 SNMP 설정
Router(config)# snmp-server community public RO
Router(config)# snmp-server community private RW

# 읽기 전용 커뮤니티 (ACL 적용)
Router(config)# snmp-server community monitor RO 10
Router(config)# access-list 10 permit 192.168.1.0 0.0.0.255

# 시스템 정보 설정
Router(config)# snmp-server contact admin@company.com
Router(config)# snmp-server location "Server Room A"
Router(config)# snmp-server chassis-id "Router-01"

# Trap 설정
Router(config)# snmp-server host 192.168.1.100 public
Router(config)# snmp-server enable traps snmp
Router(config)# snmp-server enable traps config
Router(config)# snmp-server enable traps entity
```

### SNMP v3 설정
```cisco
# SNMP v3 사용자 생성
Router(config)# snmp-server user admin ADMIN-GROUP v3 auth sha cisco123 priv aes 128 cisco456

# 그룹 설정
Router(config)# snmp-server group ADMIN-GROUP v3 auth read ADMIN-VIEW write ADMIN-VIEW

# 뷰 설정
Router(config)# snmp-server view ADMIN-VIEW iso included
Router(config)# snmp-server view RESTRICTED-VIEW 1.3.6.1.2.1.1 included

# v3 호스트 설정
Router(config)# snmp-server host 192.168.1.100 version 3 auth admin
```

### 고급 SNMP 설정
```cisco
# SNMP 엔진 ID 설정
Router(config)# snmp-server engineID local 80000009030000B064EFE100

# ifIndex 지속성
Router(config)# snmp-server ifindex persist

# SNMP 큐 크기
Router(config)# snmp-server queue-length 50

# Context 매핑
Router(config)# snmp-server context context1
Router(config)# snmp-server group CONTEXT-GROUP v3 auth context context1

# 알림 필터링
Router(config)# snmp-server host 192.168.1.100 public config entity
```

## 📊 SNMP 모니터링 및 확인

### 기본 확인 명령어
```cisco
# SNMP 상태 확인
Router# show snmp

# 커뮤니티 정보
Router# show snmp community

# SNMP 사용자 (v3)
Router# show snmp user

# SNMP 그룹 (v3)  
Router# show snmp group

# SNMP 엔진 정보
Router# show snmp engineID

# SNMP 통계
Router# show snmp stats
```

### 출력 예시 해석
```
Router# show snmp
Chassis: 1841
Contact: admin@company.com
Location: Server Room A

SNMP packets input: 1543, bad SNMP version: 0, unknown community name: 12
SNMP packets output: 1456, too big: 0, maximum packet size: 1500
SNMP logging: disabled

Authentication failures: 5
```

## 🔍 MIB 탐색 및 활용

### 주요 표준 MIB
```
MIB-II (RFC 1213): 기본 네트워크 관리 객체
├── system: 시스템 정보
├── interfaces: 인터페이스 정보  
├── ip: IP 관련 정보
├── icmp: ICMP 통계
├── tcp: TCP 연결 정보
├── udp: UDP 통계
└── snmp: SNMP 통계

Enterprise MIB:
├── Cisco MIB: 1.3.6.1.4.1.9
├── HP MIB: 1.3.6.1.4.1.11
└── Microsoft MIB: 1.3.6.1.4.1.311
```

### 유용한 Cisco MIB 객체
```
CPU 사용률:
1.3.6.1.4.1.9.9.109.1.1.1.1.7 (cpmCPUTotal5minRev)

메모리 사용률:
1.3.6.1.4.1.9.9.48.1.1.1.5 (ciscoMemoryPoolUsed)
1.3.6.1.4.1.9.9.48.1.1.1.6 (ciscoMemoryPoolFree)

온도 정보:
1.3.6.1.4.1.9.9.13.1.3.1.3 (ciscoEnvMonTemperatureValue)

전원 상태:
1.3.6.1.4.1.9.9.13.1.5.1.3 (ciscoEnvMonSupplyState)
```

## 🚨 SNMP Trap 관리

### Trap 유형 및 설정
```cisco
# 기본 Trap 활성화
Router(config)# snmp-server enable traps snmp authentication linkdown linkup

# 인터페이스 관련 Trap
Router(config)# snmp-server enable traps interface

# 설정 변경 Trap
Router(config)# snmp-server enable traps config

# 환경 모니터링 Trap
Router(config)# snmp-server enable traps envmon temperature shutdown supply fan

# CPU/메모리 Trap (임계값 기반)
Router(config)# process cpu threshold type total rising 80 interval 5

# HSRP Trap
Router(config)# snmp-server enable traps hsrp

# 보안 관련 Trap
Router(config)# snmp-server enable traps auth-framework sec-violation
```

### Trap 수신 서버 설정
```cisco
# 기본 Trap 호스트
Router(config)# snmp-server host 192.168.1.100 public

# 특정 Trap만 전송
Router(config)# snmp-server host 192.168.1.101 private config envmon

# SNMP v3 Trap
Router(config)# snmp-server host 192.168.1.102 version 3 auth admin config
```

## 🔐 SNMP 보안

### 보안 위험과 대응
```
위험 요소:
1. 평문 커뮤니티 스트링 (v1/v2c)
2. 기본 커뮤니티 이름 (public/private)
3. 무제한 접근
4. 정보 노출

보안 대책:
1. SNMP v3 사용
2. 강력한 커뮤니티 스트링
3. 접근 제어 목록 (ACL)
4. 읽기 전용 권한
5. 네트워크 분할
```

### SNMP v3 보안 레벨
| 레벨 | 인증 | 암호화 | 설명 |
|------|------|--------|------|
| **noAuthNoPriv** | 없음 | 없음 | 사용자 이름만 |
| **authNoPriv** | 있음 | 없음 | 인증만 (MD5/SHA) |
| **authPriv** | 있음 | 있음 | 인증 + 암호화 (DES/AES) |

### 보안 강화 설정
```cisco
# 기본 커뮤니티 비활성화
Router(config)# no snmp-server community public
Router(config)# no snmp-server community private

# 강력한 v3 사용자 생성
Router(config)# snmp-server user secureuser SECURE-GROUP v3 auth sha SecurePass123! priv aes 256 PrivateKey456!

# 제한된 뷰 설정
Router(config)# snmp-server view SAFE-VIEW 1.3.6.1.2.1.1 included
Router(config)# snmp-server view SAFE-VIEW 1.3.6.1.2.1.2.2.1.2 included
Router(config)# snmp-server group SECURE-GROUP v3 auth read SAFE-VIEW

# 소스 인터페이스 지정
Router(config)# snmp-server source-interface traps loopback0

# Trap 인증 실패 비활성화 (DoS 방지)
Router(config)# no snmp-server enable traps snmp authentication
```

## 🛠️ SNMP 문제해결

### 일반적인 문제들

#### 1. SNMP 응답 없음
```
문제: NMS에서 장비에 연결할 수 없음
원인:
- 잘못된 커뮤니티 스트링
- ACL 차단
- 방화벽 규칙
- SNMP 서비스 비활성화

진단:
Router# show snmp community
Router# debug snmp packet
```

#### 2. Trap 수신 안됨
```
문제: NMS에서 Trap을 받지 못함
원인:
- 잘못된 Trap 호스트 설정
- UDP 162 포트 차단
- Trap 미활성화

해결:
Router# show snmp host
Router# debug snmp packet
```

#### 3. 성능 문제
```
문제: SNMP 응답 속도 저하
원인:
- 과도한 폴링
- 복잡한 MIB 워크
- 높은 CPU 사용률

해결:
- 폴링 간격 조정
- Bulk 연산 사용
- 필요한 OID만 폴링
```

### 디버깅 명령어
```cisco
# SNMP 패킷 디버그
Router# debug snmp packet

# SNMP API 디버그
Router# debug snmp detail

# 특정 커뮤니티만
Router# debug snmp packet community monitor

# 디버그 종료
Router# undebug all
```

## 📋 SNMP 구현 모범 사례

### 설계 원칙
```
1. 보안 우선
   - SNMP v3 사용
   - 강력한 인증/암호화
   - 최소 권한 원칙

2. 성능 고려
   - 적절한 폴링 간격
   - 효율적인 MIB 선택
   - 네트워크 대역폭 고려

3. 관리 용이성
   - 표준 커뮤니티 명명 규칙
   - 문서화
   - 모니터링 정책
```

### 네트워크 분할 전략
```
관리 네트워크 분리:
- 전용 관리 VLAN
- Out-of-band 관리
- 방화벽 규칙 적용

SNMP 트래픽 제어:
- QoS 정책 적용
- 대역폭 제한
- 우선순위 설정
```

## 🔗 관련 주제
- [[네트워크 모니터링|모니터링]] - 네트워크 성능 모니터링
- [[네트워크 보안|보안]] - SNMP 보안 고려사항
- [[NTP|NTP]] - 시간 동기화와 로그 분석
- [[Syslog|Syslog]] - 로깅과 SNMP 연동

## 🏷️ 태그
#ccna #snmp #네트워크관리 #모니터링 #mib #trap #네트워크서비스 #보안 #nms