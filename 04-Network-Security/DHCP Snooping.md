# DHCP Snooping

## 📝 개요
DHCP Snooping은 스위치에서 DHCP 트래픽을 모니터링하고 제어하여 불법 DHCP 서버와 DHCP 관련 공격을 방지하는 보안 기능입니다.

## 🎯 DHCP 공격 유형

### Rogue DHCP Server Attack
```
공격 시나리오:
1. 공격자가 불법 DHCP 서버 설치
2. 클라이언트가 불법 서버로부터 IP 할당 받음
3. 잘못된 게이트웨이/DNS 정보 수신
4. 트래픽이 공격자를 경유 (Man-in-the-Middle)

결과:
❌ 트래픽 도청
❌ 피싱 공격
❌ 네트워크 접근 불가
❌ 데이터 유출
```

### DHCP Starvation Attack
```
공격 시나리오:
1. 공격자가 대량의 DHCP 요청 전송
2. DHCP 서버의 IP 풀 고갈
3. 정상 클라이언트가 IP 할당 받지 못함

결과:
❌ 서비스 거부 (DoS)
❌ 네트워크 접근 불가
❌ 생산성 저하
```

### DHCP Spoofing
```
공격 시나리오:
1. 공격자가 DHCP 응답 위조
2. 빠른 응답으로 정상 서버보다 먼저 도달
3. 악의적인 네트워크 설정 배포

결과:
❌ 잘못된 DNS 서버 할당
❌ 트래픽 리다이렉션
❌ 인증 정보 탈취
```

## 🛡️ DHCP Snooping 동작 원리

### 포트 분류
#### Trusted Port
```
특징:
- DHCP 서버 메시지 송수신 허용
- DHCP Offer, ACK, NAK 메시지 허용
- 일반적으로 업링크 포트 또는 서버 포트

설정:
- 라우터 연결 포트
- DHCP 서버 연결 포트
- 다른 스위치 연결 포트 (트렁크)
```

#### Untrusted Port (기본값)
```
특징:
- DHCP 클라이언트 메시지만 허용
- DHCP Discover, Request 메시지 허용
- DHCP 서버 메시지 차단
- 일반적으로 클라이언트 연결 포트

설정:
- PC 연결 포트
- 프린터, IP 폰 등 엔드 디바이스
- 액세스 포트
```

### 검증 과정
```
1. DHCP 메시지 수신
2. 포트 신뢰성 확인
3. 메시지 유형 검증
4. Rate Limiting 적용
5. 바인딩 테이블 업데이트
6. 합법적 메시지만 포워딩
```

## ⚙️ DHCP Snooping 설정

### 기본 설정
```cisco
# DHCP Snooping 전역 활성화
Switch(config)# ip dhcp snooping

# VLAN별 DHCP Snooping 활성화
Switch(config)# ip dhcp snooping vlan 10,20,30

# 신뢰할 수 있는 포트 설정
Switch(config)# interface gi0/24
Switch(config-if)# ip dhcp snooping trust

# 클라이언트 포트는 기본적으로 untrusted (설정 불필요)
```

### 고급 설정
```cisco
# Rate Limiting 설정 (untrusted 포트)
Switch(config)# interface range fa0/1-20
Switch(config-if-range)# ip dhcp snooping limit rate 15

# 데이터베이스 URL 설정 (바인딩 테이블 저장)
Switch(config)# ip dhcp snooping database tftp://192.168.1.100/dhcp-snooping-db
Switch(config)# ip dhcp snooping database timeout 300

# 글로벌 Rate Limiting
Switch(config)# ip dhcp snooping limit rate 100

# 정보 옵션 삽입 비활성화 (호환성 문제 해결)
Switch(config)# no ip dhcp snooping information option
```

### VLAN별 세부 설정
```cisco
# 특정 VLAN에서만 활성화
Switch(config)# ip dhcp snooping vlan 10
Switch(config)# ip dhcp snooping vlan 20

# 여러 VLAN 동시 설정
Switch(config)# ip dhcp snooping vlan 10,20,30-35,40

# 모든 VLAN에서 활성화 (권장하지 않음)
Switch(config)# ip dhcp snooping vlan 1-4094
```

## 📊 DHCP Snooping 확인 및 모니터링

### 확인 명령어
```cisco
# DHCP Snooping 상태 확인
Switch# show ip dhcp snooping

# 바인딩 테이블 확인
Switch# show ip dhcp snooping binding

# 통계 정보 확인
Switch# show ip dhcp snooping statistics

# 데이터베이스 상태 확인
Switch# show ip dhcp snooping database
```

### 바인딩 테이블 출력 예시
```
Switch# show ip dhcp snooping binding
MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
------------------  ---------------  ----------  -------------  ----  --------------------
00:1B:44:11:3A:B7   192.168.1.100    86400       dhcp-snooping   10   FastEthernet0/1
00:1C:58:22:4B:C8   192.168.1.101    86400       dhcp-snooping   10   FastEthernet0/2
00:1D:60:33:5C:D9   192.168.1.102    86400       dhcp-snooping   20   FastEthernet0/3
Total number of bindings: 3
```

### 통계 정보 해석
```
Switch# show ip dhcp snooping statistics
Packets Forwarded                    = 245
Packets Dropped                      = 12
Untrusted Port Drops                 = 8
Untrusted Port Server Drops          = 4
Interface                Packets Dropped
FastEthernet0/5          8
FastEthernet0/10         4

분석:
- 정상적인 패킷: 245개
- 차단된 패킷: 12개 (의심스러운 활동)
- Untrusted 포트에서 서버 메시지 차단: 4개
```

## 🔧 관련 보안 기능

### Dynamic ARP Inspection과 연동
```cisco
# DHCP Snooping 바인딩 테이블을 DAI에서 활용
Switch(config)# ip arp inspection vlan 10,20

# DHCP Snooping 없이는 정적 ARP ACL 필요
Switch(config)# arp access-list STATIC_ARP
Switch(config-arp-nacl)# permit ip host 192.168.1.100 mac host 0011.2233.4455
```

### IP Source Guard와 연동
```cisco
# DHCP Snooping 바인딩 기반 IP Source Guard
Switch(config)# interface fa0/1
Switch(config-if)# ip verify source

# MAC 주소 검증 추가
Switch(config-if)# ip verify source port-security
```

### Port Security와 함께 사용
```cisco
Switch(config)# interface fa0/1
# Port Security 설정
Switch(config-if)# switchport port-security
Switch(config-if)# switchport port-security maximum 1

# DHCP Snooping은 자동으로 연동됨
```

## 🚨 문제해결

### 일반적인 문제들

#### 1. 정상 DHCP 서버 차단
```
증상: 클라이언트가 IP 할당 받지 못함
원인: DHCP 서버 포트가 untrusted로 설정

확인:
Switch# show ip dhcp snooping
Switch# show interfaces gi0/24 | include trust

해결:
Switch(config)# interface gi0/24
Switch(config-if)# ip dhcp snooping trust
```

#### 2. Rate Limiting으로 인한 차단
```
증상: 간헐적으로 DHCP 실패
원인: Rate Limiting 값이 너무 낮음

확인:
Switch# show ip dhcp snooping statistics

해결:
Switch(config)# interface fa0/1
Switch(config-if)# ip dhcp snooping limit rate 50  # 값 증가
```

#### 3. 정보 옵션으로 인한 호환성 문제
```
증상: DHCP 클라이언트 오류 발생
원인: Option 82 정보 삽입으로 인한 문제

해결:
Switch(config)# no ip dhcp snooping information option
```

#### 4. 바인딩 테이블 손실
```
증상: 재부팅 후 바인딩 정보 사라짐
원인: 데이터베이스 설정 누락

해결:
Switch(config)# ip dhcp snooping database flash:dhcp-snooping-db
Switch(config)# ip dhcp snooping database timeout 300
```

### 디버깅
```cisco
# DHCP Snooping 디버그
Switch# debug ip dhcp snooping event
Switch# debug ip dhcp snooping packet

# 특정 인터페이스만 디버그
Switch# debug ip dhcp snooping packet interface fa0/1

# 디버그 비활성화
Switch# undebug all
```

## 📋 모범 사례

### 설계 원칙
```
1. 포트 분류 원칙
   ✅ 업링크/서버 포트: Trusted
   ✅ 클라이언트 포트: Untrusted
   ✅ 트렁크 포트: 신중하게 판단

2. Rate Limiting 설정
   ✅ 클라이언트 포트: 5-15 pps
   ✅ 서버 포트: Rate Limiting 미적용
   ✅ 업링크 포트: 높은 값 또는 미적용

3. 데이터베이스 관리
   ✅ 정기적인 백업
   ✅ 적절한 timeout 설정
   ✅ 안정적인 저장 위치
```

### 배포 가이드라인
```
1. 단계별 배포
   - 테스트 VLAN부터 시작
   - 점진적으로 확장
   - 모니터링 체계 구축

2. 예외상황 대비
   - 바이패스 절차 수립
   - 백업 설정 준비
   - 롤백 계획 수립

3. 문서화
   - 신뢰할 수 있는 포트 목록
   - 설정 변경 이력
   - 운영 절차서
```

## 🔗 보안 강화 방안

### 다층 보안 구성
```cisco
# 1계층: DHCP Snooping
Switch(config)# ip dhcp snooping vlan 10

# 2계층: Dynamic ARP Inspection  
Switch(config)# ip arp inspection vlan 10

# 3계층: IP Source Guard
Switch(config)# interface fa0/1
Switch(config-if)# ip verify source

# 4계층: Port Security
Switch(config-if)# switchport port-security
```

### 모니터링 및 알림
```cisco
# SNMP 트랩 설정
Switch(config)# snmp-server enable traps dhcp

# Syslog 설정
Switch(config)# logging 192.168.1.200
Switch(config)# logging trap informational
```

## 📈 성능 고려사항

### 하드웨어 요구사항
```
메모리: 바인딩 테이블 저장용
CPU: 패킷 검사 및 처리
포워딩 성능: 소프트웨어 처리로 인한 지연

권장사항:
- 적절한 하드웨어 스펙 확인
- 성능 테스트 수행
- 모니터링 지표 설정
```

### 확장성 계획
```
고려사항:
- 클라이언트 수 증가
- VLAN 확장
- 바인딩 테이블 크기
- 데이터베이스 관리

대응방안:
- 계층적 설계
- 분산된 DHCP 서버
- 효율적인 VLAN 설계
```

## 🔗 관련 주제
- [[Dynamic ARP Inspection|DAI]] - ARP 테이블 보안
- [[IP Source Guard|IP Source Guard]] - IP 스푸핑 방지
- [[포트 보안|Port Security]] - MAC 주소 기반 보안
- [[VLAN 구성|VLAN]] - 네트워크 분할

## 🏷️ 태그
#ccna #dhcp-snooping #네트워크보안 #스위치보안 #dhcp #공격방지 #rogue-server