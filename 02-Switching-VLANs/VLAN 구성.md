# VLAN 구성

## 개요
VLAN(Virtual Local Area Network)은 물리적 위치와 관계없이 논리적으로 분리된 네트워크를 만드는 기술입니다. [[이더넷 기초]]를 기반으로 하여 브로드캐스트 도메인을 분할하고 보안을 강화합니다.

## VLAN의 장점

### 보안 강화
- 브로드캐스트 도메인 분리
- 민감한 데이터 격리
- 액세스 제어 향상

### 성능 향상
- 브로드캐스트 트래픽 감소
- 네트워크 세분화
- 대역폭 효율성 증대

### 관리 편의성
- 논리적 그룹화
- 중앙 집중식 관리
- 유연한 네트워크 설계

### 비용 절감
- 물리적 재배치 불필요
- 케이블링 비용 절약
- 장비 활용도 향상

## VLAN 유형

### Data VLAN
```
용도: 사용자 데이터 트래픽
특징: 일반적인 사용자 트래픽 전송
범위: VLAN ID 1-4094 (2-1001 권장)

예시:
VLAN 10 - Sales Department
VLAN 20 - Engineering Department  
VLAN 30 - Management
```

### Voice VLAN
```
용도: VoIP 트래픽 전용
특징: QoS 우선순위 적용
장점: 음성 품질 보장

설정 예시:
switchport voice vlan 150
```

### Management VLAN
```
용도: 스위치 관리
기본값: VLAN 1 (변경 권장)
보안: 별도 VLAN 사용 권장

설정 예시:
interface vlan 99
ip address 192.168.99.10 255.255.255.0
```

### Native VLAN
```
용도: 트렁크에서 태그 없는 트래픽
기본값: VLAN 1
보안: 변경 권장 (VLAN 99 등)

설정:
switchport trunk native vlan 99
```

### Default VLAN
```
VLAN 1: 모든 포트의 기본 VLAN
특징: 삭제 불가, 이름 변경 불가
권장: 사용하지 않기
```

## VLAN 설정

### 기본 VLAN 생성
```cisco
# VLAN 생성 방법 1: 글로벌 설정 모드
Switch(config)# vlan 10
Switch(config-vlan)# name SALES
Switch(config-vlan)# exit

Switch(config)# vlan 20  
Switch(config-vlan)# name ENGINEERING
Switch(config-vlan)# exit

# VLAN 생성 방법 2: 데이터베이스 모드 (구버전)
Switch# vlan database
Switch(vlan)# vlan 10 name SALES
Switch(vlan)# vlan 20 name ENGINEERING
Switch(vlan)# exit
```

### 포트 할당
```cisco
# 액세스 포트 설정
Switch(config)# interface fastethernet 0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# description "PC in Sales Department"

# 여러 포트 동시 설정
Switch(config)# interface range fastethernet 0/1-5
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 10

# 음성 VLAN 설정
Switch(config)# interface fastethernet 0/6
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# switchport voice vlan 150
```

## 트렁킹 (Trunking)

### 트렁킹 개념
```
목적: 여러 VLAN 트래픽을 하나의 링크로 전송
방법: VLAN 태깅 (802.1Q)
용도: 스위치 간 연결, 스위치-라우터 연결
```

### 802.1Q 태깅
```
프레임 구조:
┌──────────┬─────────┬────┬──────┬────────┬─────┬─────┐
│ Dest MAC │ Src MAC │TPID│ TCI  │ Type   │Data │ FCS │
└──────────┴─────────┴────┴──────┴────────┴─────┴─────┘
                      │    │
                      │    └─ VLAN ID (12bit)
                      └─ Tag Protocol ID (0x8100)

TPID: Tag Protocol Identifier (2 bytes)
TCI: Tag Control Information (2 bytes)
  ├─ Priority (3 bits): QoS 우선순위
  ├─ CFI (1 bit): Canonical Format Indicator  
  └─ VLAN ID (12 bits): VLAN 식별자
```

### 트렁크 설정
```cisco
# 기본 트렁크 설정
Switch(config)# interface fastethernet 0/24
Switch(config-if)# switchport mode trunk

# 허용할 VLAN 지정
Switch(config-if)# switchport trunk allowed vlan 10,20,30

# Native VLAN 변경
Switch(config-if)# switchport trunk native vlan 99

# 트렁크 캡슐화 방식 설정 (필요시)
Switch(config-if)# switchport trunk encapsulation dot1q
```

### DTP (Dynamic Trunking Protocol)
```cisco
# DTP 모드
switchport mode access         # 항상 액세스
switchport mode trunk          # 항상 트렁크
switchport mode dynamic auto   # 상대방이 trunk이면 trunk
switchport mode dynamic desirable # 트렁크 협상 시도

# DTP 비활성화 (보안상 권장)
switchport nonegotiate
```

## VTP (VLAN Trunking Protocol)

### VTP 개념
```
목적: VLAN 정보를 여러 스위치에 자동 전파
장점: 중앙 집중식 VLAN 관리
단점: 잘못된 설정 시 전체 영향
```

### VTP 모드
```
Server Mode:
- VLAN 생성/수정/삭제 가능
- VTP 정보를 다른 스위치에 전파
- VLAN 정보를 NVRAM에 저장

Client Mode:  
- VLAN 정보 수신만 가능
- VLAN 생성/수정/삭제 불가
- VLAN 정보를 RAM에만 저장

Transparent Mode:
- VTP 메시지를 포워딩만 함
- 로컬 VLAN 설정 독립적으로 가능
- 다른 스위치에 영향 주지 않음

Off Mode:
- VTP 기능 완전 비활성화
- VTP 메시지 처리하지 않음
```

### VTP 설정
```cisco
# VTP 도메인 및 모드 설정
Switch(config)# vtp domain COMPANY
Switch(config)# vtp mode server
Switch(config)# vtp password cisco123
Switch(config)# vtp version 2

# VTP 상태 확인
Switch# show vtp status
Switch# show vtp counters
```

### VTP 주의사항
```
위험요소:
1. 높은 Configuration Revision으로 인한 VLAN 삭제
2. 잘못된 도메인 이름 설정
3. 패스워드 불일치

예방책:
1. Transparent 모드 사용 권장
2. 신중한 서버 모드 사용
3. 백업 설정 유지
```

## Inter-VLAN 라우팅

### Router-on-a-Stick
```cisco
# 라우터 설정
Router(config)# interface fastethernet 0/0
Router(config-if)# no shutdown

# 서브인터페이스 설정
Router(config)# interface fastethernet 0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0

Router(config)# interface fastethernet 0/0.20
Router(config-subif)# encapsulation dot1Q 20  
Router(config-subif)# ip address 192.168.20.1 255.255.255.0

# 스위치 설정
Switch(config)# interface fastethernet 0/24
Switch(config-if)# switchport mode trunk
```

### SVI (Switched Virtual Interface)
```cisco
# Layer 3 스위치에서 Inter-VLAN 라우팅
Switch(config)# ip routing

# VLAN 인터페이스 생성
Switch(config)# interface vlan 10
Switch(config-if)# ip address 192.168.10.1 255.255.255.0
Switch(config-if)# no shutdown

Switch(config)# interface vlan 20
Switch(config-if)# ip address 192.168.20.1 255.255.255.0
Switch(config-if)# no shutdown
```

## VLAN 확인 및 문제해결

### 확인 명령어
```cisco
# VLAN 정보 확인
Switch# show vlan brief
Switch# show vlan id 10
Switch# show vlan name SALES

# 인터페이스별 VLAN 확인
Switch# show interfaces fastethernet 0/1 switchport
Switch# show interfaces trunk

# MAC 주소 테이블
Switch# show mac address-table
Switch# show mac address-table vlan 10

# VTP 상태
Switch# show vtp status
Switch# show vtp counters
```

### 일반적인 문제
```
1. VLAN 할당 오류
   증상: 같은 VLAN 내 통신 안됨
   확인: show interfaces switchport
   해결: 올바른 VLAN 할당

2. 트렁크 설정 오류
   증상: Inter-VLAN 통신 안됨
   확인: show interfaces trunk
   해결: 트렁크 모드 및 허용 VLAN 확인

3. Native VLAN 불일치
   증상: CDP 경고 메시지
   확인: show interfaces trunk
   해결: 양쪽 Native VLAN 통일

4. VTP 동기화 문제
   증상: VLAN 정보 불일치
   확인: show vtp status
   해결: 도메인, 패스워드, 모드 확인
```

### 디버깅
```cisco
# 트렁크 관련 디버그
Switch# debug spanning-tree events
Switch# debug dtp

# VTP 관련 디버그  
Switch# debug vtp events
Switch# debug vtp packets
```

## VLAN 보안

### VLAN 호핑 공격
```
공격 방법:
1. Switch Spoofing: 트렁크 모드로 위장
2. Double Tagging: 이중 VLAN 태깅

방어 방법:
1. 사용하지 않는 포트 비활성화
2. DTP 비활성화 (nonegotiate)
3. Native VLAN을 사용하지 않는 VLAN으로 변경
4. 액세스 포트 명시적 설정
```

### 보안 모범 사례
```cisco
# 사용하지 않는 포트 설정
Switch(config)# interface range fastethernet 0/10-23
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 999
Switch(config-if-range)# shutdown

# DTP 비활성화
Switch(config)# interface fastethernet 0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport nonegotiate

# 트렁크 보안 강화
Switch(config)# interface fastethernet 0/24
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk native vlan 999
Switch(config-if)# switchport trunk allowed vlan 10,20,30
Switch(config-if)# switchport nonegotiate
```

## VLAN 설계 모범 사례

### 설계 원칙
```
1. 기능별 분리
   - 부서별 VLAN
   - 서버 VLAN
   - 관리 VLAN
   - 게스트 VLAN

2. 보안 고려
   - 민감한 데이터 격리
   - DMZ 구성
   - 관리 트래픽 분리

3. 성능 최적화
   - 브로드캐스트 도메인 크기 제한
   - 적절한 VLAN 개수
   - QoS 정책 적용

4. 확장성 고려
   - 미래 확장 계획
   - VLAN 번호 체계
   - 서브넷 설계
```

### VLAN 번호 체계 예시
```
VLAN 10-19: Sales Department
VLAN 20-29: Engineering Department
VLAN 30-39: Management
VLAN 40-49: Marketing
VLAN 50-59: Guest Networks
VLAN 60-69: Servers
VLAN 70-79: Infrastructure
VLAN 80-89: Voice (VoIP)
VLAN 90-99: Management/Native
```

## 관련 주제
- [[이더넷 기초]] - VLAN의 기반 기술
- [[스위치 동작원리]] - 스위칭과 VLAN
- [[STP]] - VLAN과 스패닝 트리
- [[라우팅 기초]] - Inter-VLAN 라우팅

## 태그
#ccna #vlan #스위칭 #네트워크분할 #트렁킹
