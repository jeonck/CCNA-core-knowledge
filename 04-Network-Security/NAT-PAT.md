# NAT/PAT (Network Address Translation)

## 개요
NAT(Network Address Translation)는 사설 IP 주소를 공인 IP 주소로 변환하는 기술입니다. IPv4 주소 고갈 문제를 해결하고 네트워크 보안을 강화하는 중요한 역할을 합니다.

## NAT의 필요성

### IPv4 주소 부족 문제
```
전체 IPv4 주소: 약 43억 개 (2^32)
실제 사용 가능: 약 37억 개
현재 인터넷 연결 디바이스: 수백억 개

해결책:
✅ NAT를 통한 주소 재사용
✅ 사설 IP 주소 활용
✅ 공인 IP 주소 절약
```

### 사설 IP 주소 (RFC 1918)
```
Class A: 10.0.0.0/8
- 범위: 10.0.0.0 ~ 10.255.255.255
- 주소 수: 16,777,216개
- 용도: 대규모 네트워크

Class B: 172.16.0.0/12  
- 범위: 172.16.0.0 ~ 172.31.255.255
- 주소 수: 1,048,576개
- 용도: 중간 규모 네트워크

Class C: 192.168.0.0/16
- 범위: 192.168.0.0 ~ 192.168.255.255  
- 주소 수: 65,536개
- 용도: 소규모 네트워크 (가정, 소상공인)
```

## NAT 유형

### Static NAT (정적 NAT)
```
특징:
- 1:1 주소 매핑
- 사설 IP와 공인 IP 고정 연결
- 양방향 연결 가능
- 서버 공개 시 주로 사용

장점:
✅ 예측 가능한 매핑
✅ 외부에서 내부 접근 가능
✅ 서버 호스팅에 적합

단점:
❌ 공인 IP 주소 많이 필요
❌ 확장성 제한
❌ 비용 증가
```

### Dynamic NAT (동적 NAT)
```
특징:
- 공인 IP 풀에서 동적 할당
- First-come, First-served 방식
- 풀 크기만큼 동시 연결 제한
- 세션 종료 시 IP 반환

장점:  
✅ 유연한 IP 할당
✅ 공인 IP 효율적 사용

단점:
❌ 외부에서 접근 불가
❌ 풀 고갈 시 연결 실패
❌ 로깅 복잡
```

### PAT (Port Address Translation)
```
특징:
- NAT Overload라고도 함
- 하나의 공인 IP + 포트 번호 조합
- 가장 일반적인 NAT 방식
- 수천 개 동시 연결 가능

장점:
✅ 최대 IP 주소 절약
✅ 높은 확장성
✅ 비용 효율적

단점:
❌ 일부 애플리케이션 호환성 문제
❌ 외부에서 직접 접근 불가
❌ 성능 오버헤드
```

## NAT 동작 원리

### 주소 변환 테이블
```
Inside Local    Inside Global    Outside Local    Outside Global
192.168.1.10   203.0.113.1     8.8.8.8         8.8.8.8
192.168.1.20   203.0.113.2     8.8.8.8         8.8.8.8

용어 설명:
- Inside Local: 내부 네트워크의 사설 IP
- Inside Global: 변환된 공인 IP  
- Outside Local: 외부에서 보는 목적지 IP
- Outside Global: 실제 외부 목적지 IP
```

### PAT 동작 예시
```
내부 → 외부 (Outbound):
Src: 192.168.1.10:1024 → 203.0.113.1:1024
Dst: 8.8.8.8:53       → 8.8.8.8:53

외부 → 내부 (Inbound):  
Src: 8.8.8.8:53       → 8.8.8.8:53
Dst: 203.0.113.1:1024 → 192.168.1.10:1024

NAT 테이블:
192.168.1.10:1024 ↔ 203.0.113.1:1024
```

## NAT 설정

### Static NAT 설정
```cisco
# Static NAT 매핑 정의
Router(config)# ip nat inside source static 192.168.1.10 203.0.113.10
Router(config)# ip nat inside source static 192.168.1.20 203.0.113.20

# 인터페이스 지정
Router(config)# interface fastethernet 0/0
Router(config-if)# ip nat inside

Router(config)# interface serial 0/0/0  
Router(config-if)# ip nat outside

# 포트 포워딩 (서버 공개)
Router(config)# ip nat inside source static tcp 192.168.1.100 80 203.0.113.1 80
Router(config)# ip nat inside source static tcp 192.168.1.100 443 203.0.113.1 443
```

### Dynamic NAT 설정
```cisco
# 공인 IP 풀 정의
Router(config)# ip nat pool PUBLIC_POOL 203.0.113.10 203.0.113.20 netmask 255.255.255.0

# 변환할 내부 주소 범위 정의 (ACL 사용)
Router(config)# access-list 1 permit 192.168.1.0 0.0.0.255

# NAT 규칙 연결
Router(config)# ip nat inside source list 1 pool PUBLIC_POOL

# 인터페이스 지정
Router(config)# interface fastethernet 0/0
Router(config-if)# ip nat inside

Router(config)# interface serial 0/0/0
Router(config-if)# ip nat outside
```

### PAT 설정
```cisco
# 방법 1: 인터페이스 IP 사용 (일반적)
Router(config)# access-list 1 permit 192.168.1.0 0.0.0.255
Router(config)# ip nat inside source list 1 interface serial 0/0/0 overload

# 방법 2: IP 풀 사용
Router(config)# ip nat pool PAT_POOL 203.0.113.1 203.0.113.1 netmask 255.255.255.0
Router(config)# ip nat inside source list 1 pool PAT_POOL overload

# 인터페이스 지정
Router(config)# interface fastethernet 0/0
Router(config-if)# ip nat inside

Router(config)# interface serial 0/0/0
Router(config-if)# ip nat outside
```

## NAT 확인 및 문제해결

### 확인 명령어
```cisco
# 현재 NAT 변환 테이블
Router# show ip nat translations

# NAT 통계 정보
Router# show ip nat statistics

# 특정 주소의 변환 정보
Router# show ip nat translations verbose

# 인터페이스별 NAT 설정 확인
Router# show ip interface brief | include NAT
```

### NAT 테이블 출력 예시
```
Router# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 203.0.113.1:1    192.168.1.10:1    8.8.8.8:1         8.8.8.8:1
tcp  203.0.113.1:1024 192.168.1.10:1024 8.8.8.8:80        8.8.8.8:80
tcp  203.0.113.1:1025 192.168.1.20:1025 8.8.8.8:443       8.8.8.8:443
---  203.0.113.10     192.168.1.100     ---                ---

범례:
--- : Static NAT 매핑
Pro : 프로토콜 (tcp, udp, icmp)
```

### NAT 통계 정보
```
Router# show ip nat statistics
Total active translations: 3 (1 static, 2 dynamic; 2 extended)
Peak translations: 10, occurred 00:15:23 ago
Outside interfaces:
  Serial0/0/0
Inside interfaces:
  FastEthernet0/0
Hits: 1500  Misses: 5
CEF Translated packets: 1495, CEF Punted packets: 5
Expired translations: 2
Dynamic mappings:
-- Inside Source
[Id: 1] access-list 1 pool PAT_POOL refcount 2
 pool PAT_POOL: netmask 255.255.255.0
        start 203.0.113.1 end 203.0.113.1
        type generic, total addresses 1, allocated 1 (100%), misses 0
```

## 일반적인 NAT 문제

### 1. Inside/Outside 인터페이스 미설정
```
증상: NAT가 동작하지 않음
확인: show ip nat statistics
해결: 올바른 인터페이스에 ip nat inside/outside 설정

오류 예시:
Router# show ip nat translations
% NAT not enabled on any interface
```

### 2. ACL 설정 오류
```
증상: 특정 주소만 NAT 안됨
확인: show access-lists

일반적인 실수:
# 잘못된 예 (서브넷 마스크 사용)
Router(config)# access-list 1 permit 192.168.1.0 255.255.255.0

# 올바른 예 (와일드카드 마스크)  
Router(config)# access-list 1 permit 192.168.1.0 0.0.0.255
```

### 3. NAT 풀 고갈
```
증상: 새로운 연결 실패
확인: show ip nat statistics (misses 증가)
해결: 풀 크기 증가 또는 PAT 사용

예시:
Dynamic mappings:
pool PUBLIC_POOL: netmask 255.255.255.0
    start 203.0.113.10 end 203.0.113.20
    type generic, total addresses 11, allocated 11 (100%), misses 15
```

### 4. 포트 포워딩 문제
```
증상: 외부에서 내부 서버 접근 불가
확인: 
- Static NAT 설정 확인
- 방화벽/ACL 설정 확인
- 서버 서비스 상태 확인

해결:
Router(config)# ip nat inside source static tcp 192.168.1.100 80 interface serial0/0/0 80
```

## NAT 디버깅

### 디버깅 명령어
```cisco
# 기본 NAT 디버그
Router# debug ip nat

# 상세 NAT 디버그
Router# debug ip nat detailed

# 특정 주소만 디버그
Router# debug ip nat 192.168.1.10

# 디버그 비활성화
Router# no debug all
```

### 디버그 출력 예시
```
*Mar  1 00:15:20.207: NAT: s=192.168.1.10->203.0.113.1, d=8.8.8.8 [100]
*Mar  1 00:15:20.211: NAT: s=8.8.8.8, d=203.0.113.1->192.168.1.10 [100]

설명:
s= source (출발지)
d= destination (목적지)  
-> 주소 변환
[100] 패킷 ID
```

## NAT 최적화

### 성능 최적화
```cisco
# NAT 타임아웃 조정
Router(config)# ip nat translation timeout 3600
Router(config)# ip nat translation tcp-timeout 86400
Router(config)# ip nat translation udp-timeout 300

# 불필요한 변환 제거
Router# clear ip nat translation *
Router# clear ip nat translation inside 192.168.1.10

# NAT 통계 초기화
Router# clear ip nat statistics
```

### 메모리 관리
```cisco
# 최대 변환 테이블 크기 제한
Router(config)# ip nat translation max-entries 1000

# 변환 항목별 제한
Router(config)# ip nat settings mode aggressive
```

## NAT 고급 기능

### NAT Virtual Interface (NVI)
```cisco
# 전통적인 NAT의 대안
Router(config)# interface fastethernet 0/0
Router(config-if)# ip nat enable

Router(config)# interface serial 0/0/0
Router(config-if)# ip nat enable

# NVI를 사용한 NAT 설정
Router(config)# ip nat source list 1 interface serial0/0/0 overload
```

### NAT와 DNS
```
문제: 내부에서 공인 도메인으로 내부 서버 접근 시 라우팅 문제
해결: DNS 닥터링 또는 Split DNS

Router(config)# ip nat inside source static 192.168.1.100 203.0.113.1
Router(config)# ip host server.company.com 192.168.1.100
```

### NAT와 QoS
```cisco
# NAT 후 QoS 적용
Router(config)# class-map match-all WEB_TRAFFIC
Router(config-cmap)# match protocol http

Router(config)# policy-map QOS_POLICY  
Router(config-pmap)# class WEB_TRAFFIC
Router(config-pmap-c)# bandwidth percent 30
```

## NAT 보안 고려사항

### 보안 장점
```
✅ 내부 네트워크 숨김 효과
✅ 직접적인 외부 접근 차단
✅ 주소 공간 분리
✅ 간단한 방화벽 역할
```

### 보안 한계
```
❌ 강력한 보안 솔루션 아님
❌ 애플리케이션 계층 공격 미차단
❌ VPN/터널 내 트래픽 투명하게 전달
❌ IPv6에서는 사용 불가
```

### 보안 강화 방법
```cisco
# ACL과 함께 사용
Router(config)# access-list 100 permit tcp any host 203.0.113.1 eq 80
Router(config)# access-list 100 permit tcp any host 203.0.113.1 eq 443
Router(config)# access-list 100 deny ip any any

Router(config)# interface serial 0/0/0
Router(config-if)# ip access-group 100 in
```

## NAT 모범 사례

### 설계 원칙
```
1. 용도별 NAT 방식 선택
   - 서버: Static NAT
   - 클라이언트: PAT
   - DMZ: Dynamic NAT

2. 주소 계획
   - 사설 IP 대역 표준화
   - 공인 IP 효율적 사용
   - 향후 확장 고려

3. 모니터링
   - 정기적인 변환 테이블 확인
   - 풀 사용률 모니터링
   - 성능 지표 추적

4. 문서화
   - NAT 정책 문서화
   - 포트 포워딩 목록 관리
   - 변경 이력 추적
```

### 문제 예방
```
1. 적절한 타임아웃 설정
2. 충분한 공인 IP 풀 크기
3. 백업 인터넷 연결 고려
4. 로그 모니터링 체계 구축
5. 정기적인 설정 백업
```

## IPv6와 NAT

### IPv6에서의 변화
```
IPv6 특징:
- 주소 공간 무한대 (2^128)
- NAT 불필요
- End-to-End 연결성 복원
- 보안 기능 내장 (IPSec)

전환 기술:
- Dual Stack: IPv4/IPv6 동시 지원
- Tunneling: IPv6 over IPv4
- Translation: IPv6-IPv4 변환
```

## 관련 주제
- [[ACL]] - NAT와 함께 사용하는 보안 기능
- [[라우팅 기초]] - 패킷 포워딩과 주소 변환
- [[DHCP]] - 동적 IP 할당과 NAT
- [[문제해결 방법론]] - NAT 문제해결

## 태그
#ccna #nat #pat #주소변환 #네트워크보안 #ipv4
