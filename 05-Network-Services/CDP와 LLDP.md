# CDP와 LLDP (Discovery Protocols)

## 📝 개요
CDP(Cisco Discovery Protocol)와 LLDP(Link Layer Discovery Protocol)는 인접 네트워크 장비를 자동으로 발견하고 정보를 교환하는 Layer 2 프로토콜입니다. 네트워크 토폴로지 파악과 문제해결에 필수적인 도구입니다.

## 🔍 CDP (Cisco Discovery Protocol)

### CDP 특징
- **벤더**: Cisco 독점 프로토콜
- **OSI 계층**: Layer 2 (Data Link)
- **사용 목적**: Cisco 장비 간 정보 교환
- **기본 상태**: 대부분 Cisco 장비에서 기본 활성화
- **전송 주기**: 60초마다 광고, 180초 홀드 타임

### CDP 정보 항목
```
교환되는 정보:
✅ 장비 ID (호스트명)
✅ 로컬 포트 식별자
✅ 플랫폼 (모델명)
✅ 기능 (라우터, 스위치, 폰 등)
✅ 소프트웨어 버전
✅ Native VLAN
✅ 듀플렉스 설정
✅ VTP 도메인
✅ 전력 소비 정보 (PoE)
```

### CDP 동작 원리
```
전송 방식:
- 멀티캐스트 주소: 0100.0ccc.cccc
- 모든 활성 인터페이스에서 전송
- SNAP 프레임 사용
- 라우팅되지 않음 (직접 연결된 장비만)

타이머:
- Advertisement Timer: 60초 (기본값)
- Hold Timer: 180초 (기본값)
- 홀드 타임 만료 시 정보 삭제
```

## 🌐 LLDP (Link Layer Discovery Protocol)

### LLDP 특징
- **표준**: IEEE 802.1AB
- **벤더**: 개방형 표준 (모든 벤더)
- **OSI 계층**: Layer 2 (Data Link)
- **사용 목적**: 멀티벤더 환경에서 장비 발견
- **기본 상태**: Cisco에서는 기본 비활성화

### LLDP vs CDP 비교
| 특징 | CDP | LLDP |
|------|-----|------|
| **표준** | Cisco 독점 | IEEE 802.1AB |
| **호환성** | Cisco만 | 모든 벤더 |
| **기본 상태** | 활성화 | 비활성화 |
| **전송 주기** | 60초 | 30초 |
| **홀드 타임** | 180초 | 120초 |
| **확장성** | 제한적 | TLV 확장 가능 |

## ⚙️ CDP 설정 및 관리

### CDP 기본 설정
```cisco
# CDP 전역 활성화/비활성화
Router(config)# cdp run
Router(config)# no cdp run

# 인터페이스별 CDP 제어
Router(config)# interface gi0/1
Router(config-if)# cdp enable
Router(config-if)# no cdp enable

# CDP 타이머 조정
Router(config)# cdp timer 30        # 30초마다 광고
Router(config)# cdp holdtime 90     # 90초 홀드 타임
```

### CDP 보안 고려사항
```cisco
# 보안이 중요한 인터페이스에서 CDP 비활성화
Router(config)# interface serial0/0/0  # WAN 인터페이스
Router(config-if)# no cdp enable

# 외부 연결 포트에서 CDP 비활성화
Switch(config)# interface range gi0/1-5  # DMZ 연결 포트
Switch(config-if-range)# no cdp enable
```

## ⚙️ LLDP 설정 및 관리

### LLDP 기본 설정
```cisco
# LLDP 전역 활성화
Switch(config)# lldp run

# 인터페이스별 LLDP 제어
Switch(config)# interface gi0/1
Switch(config-if)# lldp transmit    # 전송만 활성화
Switch(config-if)# lldp receive     # 수신만 활성화
Switch(config-if)# no lldp transmit # 전송 비활성화
Switch(config-if)# no lldp receive  # 수신 비활성화

# LLDP 타이머 조정
Switch(config)# lldp timer 60           # 60초마다 광고
Switch(config)# lldp holdtime 240       # 240초 홀드 타임
Switch(config)# lldp reinit 3           # 재초기화 지연
```

### LLDP TLV 설정
```cisco
# 선택적 TLV 활성화/비활성화
Switch(config-if)# lldp tlv-select port-description
Switch(config-if)# lldp tlv-select system-name
Switch(config-if)# lldp tlv-select system-description
Switch(config-if)# lldp tlv-select system-capabilities

# 특정 TLV 비활성화
Switch(config-if)# no lldp tlv-select management-address
```

## 📊 정보 확인 명령어

### CDP 확인 명령어
```cisco
# 기본 CDP 정보
Router# show cdp

# 인접 장비 요약 정보
Router# show cdp neighbors

# 인접 장비 상세 정보
Router# show cdp neighbors detail

# 특정 인터페이스 CDP 정보
Router# show cdp interface gi0/1

# CDP 트래픽 통계
Router# show cdp traffic

# CDP 항목 개수
Router# show cdp entry *
```

### CDP 출력 예시
```
Router# show cdp neighbors
Capability Codes: R - Router, T - Trans Bridge, B - Source Route Bridge
                  S - Switch, H - Host, I - IGMP, r - Repeater

Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
SW1.company.com  Gig 0/1           120          S I       WS-C2960  Gig 0/24
R2.company.com   Ser 0/0/0         150          R         1841      Ser 0/0/1
IP_Phone_001     Fas 0/5           160          H P       CP-7961G  Port 1

능력 코드 해석:
R = 라우터, S = 스위치, H = 호스트
I = IGMP, P = 전화기, T = 투명 브리지
```

### LLDP 확인 명령어
```cisco
# LLDP 상태 확인
Switch# show lldp

# 인접 장비 요약 정보
Switch# show lldp neighbors

# 인접 장비 상세 정보
Switch# show lldp neighbors detail

# 특정 인터페이스 LLDP 정보
Switch# show lldp interface gi0/1

# LLDP 트래픽 통계
Switch# show lldp traffic

# 로컬 LLDP 정보
Switch# show lldp local-information
```

### LLDP 출력 예시
```
Switch# show lldp neighbors
Capability codes:
    (R) Router, (B) Bridge, (T) Telephone, (C) DOCSIS Cable Device
    (W) WLAN Access Point, (P) Repeater, (S) Station, (O) Other

Device ID           Local Intf     Hold-time  Capability      Port ID
SW2.domain.com      Gi0/1          120        B,R             Gi0/2
Phone-12345         Fa0/10         120        T               0
Server-01           Gi0/5          120        S               eth0

Total entries displayed: 3
```

## 🔧 문제해결 활용

### 네트워크 토폴로지 파악
```cisco
# 전체 네트워크 맵핑
Router# show cdp neighbors detail | include Device ID|IP address|Platform

출력 활용:
- 물리적 연결 관계 파악
- 장비 유형 및 모델 확인
- IP 주소 매핑
- 소프트웨어 버전 일관성 확인
```

### 연결 문제 진단
```cisco
# 인터페이스별 상태 확인
Router# show cdp interface

출력 정보:
- CDP 활성화 상태
- 캡슐화 타입
- 전송/수신 통계
- 오류 정보

문제 진단:
- CDP가 비활성화되어 있는지 확인
- 인터페이스 상태 확인
- 케이블 연결 상태 추정
```

### 듀플렉스 불일치 감지
```cisco
Router# show cdp neighbors detail | include Duplex

일반적인 문제:
- Half/Full duplex 불일치
- 자동/수동 협상 불일치
- 속도 설정 불일치

해결 방법:
Router(config-if)# duplex full
Router(config-if)# speed 100
```

## 🛡️ 보안 고려사항

### CDP 보안 위험
```
위험 요소:
❌ 네트워크 토폴로지 노출
❌ 장비 정보 유출 (모델, 버전)
❌ IP 주소 정보 노출
❌ VLAN 정보 노출
❌ 잠재적 공격 벡터 제공

보안 대책:
✅ 외부 연결에서 CDP 비활성화
✅ 불필요한 인터페이스에서 비활성화
✅ 정기적인 CDP 설정 검토
✅ 네트워크 경계에서 차단
```

### 보안 강화 방법
```cisco
# DMZ 및 외부 연결에서 CDP 비활성화
Router(config)# interface gi0/0  # Internet 연결
Router(config-if)# no cdp enable

Router(config)# interface gi0/1  # DMZ 연결
Router(config-if)# no cdp enable

# 게스트 네트워크에서 비활성화
Switch(config)# interface range fa0/20-24  # 게스트 포트
Switch(config-if-range)# no cdp enable

# 중요 서버 연결에서 비활성화 (선택적)
Switch(config)# interface gi0/10  # 중요 서버
Switch(config-if)# no cdp enable
```

## 📋 모범 사례

### 설정 권장사항
```
1. 내부 네트워크
   ✅ CDP/LLDP 활성화 (관리 편의성)
   ✅ 적절한 타이머 설정
   ✅ 정기적인 정보 검토

2. 외부 연결
   ✅ CDP/LLDP 비활성화
   ✅ 보안 정책 적용
   ✅ 모니터링 강화

3. DMZ/서버 팜
   ✅ 제한적 활성화
   ✅ 선택적 정보 공유
   ✅ 접근 제어 적용
```

### 운영 가이드라인
```
1. 문서화
   - CDP/LLDP 활성화 정책 수립
   - 네트워크 다이어그램 자동 업데이트
   - 장비 인벤토리 관리

2. 모니터링
   - 비정상적인 neighbor 출현 감지
   - 토폴로지 변경 추적
   - 보안 이벤트 연계

3. 유지보수
   - 정기적인 neighbor 정보 검토
   - 오래된 정보 정리
   - 설정 일관성 확인
```

## 🔧 고급 활용

### 네트워크 자동화
```python
# Python으로 CDP 정보 수집 예시
import netmiko

def get_cdp_neighbors(device_ip, username, password):
    device = {
        'device_type': 'cisco_ios',
        'ip': device_ip,
        'username': username,
        'password': password,
    }
    
    connection = netmiko.ConnectHandler(**device)
    output = connection.send_command('show cdp neighbors detail')
    connection.disconnect()
    
    return parse_cdp_output(output)
```

### 네트워크 매핑 도구
```cisco
# 네트워크 전체 매핑을 위한 정보 수집
show cdp neighbors detail | include Device ID|IP address|Platform|Interface

활용:
- 자동 네트워크 다이어그램 생성
- 장비 인벤토리 자동 업데이트
- 연결 상태 모니터링
- 변경 사항 추적
```

## 🎯 시험 포인트 (CCNA)

### 주요 출제 영역
```
1. CDP 기본 개념
   - 동작 원리
   - 정보 항목
   - 타이머 설정

2. 명령어 활용
   - show cdp neighbors
   - show cdp neighbors detail
   - cdp enable/disable

3. 보안 고려사항
   - 언제 비활성화해야 하는가
   - 보안 위험 요소

4. LLDP 대안
   - CDP vs LLDP 차이점
   - 멀티벤더 환경에서의 선택
```

### 자주 나오는 문제 유형
```
1. 출력 해석 문제
   - CDP neighbor 정보 분석
   - 토폴로지 추정
   - 연결 상태 판단

2. 설정 명령어
   - CDP 활성화/비활성화
   - 타이머 조정
   - 인터페이스별 제어

3. 문제해결 시나리오
   - CDP 정보를 활용한 연결 문제 진단
   - 듀플렉스 불일치 해결
   - 보안 설정 적용
```

## 🔗 관련 주제
- [[네트워크 토폴로지|토폴로지]] - 물리적/논리적 네트워크 구조
- [[네트워크 디바이스|장비]] - 라우터, 스위치 등 네트워크 장비
- [[문제해결 방법론|문제해결]] - 네트워크 문제해결 접근법
- [[VLAN 구성|VLAN]] - Native VLAN 및 트렁킹 정보

## 🏷️ 태그
#ccna #cdp #lldp #discovery-protocol #네트워크관리 #토폴로지 #neighbor #시스코