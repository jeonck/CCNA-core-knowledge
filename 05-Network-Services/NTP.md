# NTP (Network Time Protocol)

## 📝 개요
NTP는 네트워크를 통해 컴퓨터 시스템 간의 시계 동기화를 제공하는 프로토콜입니다. 정확한 시간 동기화는 네트워크 보안, 로깅, 인증 등에 필수적입니다.

## ⏰ NTP의 중요성

### 시간 동기화가 필요한 이유
```
보안:
- 인증서 유효성 검사
- Kerberos 인증 (5분 이내 동기화 필요)
- 로그 상관관계 분석
- 디지털 서명 검증

네트워크 관리:
- 로그 파일 시간 정합성
- 장애 분석 및 문제해결
- 성능 모니터링
- 백업 스케줄링

규정 준수:
- 감사 요구사항
- 법적 증거 능력
- 금융 거래 기록
```

## 🏗️ NTP 계층 구조 (Stratum)

### Stratum 레벨
| 레벨 | 설명 | 예시 |
|------|------|------|
| **Stratum 0** | 기준 클록 | 원자시계, GPS, 라디오 시계 |
| **Stratum 1** | 기본 서버 | Stratum 0에 직접 연결 |
| **Stratum 2** | 보조 서버 | Stratum 1 서버 동기화 |
| **Stratum 3** | 클라이언트 | Stratum 2 서버 동기화 |
| ... | ... | 최대 Stratum 15까지 |
| **Stratum 16** | 비동기화 | 동기화되지 않은 상태 |

### 계층 구조 예시
```
[Stratum 0] - GPS/원자시계
     │
[Stratum 1] - time.nist.gov
     │
[Stratum 2] - 지역 NTP 서버
     │
[Stratum 3] - 기업 NTP 서버
     │
[Stratum 4] - 네트워크 장비/PC
```

## 🔧 NTP 설정 (Cisco)

### 기본 NTP 클라이언트 설정
```cisco
# NTP 서버 지정
Router(config)# ntp server 129.6.15.28
Router(config)# ntp server 132.163.96.1

# 시간대 설정
Router(config)# clock timezone KST 9
Router(config)# clock summer-time KDT recurring

# NTP 업데이트 활성화
Router(config)# ntp update-calendar
```

### NTP 서버 설정
```cisco
# 로컬 클록을 Stratum 8로 설정
Router(config)# ntp master 8

# 클라이언트 접근 제한
Router(config)# ntp access-group peer 10
Router(config)# ntp access-group serve 20
Router(config)# ntp access-group query-only 30

# ACL 설정
Router(config)# access-list 10 permit 192.168.1.0 0.0.0.255
Router(config)# access-list 20 permit 192.168.0.0 0.0.255.255
Router(config)# access-list 30 permit any
```

### NTP 인증 설정
```cisco
# NTP 인증 활성화
Router(config)# ntp authenticate

# 인증 키 설정
Router(config)# ntp authentication-key 1 md5 cisco123
Router(config)# ntp trusted-key 1

# 서버에 인증키 적용
Router(config)# ntp server 192.168.1.10 key 1
```

## 📊 NTP 모니터링 및 확인

### 기본 확인 명령어
```cisco
# NTP 상태 확인
Router# show ntp status

Output:
Clock is synchronized, stratum 3, reference is 129.6.15.28
nominal freq is 250.0000 Hz, actual freq is 249.9995 Hz
precision is 2**19, version is 4
reference time is E2C4AA3C.A3F1B117 (13:25:00.640 UTC Fri Jul 1 2024)
clock offset is -2.4056 msec, root delay is 75.77 msec
root dispersion is 45.23 msec, peer dispersion is 3.20 msec

# NTP 연결 정보
Router# show ntp associations

     address         ref clock       st   when   poll reach  delay  offset   disp
*~129.6.15.28       .NIST.           1     45     64   377  75.77   -2.406   3.20
+~132.163.96.1      .NIST.           1     12     64   377  68.45   -1.234   2.45

# 상세 정보
Router# show ntp associations detail
```

### 출력 해석
```
기호 의미:
* : 현재 동기화 중인 서버
+ : 동기화 가능한 서버
- : 동기화 불가능한 서버
~ : 설정된 서버
# : 로컬 클록

reach: 마지막 8번의 폴링 성공 여부 (377 = 모두 성공)
delay: 왕복 시간 (밀리초)
offset: 시간 차이 (밀리초)
disp: 분산 값 (밀리초)
```

## 🚨 문제해결

### 일반적인 문제들

#### 1. 동기화 실패
```
문제: show ntp status에서 "unsynchronized" 표시
원인:
- 네트워크 연결 문제
- 방화벽에서 UDP 123 차단
- NTP 서버 장애
- 큰 시간 차이 (panic threshold)

해결:
1. 연결성 확인: ping ntp_server_ip
2. 방화벽 규칙 확인
3. 다른 NTP 서버 시도
4. 수동 시간 설정 후 NTP 재시작
```

#### 2. 인증 실패
```
문제: 인증 설정 시 동기화 안됨
원인:
- 잘못된 인증키
- 키 ID 불일치
- trusted-key 설정 누락

해결:
Router# debug ntp events
Router# debug ntp authentication
```

#### 3. Stratum 레벨 문제
```
문제: 높은 Stratum 레벨
원인:
- 상위 서버와의 연결 끊어짐
- 설정된 서버가 모두 unreachable

해결:
- 여러 NTP 서버 설정
- 다양한 Stratum 레벨의 서버 사용
```

### 디버깅 명령어
```cisco
# NTP 이벤트 디버그
Router# debug ntp events

# NTP 패킷 디버그
Router# debug ntp packets

# NTP 동기화 디버그
Router# debug ntp sync

# NTP 인증 디버그
Router# debug ntp authentication

# 디버그 종료
Router# undebug all
```

## ⚙️ 고급 NTP 설정

### NTP 액세스 제어
```cisco
# 액세스 그룹 유형
- peer: 양방향 시간 동기화 허용
- serve: 시간 제공만 허용
- serve-only: 시간 제공만 허용 (쿼리 불가)
- query-only: 쿼리만 허용

# 설정 예시
Router(config)# ntp access-group peer 1
Router(config)# ntp access-group serve 2  
Router(config)# ntp access-group query-only 3

Router(config)# access-list 1 permit 192.168.1.0 0.0.0.255
Router(config)# access-list 2 permit 192.168.0.0 0.0.255.255
Router(config)# access-list 3 permit any
```

### NTP Peer 설정
```cisco
# 피어 관계 설정 (상호 동기화)
Router(config)# ntp peer 192.168.1.10
Router(config)# ntp peer 192.168.1.20 key 1
```

### Broadcast/Multicast 모드
```cisco
# 브로드캐스트 서버
Router(config)# interface fastethernet 0/0
Router(config-if)# ntp broadcast

# 브로드캐스트 클라이언트
Router(config-if)# ntp broadcast client

# 멀티캐스트 설정
Router(config-if)# ntp multicast 224.0.1.1
Router(config-if)# ntp multicast client 224.0.1.1
```

## 🔐 NTP 보안

### 보안 위협
```
공격 유형:
1. Man-in-the-Middle: 시간 조작
2. Replay Attack: 이전 패킷 재전송
3. DoS Attack: 서비스 거부
4. Amplification: 증폭 공격

대응 방안:
- 인증 사용
- 접근 제어 목록
- 방화벽 설정
- 모니터링
```

### 보안 강화 설정
```cisco
# 강력한 인증
Router(config)# ntp authentication-key 1 md5 StrongPassword123!
Router(config)# ntp trusted-key 1
Router(config)# ntp authenticate

# 소스 인터페이스 지정
Router(config)# ntp source loopback0

# 로깅 활성화
Router(config)# ntp logging

# 제한된 접근
Router(config)# ntp access-group serve-only 10
Router(config)# access-list 10 permit 192.168.1.0 0.0.0.255
```

## 🌐 공용 NTP 서버

### 권장 공용 NTP 서버
| 제공자 | 서버 주소 | 특징 |
|--------|-----------|------|
| **NIST** | time.nist.gov | 미국 표준기술연구소 |
| **Google** | time.google.com | 글로벌 anycast |
| **Cloudflare** | time.cloudflare.com | 빠른 응답 |
| **Pool Project** | pool.ntp.org | 분산 서버 풀 |
| **한국** | time.kriss.re.kr | 한국표준과학연구원 |

### 지역별 Pool 서버
```
전 세계: pool.ntp.org
아시아: asia.pool.ntp.org
한국: kr.pool.ntp.org
미국: us.pool.ntp.org
유럽: europe.pool.ntp.org
```

## 📋 NTP 구현 계획

### 설계 고려사항
```
1. 서버 선택
   - 지리적 위치 (네트워크 지연 최소화)
   - 신뢰성 (Stratum 레벨)
   - 가용성 (여러 서버 사용)

2. 네트워크 설계
   - 계층적 구조
   - 내부 NTP 서버 구성
   - 백업 시간 소스

3. 보안 정책
   - 인증 키 관리
   - 접근 제어
   - 모니터링 체계
```

### 구현 단계
```
1단계: 계획 수립
- 요구사항 분석
- 토폴로지 설계
- 서버 선택

2단계: 서버 구성
- 기본 NTP 서버 설정
- 인증 설정
- 접근 제어 설정

3단계: 클라이언트 설정
- 모든 네트워크 장비에 NTP 설정
- 시간대 설정
- 동기화 확인

4단계: 모니터링
- 동기화 상태 모니터링
- 로그 수집 및 분석
- 알림 설정
```

## 🔗 관련 주제
- [[네트워크 보안|보안]] - 시간 기반 인증과 보안
- [[로깅과 모니터링|로깅]] - 시간 동기화와 로그 분석
- [[DNS|DNS]] - 네트워크 서비스 통합
- [[SNMP|SNMP]] - 네트워크 관리 프로토콜

## 🏷️ 태그
#ccna #ntp #시간동기화 #네트워크서비스 #네트워크관리 #보안 #로깅 #stratum