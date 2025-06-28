# VPN (Virtual Private Network)

## 개요
VPN은 공중 네트워크(인터넷)를 통해 사설 네트워크를 확장하는 기술입니다. 암호화와 터널링을 사용하여 안전하고 비용 효율적인 원격 연결을 제공합니다.

## VPN의 필요성

### 전통적인 WAN의 한계
```
전용선 (Leased Line):
❌ 높은 비용
❌ 구축 시간 오래 걸림
❌ 지리적 제약
❌ 확장성 부족
❌ 유연성 부족

Frame Relay:
❌ 복잡한 설정
❌ 제한된 대역폭
❌ 벤더 종속성
❌ 레거시 기술
```

### VPN의 장점
```
✅ 비용 효율성 (인터넷 활용)
✅ 빠른 배포
✅ 지리적 제약 없음
✅ 확장성 우수
✅ 유연한 접근 제어
✅ 강력한 보안 (암호화)
✅ 원격 근무 지원
✅ 다양한 연결 방식
```

## VPN 유형

### 연결 방식에 따른 분류

#### Site-to-Site VPN
```
특징:
- 지사 간 고정 연결
- 라우터/방화벽 간 터널
- 항상 연결 상태 유지
- 사용자 개입 최소

용도:
- 본사-지사 연결
- 지사 간 연결
- 파트너사와의 연결
- 데이터센터 백업

장점:
✅ 투명한 연결 (사용자가 인식 불필요)
✅ 자동 연결
✅ 고성능
✅ 중앙 관리

단점:
❌ 고정비용
❌ 항상 대역폭 점유
❌ 복잡한 설정
```

#### Remote Access VPN
```
특징:
- 개별 사용자 연결
- 클라이언트 소프트웨어 필요
- 필요 시에만 연결
- 동적 IP 할당

용도:
- 재택근무
- 출장 중 회사 접속
- 외부 협력업체 접근
- 모바일 근무

장점:
✅ 유연한 접근
✅ 사용한 만큼 과금
✅ 개별 보안 정책
✅ 간단한 클라이언트

단점:
❌ 클라이언트 설치 필요
❌ 사용자 교육 필요
❌ 성능 제한
❌ 관리 복잡성
```

### 구현 방식에 따른 분류

#### IPSec VPN
```
특징:
- IP 계층에서 동작
- 강력한 보안
- 표준 프로토콜
- 다양한 벤더 지원

보안 서비스:
- 인증 (Authentication)
- 무결성 (Integrity)
- 기밀성 (Confidentiality)
- 재전송 방지 (Anti-replay)

프로토콜:
- ESP (Encapsulating Security Payload)
- AH (Authentication Header)
- IKE (Internet Key Exchange)
```

#### SSL/TLS VPN
```
특징:
- 애플리케이션 계층에서 동작
- 웹 브라우저 기반
- 클라이언트 설치 최소화
- 사용 편의성 우수

방식:
- Clientless (웹 브라우저만)
- Thin Client (작은 플러그인)
- Thick Client (전용 클라이언트)

장점:
✅ 쉬운 사용
✅ 플랫폼 독립적
✅ NAT 친화적
✅ 세밀한 접근 제어
```

#### MPLS VPN
```
특징:
- 서비스 제공자 네트워크 기반
- Layer 3 VPN 서비스
- 고성능 및 QoS 보장
- 관리 서비스

장점:
✅ 고성능
✅ QoS 보장
✅ 관리 편의성
✅ 확장성

단점:
❌ 높은 비용
❌ 벤더 종속성
❌ 지리적 제약
```

## IPSec VPN 상세

### IPSec 구성 요소

#### 보안 프로토콜
```
ESP (Encapsulating Security Payload):
- 데이터 암호화 및 인증
- 기밀성과 무결성 동시 제공
- Transport 모드와 Tunnel 모드 지원

AH (Authentication Header):
- 인증 및 무결성만 제공
- 암호화 없음 (기밀성 없음)
- ESP보다 오버헤드 적음
```

#### 동작 모드
```
Transport Mode:
- 원본 IP 헤더 유지
- 페이로드만 보호
- 호스트 간 통신에 적합
- 오버헤드 적음

구조: [IP Header][ESP/AH Header][Original Payload]

Tunnel Mode:
- 전체 IP 패킷 보호
- 새로운 IP 헤더 추가
- 게이트웨이 간 통신에 적합
- 내부 네트워크 주소 숨김

구조: [New IP Header][ESP/AH Header][Original IP Packet]
```

#### 암호화 알고리즘
```
대칭키 암호화:
- DES (56비트) - 취약, 사용 금지
- 3DES (168비트) - 레거시
- AES (128/192/256비트) - 권장

해시 알고리즘:
- MD5 (128비트) - 취약, 사용 금지
- SHA-1 (160비트) - 레거시
- SHA-256/384/512 - 권장

Diffie-Hellman 그룹:
- Group 1 (768비트) - 취약
- Group 2 (1024비트) - 최소
- Group 14 (2048비트) - 권장
- Group 19-21 (ECC) - 최신
```

### IKE (Internet Key Exchange)

#### IKE 버전
```
IKEv1:
- 2단계 협상 (Phase 1, Phase 2)
- Main Mode / Aggressive Mode
- 복잡한 설정
- 레거시 시스템과 호환

IKEv2:
- 단순화된 협상
- 개선된 성능
- NAT 지원 강화
- 모바일 지원
- 권장 버전
```

#### IKE Phase 1 (ISAKMP SA)
```
목적: 안전한 통신 채널 구성
협상 내용:
- 암호화 알고리즘
- 해시 알고리즘
- 인증 방법
- DH 그룹
- SA 수명

인증 방법:
- Pre-shared Key (PSK)
- RSA 서명
- RSA 암호화
- XAUTH (확장 인증)
```

#### IKE Phase 2 (IPSec SA)
```
목적: 실제 데이터 보호 설정
협상 내용:
- ESP/AH 프로토콜
- 암호화 알고리즘
- 인증 알고리즘
- PFS (Perfect Forward Secrecy)
- SA 수명

결과:
- IPSec SA 생성
- 암호화 키 교환
- 터널 설정 완료
```

## Site-to-Site VPN 설정 (Cisco)

### 기본 설정 단계
```
1. IKE Policy 설정 (Phase 1)
2. Pre-shared Key 설정
3. IPSec Transform Set 설정 (Phase 2)
4. 암호화 트래픽 정의 (ACL)
5. Crypto Map 설정
6. 인터페이스에 Crypto Map 적용
```

### 상세 설정 예시
```cisco
# 1단계: IKE Policy (Phase 1)
R1(config)# crypto isakmp policy 10
R1(config-isakmp)# encryption aes 256
R1(config-isakmp)# hash sha256
R1(config-isakmp)# authentication pre-share
R1(config-isakmp)# group 14
R1(config-isakmp)# lifetime 3600

# 2단계: Pre-shared Key
R1(config)# crypto isakmp key cisco123 address 203.0.113.2

# 3단계: Transform Set (Phase 2)
R1(config)# crypto ipsec transform-set STRONG_SET esp-aes 256 esp-sha256-hmac
R1(config-transform)# mode tunnel

# 4단계: 암호화 트래픽 ACL
R1(config)# access-list 101 permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255

# 5단계: Crypto Map
R1(config)# crypto map VPN_MAP 10 ipsec-isakmp
R1(config-crypto-map)# set peer 203.0.113.2
R1(config-crypto-map)# set transform-set STRONG_SET
R1(config-crypto-map)# match address 101
R1(config-crypto-map)# set pfs group14

# 6단계: 인터페이스 적용
R1(config)# interface serial 0/0/0
R1(config-if)# crypto map VPN_MAP
```

### 고급 설정
```cisco
# Dead Peer Detection (DPD)
R1(config)# crypto isakmp keepalive 10 3

# NAT Traversal 지원
R1(config)# crypto isakmp nat traversal

# 여러 Transform Set 지원
R1(config-crypto-map)# set transform-set STRONG_SET MEDIUM_SET

# 백업 피어 설정
R1(config-crypto-map)# set peer 203.0.113.2
R1(config-crypto-map)# set peer 203.0.113.3

# 터널 보호 (GRE over IPSec)
R1(config)# interface tunnel 0
R1(config-if)# tunnel protection ipsec profile VPN_PROFILE
```

## Remote Access VPN

### SSL VPN (AnyConnect)
```cisco
# ASA에서 SSL VPN 설정
ASA(config)# webvpn
ASA(config-webvpn)# enable outside
ASA(config-webvpn)# anyconnect image disk0:/anyconnect-linux-4.8.0-k9.pkg
ASA(config-webvpn)# anyconnect enable

# IP 풀 설정
ASA(config)# ip local pool VPN_POOL 192.168.100.1-192.168.100.100 mask 255.255.255.0

# 그룹 정책
ASA(config)# group-policy ANYCONNECT_POLICY internal
ASA(config-group-policy)# vpn-tunnel-protocol ssl-client
ASA(config-group-policy)# split-tunnel-policy tunnelspecified
ASA(config-group-policy)# split-tunnel-network-list SPLIT_TUNNEL_ACL
ASA(config-group-policy)# address-pools VPN_POOL

# 사용자 계정
ASA(config)# username john password cisco123
ASA(config)# username john attributes
ASA(config-username)# vpn-group-policy ANYCONNECT_POLICY
```

### IPSec Remote Access (IKEv2)
```cisco
# IKEv2 설정
R1(config)# crypto ikev2 proposal AES/GCM/256
R1(config-ikev2-proposal)# encryption aes-gcm-256
R1(config-ikev2-proposal)# prf sha384
R1(config-ikev2-proposal)# group 19

# IKEv2 Policy
R1(config)# crypto ikev2 policy FLEX_POLICY
R1(config-ikev2-policy)# proposal AES/GCM/256

# 인증 방법
R1(config)# crypto ikev2 keyring FLEX_KEYRING
R1(config-ikev2-keyring)# peer ANY
R1(config-ikev2-keyring-peer)# address 0.0.0.0 0.0.0.0
R1(config-ikev2-keyring-peer)# pre-shared-key cisco123
```

## VPN 문제해결

### 일반적인 문제
```
1. Phase 1 협상 실패
   원인:
   - IKE Policy 불일치
   - Pre-shared Key 불일치
   - 방화벽 차단 (UDP 500, 4500)
   - NAT 문제

   해결:
   - 양쪽 설정 비교
   - 키 정확성 확인
   - 방화벽 규칙 점검
   - NAT Traversal 활성화

2. Phase 2 협상 실패
   원인:
   - Transform Set 불일치
   - 암호화 ACL 불일치
   - Proxy ID 불일치

   해결:
   - Transform Set 통일
   - ACL 미러링 확인
   - 트래픽 플로우 점검

3. 터널 연결되지만 통신 안됨
   원인:
   - 라우팅 문제
   - 방화벽 정책
   - NAT 설정 충돌

   해결:
   - 라우팅 테이블 확인
   - End-to-end 연결성 테스트
   - NAT 예외 설정
```

### 디버깅 명령어
```cisco
# IKE 디버그
Router# debug crypto isakmp
Router# debug crypto ipsec

# 상세 디버그
Router# debug crypto isakmp sa
Router# debug crypto ipsec sa

# 특정 피어만 디버그
Router# debug crypto condition peer ipv4 203.0.113.2

# 패킷 레벨 디버그
Router# debug crypto pki packets
```

### 상태 확인 명령어
```cisco
# IKE SA 상태
Router# show crypto isakmp sa
Router# show crypto ikev2 sa

# IPSec SA 상태
Router# show crypto ipsec sa

# Crypto Map 확인
Router# show crypto map

# VPN 통계
Router# show crypto ipsec statistics

# 터널 인터페이스 상태
Router# show interface tunnel 0
```

## VPN 성능 최적화

### 하드웨어 가속
```
VPN 하드웨어 가속:
- 전용 암호화 칩 사용
- CPU 부하 감소
- 처리량 대폭 향상
- 지연시간 감소

확인 방법:
Router# show crypto engine accelerator statistic
```

### 튜닝 매개변수
```cisco
# MTU 크기 조정
Router(config-if)# ip mtu 1436
Router(config-if)# ip tcp adjust-mss 1396

# 터널 keepalive
Router(config-if)# keepalive 10 3

# QoS 적용
Router(config)# policy-map VPN_QOS
Router(config-pmap)# class VOICE
Router(config-pmap-c)# priority percent 30
```

### 대역폭 최적화
```
압축 기능:
- LZS 압축 지원
- 반복 데이터 제거
- 대역폭 효율성 향상

Router(config-crypto-map)# set compression lzs
```

## VPN 보안 모범 사례

### 강력한 암호화
```
권장 설정:
- 암호화: AES-256
- 해시: SHA-256 이상
- DH 그룹: Group 14 이상
- PFS: 반드시 활성화
- 키 교체: 정기적 수행

금지 사항:
- DES, 3DES 사용 금지
- MD5 해시 금지
- Group 1, 2 사용 금지
- 약한 Pre-shared Key 금지
```

### 인증 강화
```
다중 인증:
- 인증서 기반 인증
- XAUTH 추가 인증
- RADIUS/LDAP 연동
- 토큰 기반 인증

접근 제어:
- 시간 기반 접근 제어
- IP 주소 기반 제한
- 사용자별 권한 분리
- 세션 시간 제한
```

### 모니터링 및 감사
```
로그 관리:
- 모든 VPN 세션 로깅
- 실패한 연결 시도 기록
- 정기적인 로그 분석
- 이상 징후 탐지

성능 모니터링:
- 연결 수 모니터링
- 대역폭 사용량 추적
- 응답 시간 측정
- 가용성 모니터링
```

## 차세대 VPN 기술

### SD-WAN
```
특징:
- 소프트웨어 정의 WAN
- 다중 연결 활용
- 동적 경로 선택
- 중앙 집중식 관리

장점:
✅ 비용 절감
✅ 성능 향상
✅ 관리 단순화
✅ 유연성 향상
```

### SASE (Secure Access Service Edge)
```
개념:
- 네트워크와 보안 융합
- 클라우드 기반 서비스
- Zero Trust 보안 모델
- 글로벌 PoP 활용

구성 요소:
- SD-WAN
- CASB (Cloud Access Security Broker)
- FWaaS (Firewall as a Service)
- Zero Trust Network Access
```

### Zero Trust VPN
```
원칙:
- "신뢰하지 말고 검증하라"
- 최소 권한 접근
- 지속적인 인증
- 마이크로 세그멘테이션

기술:
- Software-Defined Perimeter
- Identity-based Access
- Device Trust
- Context-aware Access
```

## 실습 시나리오

### 시나리오 1: 기본 Site-to-Site VPN
```
목표: 본사와 지사 간 IPSec VPN 구축
토폴로지:
HQ (192.168.1.0/24) -- Internet -- Branch (192.168.2.0/24)

설정 포인트:
- IKE Policy 협상
- Pre-shared Key 설정
- Transform Set 구성
- 암호화 트래픽 정의
```

### 시나리오 2: Remote Access VPN
```
목표: 재택근무자를 위한 SSL VPN 구축
요구사항:
- 웹 브라우저 기반 접속
- 내부 리소스 접근
- 사용자 인증
- 트래픽 분할
```

### 시나리오 3: VPN 장애 복구
```
증상: VPN 연결 불안정
진단 단계:
1. Phase 1 상태 확인
2. Phase 2 상태 확인  
3. 네트워크 연결성 테스트
4. 설정 검증
5. 로그 분석
```

## 관련 주제
- [[네트워크 보안]] - VPN 보안 기초
- [[ACL]] - VPN 트래픽 제어
- [[NAT-PAT]] - VPN과 주소 변환
- [[라우팅 기초]] - VPN 라우팅

## 태그
#ccna #vpn #ipsec #ssl #wan #보안 #터널링 #암호화
