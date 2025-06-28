# Syslog

## 📝 개요
Syslog는 네트워크 장비에서 발생하는 시스템 메시지, 이벤트, 오류 정보를 기록하고 전송하는 표준 프로토콜입니다. 네트워크 모니터링, 문제해결, 보안 감사에 필수적인 도구입니다.

## 🎯 Syslog의 중요성

### 주요 용도
```
📊 네트워크 모니터링
- 장비 상태 추적
- 성능 지표 수집
- 이상 상황 감지

🔧 문제해결
- 장애 원인 분석
- 이벤트 타임라인 구성
- 근본 원인 추적

🛡️ 보안 감사
- 접근 시도 로깅
- 설정 변경 추적
- 보안 정책 위반 감지

📋 규정 준수
- 감사 요구사항 충족
- 로그 보존 정책 준수
- 컴플라이언스 리포팅
```

## 📊 Syslog 메시지 구조

### RFC 3164 표준 형식
```
<Priority>Timestamp Hostname Process[PID]: Message

예시:
<189>Mar 15 10:30:25 R1 %SYS-5-CONFIG_I: Configured from console by admin on vty0
```

### 메시지 구성 요소
#### 1. Priority (우선순위)
```
계산 공식: Priority = Facility × 8 + Severity

Facility (0-23): 메시지 소스
Severity (0-7): 메시지 심각도
```

#### 2. Timestamp (시간 정보)
```
형식: MMM DD HH:MM:SS
예시: Mar 15 10:30:25

주의사항:
- 장비별 시간 동기화 필수 (NTP 사용)
- 타임존 설정 중요
- 정확한 시간 추적을 위한 기본 요소
```

#### 3. Hostname (호스트명)
```
메시지를 생성한 장비의 식별자
- 호스트명 또는 IP 주소
- 네트워크에서 유일해야 함
- 문제해결 시 장비 식별에 중요
```

## 🔢 Syslog Severity Levels

### 심각도 레벨 (0-7)
| 레벨 | 이름 | 키워드 | 설명 | 예시 |
|------|------|--------|------|------|
| **0** | Emergency | emergencies | 시스템 사용 불가 | 시스템 크래시 |
| **1** | Alert | alerts | 즉시 조치 필요 | 하드웨어 실패 |
| **2** | Critical | critical | 중요한 조건 | 온도 임계값 초과 |
| **3** | Error | errors | 오류 조건 | 인터페이스 다운 |
| **4** | Warning | warnings | 경고 조건 | 메모리 부족 |
| **5** | Notice | notifications | 정상이지만 중요한 조건 | 설정 변경 |
| **6** | Informational | informational | 정보성 메시지 | 인터페이스 업 |
| **7** | Debug | debugging | 디버그 메시지 | 프로토콜 상세 정보 |

### Cisco IOS 메시지 예시
```
%SYS-0-CPUHOG: Task is hogging the CPU          # Emergency
%SYS-1-FREEMEMLOW: Free Memory has dropped      # Alert  
%SYS-2-MALLOCFAIL: Memory allocation failed     # Critical
%LINK-3-UPDOWN: Interface Gi0/1, changed state  # Error
%SYS-4-CONFIG_NEWER: Configuration newer        # Warning
%SYS-5-CONFIG_I: Configured from console        # Notice
%LINK-6-UPDOWN: Interface Gi0/1, up             # Informational
%OSPF-7-ADJ: Sending DBD packet                 # Debug
```

## 🏢 Syslog Facilities

### 주요 Facility 코드
| 코드 | Facility | 설명 |
|------|----------|------|
| **0** | kernel | 커널 메시지 |
| **1** | user | 사용자 프로세스 |
| **4** | security | 보안/인증 메시지 |
| **6** | daemon | 시스템 데몬 |
| **16** | local0 | 로컬 사용 0 |
| **17** | local1 | 로컬 사용 1 |
| **18** | local2 | 로컬 사용 2 |
| **19** | local3 | 로컬 사용 3 |
| **20** | local4 | 로컬 사용 4 |
| **21** | local5 | 로컬 사용 5 |
| **22** | local6 | 로컬 사용 6 |
| **23** | local7 | 로컬 사용 7 |

### Cisco 장비에서의 Facility 활용
```cisco
# Facility 설정
Router(config)# logging facility local0

일반적인 용도:
local0: 라우터 로그
local1: 스위치 로그  
local2: 방화벽 로그
local3: 무선 장비 로그
local4: 보안 장비 로그
```

## ⚙️ Syslog 설정 (Cisco IOS)

### 기본 로깅 설정
```cisco
# 로깅 활성화 (기본적으로 활성화됨)
Router(config)# logging on

# 콘솔 로깅 설정
Router(config)# logging console warnings  # 레벨 4 이상만 콘솔에 표시

# 버퍼 로깅 설정
Router(config)# logging buffered 8192 informational  # 8KB 버퍼, 레벨 6 이상

# 원격 Syslog 서버 설정
Router(config)# logging host 192.168.1.100
Router(config)# logging trap informational  # 레벨 6 이상을 서버로 전송
```

### 고급 설정
```cisco
# 여러 Syslog 서버 설정
Router(config)# logging host 192.168.1.100
Router(config)# logging host 192.168.1.101
Router(config)# logging host 192.168.2.100

# Facility 설정
Router(config)# logging facility local0

# 소스 인터페이스 지정
Router(config)# logging source-interface loopback0

# 로깅 동기화 (콘솔 메시지 정리)
Router(config)# line console 0
Router(config-line)# logging synchronous

# 로깅 속도 제한
Router(config)# logging rate-limit 10
```

### 로깅 레벨별 설정
```cisco
# 목적지별 로깅 레벨 설정
Router(config)# logging console critical      # 콘솔: Critical 이상
Router(config)# logging buffered debugging    # 버퍼: 모든 레벨
Router(config)# logging monitor warnings      # VTY: Warning 이상
Router(config)# logging trap errors           # Syslog 서버: Error 이상

# 특정 서버에 특정 레벨 설정
Router(config)# logging host 192.168.1.100 transport udp port 514
Router(config)# logging host 192.168.1.100 severity informational
```

## 📋 Syslog 확인 명령어

### 기본 확인 명령어
```cisco
# 로깅 설정 확인
Router# show logging

# 로그 버퍼 내용 확인
Router# show logging buffer

# 로깅 히스토리 확인
Router# show logging history

# 로깅 통계 확인
Router# show logging statistics
```

### show logging 출력 예시
```
Router# show logging
Syslog logging: enabled (0 messages dropped, 3 messages rate-limited,
                0 flushes, 0 overruns, xml disabled, filtering disabled)

No Active Message Discriminator.

No Inactive Message Discriminator.

    Console logging: level warnings (4), 127 messages logged, xml disabled,
                     filtering disabled
    Monitor logging: level informational (6), 0 messages logged, xml disabled,
                     filtering disabled
    Buffer logging:  level informational (6), 156 messages logged, xml disabled,
                     filtering disabled
    Exception Logging: size (4096 bytes)
    Count and timestamp logging messages: disabled
    Persistent logging: disabled

No active filter modules.

    Trap logging: level informational (6), 89 message lines logged
        Logging to 192.168.1.100  (udp port 514, audit disabled,
              link up), 89 message lines logged,
              0 message lines rate-limited,
              0 message lines dropped-by-MD,
              xml disabled, sequence number disabled
              filtering disabled
        Logging Source-Interface:       VRF Name:
```

## 🔧 로그 관리 및 최적화

### 로그 회전 및 관리
```cisco
# 로그 버퍼 크기 조정
Router(config)# logging buffered 16384  # 16KB로 증가

# 로그 버퍼 지우기
Router# clear logging

# 로그 아카이브 설정 (일부 플랫폼)
Router(config)# archive
Router(config-archive)# log config
Router(config-archive-log-cfg)# logging enable
Router(config-archive-log-cfg)# logging size 50
```

### 로그 필터링
```cisco
# 메시지 차별자 (Message Discriminator) 설정
Router(config)# logging discriminator FILTER facility drops local0

# 특정 메시지 필터링 설정
Router(config)# logging filter filter_name
Router(config)# no logging console
Router(config)# logging console filtered
```

### 성능 최적화
```cisco
# 로깅 속도 제한 설정
Router(config)# logging rate-limit 100  # 초당 100개 메시지 제한

# 로깅 큐 크기 조정
Router(config)# logging queue-limit 1000

# 로깅 버퍼 크기 최적화
Router(config)# logging buffered 32768  # 32KB
```

## 🛡️ 보안 고려사항

### 로그 보안
```cisco
# 보안 이벤트 로깅
Router(config)# logging facility local4
Router(config)# logging trap warnings

# 인증 실패 로깅
Router(config)# login on-failure log
Router(config)# login on-success log

# 설정 변경 로깅
Router(config)# archive
Router(config-archive)# log config
Router(config-archive-log-cfg)# logging enable
```

### 로그 무결성
```
보안 위험:
❌ 로그 변조 가능성
❌ 네트워크 전송 중 도청
❌ 서버 침해 시 로그 손실

보안 강화 방법:
✅ 중앙집중식 로그 서버
✅ 로그 서버 접근 제어
✅ 정기적인 로그 백업
✅ 로그 무결성 검증
✅ 암호화된 전송 (TLS Syslog)
```

## 📊 중앙집중식 로깅

### Syslog 서버 구성
```
서버 소프트웨어:
- rsyslog (Linux)
- syslog-ng (다중 플랫폼)
- Splunk (엔터프라이즈)
- ELK Stack (Elasticsearch, Logstash, Kibana)
- Graylog (오픈소스)

서버 설정 예시 (rsyslog):
# /etc/rsyslog.conf
$ModLoad imudp
$UDPServerRun 514
$UDPServerAddress 192.168.1.100

# 네트워크 장비별 로그 분리
local0.*    /var/log/routers.log
local1.*    /var/log/switches.log
local2.*    /var/log/firewalls.log
```

### 로그 분석 및 모니터링
```
분석 도구:
1. 로그 집계 및 검색
   - 키워드 기반 검색
   - 시간 범위 필터링
   - 장비별 필터링

2. 알림 시스템
   - 임계 이벤트 실시간 알림
   - 패턴 기반 알림
   - 에스컬레이션 체계

3. 리포팅
   - 정기적인 로그 요약
   - 트렌드 분석
   - 규정 준수 리포트
```

## 🔧 문제해결에서의 활용

### 일반적인 문제 진단
```cisco
# 인터페이스 문제 추적
Router# show logging | include UPDOWN
%LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to down
%LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to up

# 라우팅 프로토콜 문제
Router# show logging | include OSPF
%OSPF-5-ADJCHG: Process 1, Nbr 192.168.1.2 on GigabitEthernet0/1 from FULL to DOWN

# 인증 실패 추적
Router# show logging | include Authentication
%SEC_LOGIN-1-QUIET_MODE_ON: Still timeleft for watching failures is 30 secs
```

### 로그 분석 기법
```
1. 시간 순서 분석
   - 이벤트 발생 순서 추적
   - 연관 이벤트 식별
   - 근본 원인 파악

2. 패턴 분석
   - 반복되는 오류 식별
   - 주기적 문제 발견
   - 성능 저하 추세 파악

3. 상관관계 분석
   - 여러 장비 간 연관성
   - 네트워크 전체 이벤트 연계
   - 복합적 문제 원인 분석
```

## 📋 모범 사례

### 설계 권장사항
```
1. 로깅 정책 수립
   ✅ 로깅 레벨 표준화
   ✅ 로그 보존 정책
   ✅ 접근 권한 관리

2. 인프라 설계
   ✅ 중앙집중식 로그 서버
   ✅ 로그 서버 이중화
   ✅ 네트워크 분리 (관리 VLAN)

3. 모니터링 체계
   ✅ 실시간 알림 시스템
   ✅ 대시보드 구성
   ✅ 자동화된 분석
```

### 운영 가이드라인
```
1. 정기적인 검토
   - 로그 레벨 적정성 검토
   - 불필요한 로그 식별
   - 스토리지 사용량 모니터링

2. 백업 및 아카이브
   - 정기적인 로그 백업
   - 장기 보존 정책
   - 규정 준수 요구사항 충족

3. 보안 관리
   - 로그 접근 권한 관리
   - 무결성 검증
   - 감사 추적 체계
```

## 🔗 관련 주제
- [[NTP|NTP]] - 정확한 시간 동기화
- [[SNMP|SNMP]] - 네트워크 관리 프로토콜
- [[문제해결 방법론|문제해결]] - 로그를 활용한 문제해결
- [[네트워크 보안|보안]] - 보안 이벤트 로깅

## 🏷️ 태그
#ccna #syslog #로깅 #네트워크모니터링 #문제해결 #보안감사 #시스템관리 #네트워크관리