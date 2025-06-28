# DNS (Domain Name System)

## 개요
DNS는 도메인 이름을 IP 주소로 변환하는 분산 데이터베이스 시스템입니다. 인터넷의 전화번호부 역할을 하며, 사용자가 기억하기 쉬운 도메인 이름을 통해 웹사이트나 서버에 접근할 수 있게 해줍니다.

## DNS의 필요성

### 문제점
```
IP 주소 사용의 어려움:
❌ 기억하기 어려움 (예: 172.217.160.78)
❌ 변경 시 모든 사용자에게 알려야 함
❌ 로드 밸런싱 구현 어려움
❌ 지역별 서버 분산 어려움
❌ 사용자 친화적이지 않음
```

### DNS 해결책
```
도메인 이름의 장점:
✅ 기억하기 쉬움 (예: google.com)
✅ IP 변경 시 DNS만 업데이트
✅ 로드 밸런싱 및 장애 복구 지원
✅ 지역별 서버 연결 가능
✅ 브랜드 인식 향상
✅ 계층적 구조로 관리 효율성
```

## DNS 계층 구조

### 계층적 네임스페이스
```
Root Domain (.)
├── Top-Level Domain (TLD)
│   ├── Generic TLD (.com, .org, .net)
│   ├── Country Code TLD (.kr, .jp, .us)
│   └── New gTLD (.tech, .info, .biz)
└── Second-Level Domain
    └── Subdomain

예시: www.example.com.
- . (Root)
- com (TLD)
- example (Second-Level Domain)
- www (Subdomain)
```

### FQDN (Fully Qualified Domain Name)
```
구성 요소:
[hostname].[subdomain].[domain].[tld].[root]

예시:
www.sales.company.com.
│   │     │       │   │
│   │     │       │   └─ Root (생략 가능)
│   │     │       └───── TLD
│   │     └─────────────── Domain
│   └───────────────────── Subdomain
└─────────────────────────── Hostname

실제 사용:
- www.google.com
- mail.company.co.kr
- ftp.download.microsoft.com
```

## DNS 동작 원리

### 재귀적 질의 (Recursive Query)
```
클라이언트 질의 과정:
1. PC → Local DNS Server: "www.example.com의 IP는?"
2. Local DNS → Root DNS: "com 서버 위치는?"
3. Root DNS → Local DNS: "com 서버는 xxx.xxx.xxx.xxx"
4. Local DNS → com DNS: "example.com 서버 위치는?"
5. com DNS → Local DNS: "example.com 서버는 yyy.yyy.yyy.yyy"
6. Local DNS → example.com DNS: "www.example.com IP는?"
7. example.com DNS → Local DNS: "www.example.com은 93.184.216.34"
8. Local DNS → PC: "93.184.216.34"
```

### 반복적 질의 (Iterative Query)
```
클라이언트가 직접 질의:
1. PC → Root DNS: "www.example.com IP는?"
2. Root DNS → PC: "com 서버에 물어보세요"
3. PC → com DNS: "www.example.com IP는?"
4. com DNS → PC: "example.com 서버에 물어보세요"
5. PC → example.com DNS: "www.example.com IP는?"
6. example.com DNS → PC: "93.184.216.34"
```

### DNS 캐싱
```
캐싱 레벨:
1. 브라우저 캐시 (수분~수십분)
2. 운영체제 캐시 (수분~수시간)
3. 로컬 DNS 서버 캐시 (TTL 기반)
4. ISP DNS 캐시 (TTL 기반)

TTL (Time To Live):
- DNS 레코드의 유효 시간
- 초 단위로 설정
- 캐시 만료 후 재질의
```

## DNS 레코드 유형

### 주요 레코드 타입
```
A (Address) 레코드:
- 도메인 → IPv4 주소 매핑
- 예: www.example.com → 93.184.216.34

AAAA 레코드:
- 도메인 → IPv6 주소 매핑
- 예: www.example.com → 2606:2800:220:1:248:1893:25c8:1946

CNAME (Canonical Name) 레코드:
- 별칭 → 실제 도메인 매핑
- 예: ftp.example.com → files.example.com

MX (Mail Exchange) 레코드:
- 메일 서버 지정
- 우선순위 포함
- 예: example.com → mail.example.com (우선순위 10)

PTR (Pointer) 레코드:
- IP 주소 → 도메인 (역방향 조회)
- 예: 93.184.216.34 → www.example.com

NS (Name Server) 레코드:
- 권한 있는 DNS 서버 지정
- 예: example.com → ns1.example.com

SOA (Start of Authority) 레코드:
- 도메인의 권한 정보
- 시리얼 번호, TTL, 갱신 주기 등

TXT 레코드:
- 텍스트 정보 저장
- SPF, DKIM, 도메인 인증 등
```

### 레코드 예시
```
example.com.    IN  SOA   ns1.example.com. admin.example.com. (
                          2023120101 ; 시리얼 번호
                          3600       ; 갱신 주기
                          1800       ; 재시도 간격
                          604800     ; 만료 시간
                          86400 )    ; 최소 TTL

example.com.    IN  NS    ns1.example.com.
example.com.    IN  NS    ns2.example.com.
example.com.    IN  MX    10 mail.example.com.
example.com.    IN  A     93.184.216.34
www.example.com. IN A     93.184.216.34
ftp.example.com. IN CNAME files.example.com.
files.example.com. IN A   93.184.216.35
```

## DNS 서버 유형

### 권한 있는 서버 (Authoritative Server)
```
Primary DNS Server:
- 도메인의 원본 데이터 보유
- 모든 DNS 레코드의 마스터
- 변경 사항의 최초 적용점

Secondary DNS Server:
- Primary 서버의 복사본
- Zone Transfer로 데이터 동기화
- Primary 서버 장애 시 백업 역할
- 부하 분산 효과
```

### 재귀적 서버 (Recursive Server)
```
특징:
- 클라이언트 대신 완전한 질의 수행
- 결과를 캐싱하여 성능 향상
- 일반적으로 ISP나 조직에서 운영

예시:
- Google DNS: 8.8.8.8, 8.8.4.4
- Cloudflare DNS: 1.1.1.1, 1.0.0.1
- Quad9 DNS: 9.9.9.9
```

### 포워딩 서버 (Forwarding Server)
```
동작:
- 질의를 다른 DNS 서버로 전달
- 자체적으로 재귀 질의 수행하지 않음
- 네트워크 정책 적용 가능

용도:
- 내부 네트워크 DNS 필터링
- 특정 도메인만 내부 서버로 전달
- 보안 정책 적용
```

## Cisco에서 DNS 설정

### 기본 DNS 설정
```cisco
# DNS 조회 활성화 (기본값)
Router(config)# ip domain-lookup

# DNS 서버 지정
Router(config)# ip name-server 8.8.8.8
Router(config)# ip name-server 8.8.4.4

# 도메인 이름 설정
Router(config)# ip domain-name company.com

# 도메인 리스트 추가 (검색 순서)
Router(config)# ip domain-list company.com
Router(config)# ip domain-list company.co.kr
```

### 정적 DNS 항목
```cisco
# 로컬 호스트 테이블
Router(config)# ip host server1 192.168.1.100
Router(config)# ip host server2 192.168.1.101
Router(config)# ip host printer1 192.168.1.200

# 여러 IP 주소 지정 (로드 밸런싱)
Router(config)# ip host web-server 192.168.1.10 192.168.1.11
```

### DNS 캐시 관리
```cisco
# DNS 캐시 확인
Router# show hosts

# DNS 캐시 삭제
Router# clear host *
Router# clear host server1

# DNS 캐시 비활성화
Router(config)# no ip domain-lookup
```

## DNS 문제해결

### 일반적인 DNS 문제
```
1. DNS 서버 응답 없음
   증상: "Server can't find domain: NXDOMAIN"
   원인: DNS 서버 장애, 방화벽 차단
   해결: 다른 DNS 서버 시도, 연결성 확인

2. 도메인 존재하지 않음
   증상: "Non-existent domain"
   원인: 잘못된 도메인 이름, 만료된 도메인
   해결: 도메인 이름 확인, 등록 상태 점검

3. DNS 응답 지연
   증상: 웹사이트 로딩 느림
   원인: DNS 서버 과부하, 네트워크 지연
   해결: 더 빠른 DNS 서버 사용

4. 캐시된 잘못된 정보
   증상: 변경된 IP로 접속 안됨
   원인: 오래된 DNS 캐시
   해결: DNS 캐시 플러시
```

### DNS 진단 도구

#### Windows 명령어
```cmd
# DNS 조회
nslookup google.com

# 특정 레코드 타입 조회
nslookup -type=MX google.com

# 권한 있는 서버 조회
nslookup -type=NS google.com

# 역방향 조회
nslookup 8.8.8.8

# DNS 캐시 확인
ipconfig /displaydns

# DNS 캐시 플러시
ipconfig /flushdns
```

#### Linux/Unix 명령어
```bash
# DNS 조회 (dig 권장)
dig google.com

# 특정 레코드 타입
dig google.com MX
dig google.com NS
dig google.com AAAA

# 권한 있는 서버에서 직접 조회
dig @8.8.8.8 google.com

# 역방향 조회
dig -x 8.8.8.8

# 전체 질의 과정 추적
dig +trace google.com

# 짧은 출력
dig +short google.com
```

#### Cisco 장비 명령어
```cisco
# DNS 조회 테스트
Router# nslookup google.com

# 특정 DNS 서버로 조회
Router# nslookup google.com 8.8.8.8

# 호스트 테이블 확인
Router# show hosts

# IP 주소로 역방향 조회
Router# nslookup 8.8.8.8
```

## DNS 보안

### 보안 위협
```
DNS 스푸핑:
- 가짜 DNS 응답으로 사용자를 악성 사이트로 유도
- 캐시 포이즈닝 공격

DNS 하이재킹:
- DNS 설정을 변경하여 트래픽 가로채기
- 라우터나 시스템 해킹 통해 수행

DNS 터널링:
- DNS 질의를 통한 데이터 은밀 전송
- 방화벽 우회 기법

DDoS 공격:
- DNS 서버에 대량 질의로 서비스 마비
- 증폭 공격에 DNS 서버 악용
```

### 보안 대책
```
DNSSEC (DNS Security Extensions):
- 디지털 서명으로 DNS 응답 인증
- 데이터 무결성 보장
- 스푸핑 방지

DNS over HTTPS (DoH):
- HTTPS를 통한 암호화된 DNS 질의
- 개인정보 보호 강화

DNS over TLS (DoT):
- TLS를 통한 암호화된 DNS 질의
- 포트 853 사용

DNS 필터링:
- 악성 도메인 차단
- 카테고리별 접근 제어
- 보안 정책 적용
```

### Cisco DNS 보안 설정
```cisco
# DNS 기반 방화벽 (Umbrella 연동)
Router(config)# ip dns server

# DNS 인스펙션 (ASA)
ASA(config)# policy-map global_policy
ASA(config-pmap)# class inspection_default
ASA(config-pmap-c)# inspect dns

# DNS 질의 로깅
Router(config)# ip dns spoofing
Router(config)# logging on
```

## DNS 성능 최적화

### 캐싱 전략
```
TTL 최적화:
- 자주 변하는 레코드: 짧은 TTL (5분~1시간)
- 안정적인 레코드: 긴 TTL (24시간~1주일)
- 긴급 변경 시: TTL을 미리 짧게 설정

로컬 DNS 캐시:
- 조직 내부 DNS 서버 운영
- 자주 사용하는 도메인 사전 캐싱
- 외부 의존성 감소
```

### 로드 밸런싱
```
Round Robin DNS:
- 동일 도메인에 여러 A 레코드
- 순차적으로 다른 IP 반환
- 간단한 부하 분산

지리적 DNS:
- 사용자 위치 기반 응답
- 가장 가까운 서버로 연결
- 성능 향상 및 지연 감소

가중치 기반:
- 서버 성능에 따른 트래픽 분배
- 동적 부하 조절
- 장애 서버 자동 제외
```

## DNS 모니터링

### 성능 지표
```
질의 응답 시간:
- 평균 응답 시간 < 100ms 권장
- 95% 질의가 200ms 이내

성공률:
- 99.9% 이상 성공적 응답
- 오류율 < 0.1%

캐시 히트율:
- 80% 이상 캐시에서 응답
- 외부 질의 최소화

가용성:
- 99.99% 이상 서비스 가용성
- 다중화를 통한 고가용성
```

### 모니터링 도구
```
내장 도구:
- dig, nslookup 스크립트
- 정기적인 질의 테스트
- 응답 시간 측정

전문 도구:
- PRTG Network Monitor
- SolarWinds DNS 모니터링
- Nagios DNS 플러그인
- Zabbix DNS 템플릿

로그 분석:
- DNS 질의 로그 분석
- 트래픽 패턴 파악
- 이상 징후 탐지
```

## 실습 시나리오

### 시나리오 1: 기본 DNS 설정
```cisco
# 라우터에서 DNS 클라이언트 설정
Router(config)# ip domain-lookup
Router(config)# ip name-server 8.8.8.8
Router(config)# ip name-server 168.126.63.1
Router(config)# ip domain-name company.com

# 테스트
Router# nslookup google.com
Router# ping google.com
```

### 시나리오 2: 내부 DNS 서버
```cisco
# 내부 서버에 대한 정적 항목
Router(config)# ip host intranet 192.168.1.100
Router(config)# ip host mail 192.168.1.101
Router(config)# ip host file-server 192.168.1.102

# 확인
Router# show hosts
Router# ping intranet
```

### 시나리오 3: DNS 문제해결
```
증상: 웹사이트 접속 불가
진단 단계:
1. ping IP 주소 (예: ping 8.8.8.8)
2. nslookup 도메인 이름
3. 다른 DNS 서버로 테스트
4. DNS 설정 확인
```

## 미래 동향

### DNS over HTTPS (DoH)
```
특징:
- HTTPS(443 포트)로 DNS 질의
- 웹 트래픽으로 위장
- 검열 및 감시 우회

도입:
- Firefox, Chrome 기본 지원
- Windows 10 내장 지원
- 모바일 운영체제 확산
```

### IPv6 DNS
```
변화:
- AAAA 레코드 사용 증가
- 듀얼 스택 환경 고려
- IPv6 전용 환경 대비

과제:
- IPv4/IPv6 동시 지원
- 성능 최적화
- 호환성 확보
```

## 관련 주제
- [[DHCP]] - 동적 네트워크 설정
- [[NAT-PAT]] - 주소 변환과 DNS
- [[네트워크 보안]] - DNS 보안
- [[문제해결 방법론]] - DNS 문제해결

## 태그
#ccna #dns #도메인 #이름해석 #네트워크서비스 #인터넷
