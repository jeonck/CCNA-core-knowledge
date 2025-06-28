# DHCP (Dynamic Host Configuration Protocol)

## 개요
DHCP는 네트워크 장치에 IP 주소와 네트워크 설정을 자동으로 할당하는 프로토콜입니다. 중앙 집중식 IP 관리를 통해 네트워크 관리를 간소화합니다.

## DHCP 동작 과정 (DORA)

### 4단계 프로세스
1. **DISCOVER**: 클라이언트가 DHCP 서버 찾기 (브로드캐스트)
2. **OFFER**: DHCP 서버가 IP 주소 제안
3. **REQUEST**: 클라이언트가 IP 주소 요청
4. **ACK**: DHCP 서버가 IP 주소 할당 확인

```
Client                    DHCP Server
  │                           │
  │ DHCP DISCOVER (broadcast) │
  ├──────────────────────────►│
  │                           │
  │      DHCP OFFER           │
  │◄──────────────────────────┤
  │                           │
  │     DHCP REQUEST          │
  ├──────────────────────────►│
  │                           │
  │       DHCP ACK            │
  │◄──────────────────────────┤
```

## DHCP 서버 설정 (Cisco)

### 기본 설정
```cisco
# DHCP 풀 생성
Router(config)# ip dhcp pool LAN_POOL
Router(dhcp-config)# network 192.168.1.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.1.1
Router(dhcp-config)# dns-server 8.8.8.8 8.8.4.4
Router(dhcp-config)# domain-name company.com
Router(dhcp-config)# lease 7

# 제외할 주소 범위 설정
Router(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10
```

### 고급 설정
```cisco
# 특정 MAC 주소에 고정 IP 할당
Router(config)# ip dhcp pool PRINTER
Router(dhcp-config)# host 192.168.1.100 255.255.255.0
Router(dhcp-config)# client-identifier 01aa.bbcc.ddee.ff
Router(dhcp-config)# default-router 192.168.1.1
Router(dhcp-config)# dns-server 8.8.8.8

# Option 43 설정 (무선 컨트롤러 정보)
Router(dhcp-config)# option 43 hex f104c0a80a0a
```

## DHCP 릴레이 (IP Helper)

### 필요성
DHCP는 브로드캐스트를 사용하므로 다른 서브넷의 클라이언트는 서버를 찾을 수 없습니다.

### 설정
```cisco
# 인터페이스에서 DHCP 릴레이 설정
Router(config)# interface fastethernet 0/1
Router(config-if)# ip helper-address 192.168.10.100

# 여러 DHCP 서버 설정 가능
Router(config-if)# ip helper-address 192.168.10.101
```

## DHCP 옵션

### 주요 옵션
| 옵션 | 설명 | 예시 |
|------|------|------|
| 1 | 서브넷 마스크 | 255.255.255.0 |
| 3 | 기본 게이트웨이 | 192.168.1.1 |
| 6 | DNS 서버 | 8.8.8.8 |
| 15 | 도메인 이름 | company.com |
| 42 | NTP 서버 | 192.168.1.10 |
| 43 | 벤더별 정보 | 무선 컨트롤러 IP |
| 66 | TFTP 서버 | 192.168.1.20 |
| 67 | 부트 파일명 | cisco.bin |

## DHCP 확인 및 문제해결

### 확인 명령어
```cisco
# DHCP 바인딩 확인
Router# show ip dhcp binding

# DHCP 충돌 확인
Router# show ip dhcp conflict

# DHCP 통계
Router# show ip dhcp statistics

# DHCP 서버 상태
Router# show ip dhcp server statistics
```

### 클라이언트 명령어
```bash
# Windows
ipconfig /all
ipconfig /release
ipconfig /renew

# Linux
dhclient -r  # release
dhclient     # renew
```

## DHCP 보안

### DHCP Snooping
스위치에서 DHCP 메시지를 검사하여 비인가 DHCP 서버를 차단합니다.

```cisco
# DHCP Snooping 활성화
Switch(config)# ip dhcp snooping
Switch(config)# ip dhcp snooping vlan 10

# 신뢰할 수 있는 포트 설정
Switch(config)# interface fastethernet 0/24
Switch(config-if)# ip dhcp snooping trust

# 속도 제한 설정
Switch(config)# interface range fastethernet 0/1-23
Switch(config-if-range)# ip dhcp snooping limit rate 10
```

### Option 82 (DHCP Relay Agent Information)
```cisco
# Option 82 활성화
Switch(config)# ip dhcp snooping information option
```

## DHCPv6

### Stateful vs Stateless
```
Stateful DHCPv6:
- IPv4 DHCP와 유사
- 서버가 주소 할당 관리
- 주소, 설정 정보 모두 제공

Stateless DHCPv6:  
- SLAAC와 함께 사용
- 주소는 자동 생성
- 설정 정보만 제공 (DNS 등)
```

### 설정 예시
```cisco
# DHCPv6 풀 설정
Router(config)# ipv6 dhcp pool POOL1
Router(config-dhcpv6)# address prefix 2001:db8:1::/64
Router(config-dhcpv6)# dns-server 2001:4860:4860::8888

# 인터페이스 설정
Router(config)# interface fastethernet 0/0
Router(config-if)# ipv6 dhcp server POOL1
```

## DHCP 모범 사례

### 설계 권장사항
1. **중복성**: 여러 DHCP 서버 구성 (80/20 규칙)
2. **모니터링**: 주소 풀 사용률 감시
3. **보안**: DHCP Snooping 활용
4. **문서화**: IP 할당 정책 문서화

### 주소 할당 전략
```
고정 할당: 서버, 프린터, 네트워크 장비
동적 할당: 클라이언트 PC, 모바일 기기

예약 범위:
1-50:    네트워크 장비
51-100:  서버
101-200: 고정 할당
201-254: 동적 할당
```

## 관련 주제
- [[DNS]] - 도메인 이름 해석
- [[NAT-PAT]] - 사설 IP 주소 변환
- [[무선 네트워킹]] - 무선 클라이언트 설정
- [[네트워크 토폴로지]] - 네트워크 구조

## 태그
#ccna #dhcp #네트워크서비스 #ip할당 #자동화
