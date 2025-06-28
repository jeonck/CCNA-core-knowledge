# Dynamic ARP Inspection (DAI)

## 📝 개요
Dynamic ARP Inspection(DAI)은 ARP 스푸핑 공격을 방지하기 위해 스위치에서 ARP 패킷을 검증하고 제어하는 보안 기능입니다. DHCP Snooping 바인딩 테이블을 활용하여 유효한 IP-MAC 매핑만 허용합니다.

## 🎯 ARP 공격 유형

### ARP Spoofing/Poisoning
```
공격 시나리오:
1. 공격자가 위조된 ARP 응답 전송
2. 피해자의 ARP 테이블을 오염시킴
3. 트래픽이 공격자를 경유하도록 리다이렉션
4. Man-in-the-Middle 공격 수행

예시:
정상: 192.168.1.1 → MAC: AA:BB:CC:DD:EE:FF (진짜 게이트웨이)
공격: 192.168.1.1 → MAC: 11:22:33:44:55:66 (공격자 MAC)

결과:
❌ 트래픽 도청
❌ 세션 하이재킹  
❌ 데이터 변조
❌ 서비스 거부
```

### Gratuitous ARP Attack
```
공격 시나리오:
1. 공격자가 무차별 Gratuitous ARP 전송
2. 네트워크의 모든 장비 ARP 테이블 오염
3. 대규모 트래픽 가로채기

Gratuitous ARP:
- 요청하지 않은 ARP 응답
- IP 주소 변경 시 정상적으로 사용
- 공격자가 악용하기 쉬움
```

### ARP Cache Poisoning
```
공격 방법:
1. 대량의 가짜 ARP 응답 전송
2. ARP 캐시 테이블 오버플로우
3. 정상 항목을 악의적 항목으로 대체

영향:
❌ 네트워크 성능 저하
❌ 연결 불안정
❌ 보안 우회
```

## 🛡️ DAI 동작 원리

### 검증 소스
#### DHCP Snooping Binding Table (기본)
```
검증 과정:
1. ARP 패킷 수신
2. DHCP Snooping 바인딩 테이블 조회
3. IP-MAC-VLAN-포트 매핑 확인
4. 일치하면 허용, 불일치하면 차단

장점:
✅ 동적 검증
✅ 자동 업데이트
✅ 관리 부담 적음
```

#### ARP ACL (정적 검증)
```
사용 경우:
- DHCP 미사용 환경
- 정적 IP 할당
- 서버/인프라 장비

설정 예시:
arp access-list STATIC_HOSTS
 permit ip host 192.168.1.10 mac host aabb.ccdd.eeff
 permit ip host 192.168.1.20 mac host 1122.3344.5566
```

### 포트 분류
#### Trusted Port
```
특징:
- ARP 검증 없이 모든 ARP 패킷 허용
- 일반적으로 라우터, 서버, 다른 스위치 연결 포트
- 업링크/트렁크 포트

설정:
Switch(config-if)# ip arp inspection trust
```

#### Untrusted Port (기본값)
```
특징:
- 모든 ARP 패킷 검증
- 일반적으로 클라이언트 연결 포트
- 액세스 포트

검증 기준:
- 바인딩 테이블 또는 ARP ACL과 일치
- Rate Limiting 적용
```

## ⚙️ DAI 설정

### 기본 설정
```cisco
# DAI 전역 활성화 (VLAN별)
Switch(config)# ip arp inspection vlan 10,20,30

# 신뢰할 수 있는 포트 설정
Switch(config)# interface gi0/24
Switch(config-if)# ip arp inspection trust

# DHCP Snooping과 연동 (자동)
Switch(config)# ip dhcp snooping vlan 10,20,30
```

### ARP ACL 설정 (정적 검증)
```cisco
# ARP ACL 생성
Switch(config)# arp access-list STATIC_MAPPINGS
Switch(config-arp-nacl)# permit ip host 192.168.1.10 mac host aabb.ccdd.eeff
Switch(config-arp-nacl)# permit ip host 192.168.1.20 mac host 1122.3344.5566
Switch(config-arp-nacl)# permit ip host 192.168.1.30 mac host 3344.5566.7788

# VLAN에 ARP ACL 적용
Switch(config)# ip arp inspection vlan 10 arp access-list STATIC_MAPPINGS
```

### 고급 설정
```cisco
# Rate Limiting 설정 (untrusted 포트)
Switch(config)# interface range fa0/1-20
Switch(config-if-range)# ip arp inspection limit rate 15

# 추가 검증 옵션
Switch(config)# ip arp inspection validate src-mac dst-mac ip

# 로깅 설정
Switch(config)# ip arp inspection log-buffer entries 1024
Switch(config)# ip arp inspection log-buffer logs 200 interval 10
```

### 검증 옵션 상세
```cisco
# 개별 검증 옵션
Switch(config)# ip arp inspection validate src-mac    # 소스 MAC 검증
Switch(config)# ip arp inspection validate dst-mac    # 목적지 MAC 검증  
Switch(config)# ip arp inspection validate ip         # IP 주소 검증

# 다중 검증 (권장)
Switch(config)# ip arp inspection validate src-mac dst-mac ip

검증 내용:
src-mac: 이더넷 헤더와 ARP 헤더의 소스 MAC 일치 확인
dst-mac: 이더넷 헤더와 ARP 헤더의 목적지 MAC 일치 확인
ip: 유효하지 않은 IP 주소 차단 (0.0.0.0, 224.x.x.x 등)
```

## 📊 DAI 확인 및 모니터링

### 확인 명령어
```cisco
# DAI 상태 확인
Switch# show ip arp inspection

# VLAN별 DAI 설정 확인  
Switch# show ip arp inspection vlan 10

# 인터페이스별 설정 확인
Switch# show ip arp inspection interfaces

# 통계 정보 확인
Switch# show ip arp inspection statistics

# 로그 확인
Switch# show ip arp inspection log
```

### DAI 상태 출력 예시
```
Switch# show ip arp inspection
Source Mac Validation      : Enabled
Destination Mac Validation : Enabled  
IP Address Validation      : Enabled
 Vlan     Configuration    Operation   ACL Match          Static ACL
 ----     -------------    ---------   ---------          ----------
   10     Enabled          Active      Deny               STATIC_MAPPINGS
   20     Enabled          Active      Permit             
   30     Enabled          Active      Permit             

 Vlan     ACL Logging      DHCP Logging      Probe Logging
 ----     -----------      ------------      -------------
   10     Deny             Deny              Off
   20     Deny             Deny              Off  
   30     Deny             Deny              Off
```

### 통계 정보 해석
```
Switch# show ip arp inspection statistics

 Vlan      Forwarded        Dropped     DHCP Drops      ACL Drops
 ----      ---------        -------     ----------      ---------
   10            150             25              0             25
   20            200              5              5              0
   30            175             10             10              0

분석:
- VLAN 10: ARP ACL에 의한 차단 25건
- VLAN 20: DHCP 바인딩 불일치로 차단 5건  
- VLAN 30: DHCP 바인딩 불일치로 차단 10건
```

## 🔧 Rate Limiting

### 설정 방법
```cisco
# 인터페이스별 Rate Limiting
Switch(config)# interface fa0/1
Switch(config-if)# ip arp inspection limit rate 15

# 버스트 허용 설정
Switch(config-if)# ip arp inspection limit rate 15 burst interval 1

# Rate Limiting 비활성화 (trusted 포트)
Switch(config)# interface gi0/24
Switch(config-if)# ip arp inspection limit rate none
```

### Rate Limiting 매개변수
| 매개변수 | 기본값 | 설명 |
|----------|--------|------|
| **rate** | 15 pps | 초당 허용 ARP 패킷 수 |
| **burst interval** | 1초 | 버스트 측정 간격 |
| **none** | - | Rate Limiting 비활성화 |

### 권장 설정값
```
포트 유형별 권장값:
- 클라이언트 포트: 5-15 pps
- 서버 포트: 30-50 pps  
- 업링크 포트: none (비활성화)
- 트렁크 포트: 높은 값 또는 none
```

## 🚨 문제해결

### 일반적인 문제들

#### 1. 정상 ARP 트래픽 차단
```
증상: 클라이언트가 네트워크 접근 불가
원인: 
- DHCP Snooping 바인딩 누락
- ARP ACL 설정 오류
- 잘못된 신뢰 포트 설정

확인:
Switch# show ip dhcp snooping binding
Switch# show ip arp inspection log
Switch# show ip arp inspection statistics

해결:
1. DHCP Snooping 먼저 확인
2. ARP ACL 정확성 검증
3. 포트 신뢰성 재검토
```

#### 2. Rate Limiting으로 인한 차단
```
증상: 간헐적으로 ARP 실패
원인: Rate Limiting 값이 너무 낮음

확인:
Switch# show ip arp inspection statistics
Switch# show ip arp inspection interfaces

해결:
Switch(config-if)# ip arp inspection limit rate 30  # 값 증가
```

#### 3. DHCP Snooping 의존성 문제
```
증상: DAI가 작동하지 않음
원인: DHCP Snooping 비활성화 또는 설정 오류

해결:
1. DHCP Snooping 활성화 확인
2. 동일 VLAN에서 두 기능 모두 활성화
3. 바인딩 테이블 존재 확인
```

### 디버깅
```cisco
# DAI 디버그
Switch# debug arp inspection

# 특정 VLAN만 디버그
Switch# debug arp inspection vlan 10

# 디버그 비활성화
Switch# undebug all
```

### 로그 분석
```cisco
Switch# show ip arp inspection log

Output:
Total Log Buffer Size : 1024
Total Log Entries     : 15
Total Messages Printed: 10

Interface   Vlan  Src MAC         Dest MAC        Src IP       Dest IP     Time
----------  ----  --------------  --------------  -----------  ----------  --------
Fa0/3       10    aabb.ccdd.eeff  ffff.ffff.ffff  192.168.1.50 192.168.1.1 12:34:56

분석:
- 인터페이스 Fa0/3에서 ARP 스푸핑 시도 감지
- MAC aabb.ccdd.eeff가 게이트웨이 IP 192.168.1.1 요구
```

## 🔗 다른 보안 기능과의 연동

### DHCP Snooping과 연동
```cisco
# 필수 설정 순서
1. DHCP Snooping 활성화
Switch(config)# ip dhcp snooping vlan 10

2. DAI 활성화  
Switch(config)# ip arp inspection vlan 10

3. 포트 신뢰성 설정
Switch(config-if)# ip dhcp snooping trust
Switch(config-if)# ip arp inspection trust
```

### IP Source Guard와 연동
```cisco
# 3계층 보안 구성
Switch(config)# interface fa0/1

# DHCP Snooping (기반)
Switch(config-if)# switchport access vlan 10

# IP Source Guard (IP 스푸핑 방지)
Switch(config-if)# ip verify source

# DAI는 VLAN 레벨에서 자동 적용
```

### Port Security와 함께 사용
```cisco
Switch(config)# interface fa0/1
# Port Security
Switch(config-if)# switchport port-security
Switch(config-if)# switchport port-security maximum 1

# DHCP Snooping Rate Limiting
Switch(config-if)# ip dhcp snooping limit rate 15

# DAI Rate Limiting  
Switch(config-if)# ip arp inspection limit rate 15
```

## 📋 모범 사례

### 설계 원칙
```
1. 계층적 보안 접근
   DHCP Snooping → DAI → IP Source Guard

2. 포트 분류 일관성
   - 같은 포트에서 모든 보안 기능의 신뢰성 일치
   - 업링크는 모두 trusted
   - 클라이언트는 모두 untrusted

3. 단계적 배포
   - 테스트 환경에서 먼저 검증
   - 로그 모드로 시작 후 점진적 적용
   - 모니터링 체계 먼저 구축
```

### 운영 가이드라인
```
1. 모니터링
   - 정기적인 로그 검토
   - 통계 지표 추적
   - 이상 패턴 감지

2. 문제 대응
   - 빠른 우회 절차 수립
   - 롤백 계획 준비
   - 에스컬레이션 체계 구축

3. 유지보수
   - 정기적인 설정 백업
   - ARP ACL 업데이트
   - 성능 최적화
```

## 🔍 고급 기능

### 고급 검증 옵션
```cisco
# 추가 검증 활성화
Switch(config)# ip arp inspection validate src-mac dst-mac ip

# 검증 실패 시 동작
Switch(config)# ip arp inspection drop-threshold 50

# Gratuitous ARP 처리
Switch(config)# ip arp inspection filter GARP_FILTER vlan 10
Switch(config)# arp access-list GARP_FILTER
Switch(config-arp-nacl)# permit request ip any mac any
Switch(config-arp-nacl)# deny response ip any mac any
```

### 성능 최적화
```cisco
# 로그 버퍼 최적화
Switch(config)# ip arp inspection log-buffer entries 2048
Switch(config)# ip arp inspection log-buffer logs 100 interval 5

# 메모리 효율성
Switch(config)# no ip arp inspection log-buffer
```

## 📈 확장성 고려사항

### 대규모 환경 설계
```
고려사항:
- VLAN 수 증가에 따른 설정 복잡성
- ARP 트래픽 증가에 따른 성능 영향
- 바인딩 테이블 크기 제한

해결방안:
- 자동화 도구 활용
- 계층적 네트워크 설계
- 하드웨어 성능 고려
```

### 하이브리드 환경
```
DHCP + 정적 IP 혼합:
1. DHCP Snooping으로 동적 IP 처리
2. ARP ACL로 정적 IP 처리
3. 우선순위: DHCP > ARP ACL
```

## 🔗 관련 주제
- [[DHCP Snooping|DHCP Snooping]] - DAI의 기반 기능
- [[IP Source Guard|IP Source Guard]] - IP 레벨 보안
- [[포트 보안|Port Security]] - MAC 주소 기반 보안
- [[네트워크 보안|보안]] - 전체적인 보안 전략

## 🏷️ 태그
#ccna #dai #dynamic-arp-inspection #네트워크보안 #arp-spoofing #스위치보안 #공격방지