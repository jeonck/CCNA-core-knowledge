# Cisco IOS 명령어

## 개요
Cisco IOS(Internetwork Operating System)는 Cisco 네트워크 장비의 운영체제입니다. 명령줄 인터페이스(CLI)를 통해 장비를 설정, 모니터링, 문제해결할 수 있습니다.

## 기본 모드 및 탐색

### 사용자 모드 (User EXEC)
```cisco
Router>
- 제한된 명령어만 사용 가능
- 주로 확인 명령어
- 설정 변경 불가
```

### 특권 모드 (Privileged EXEC)  
```cisco
Router#
- 모든 확인 명령어 사용 가능
- 디버그 및 관리 명령어
- 설정 모드로 진입 가능

# 진입 방법
Router> enable
Router# 
```

### 글로벌 설정 모드
```cisco
Router(config)#
- 전역 설정 명령어
- 라우팅 프로토콜, VLAN 등

# 진입 방법
Router# configure terminal
Router(config)#
```

### 인터페이스 설정 모드
```cisco
Router(config-if)#
- 특정 인터페이스 설정

# 진입 방법
Router(config)# interface fastethernet 0/0
Router(config-if)#
```

### 모드 간 이동
```cisco
# 이전 모드로 돌아가기
exit

# 특권 모드로 바로 이동
end
Ctrl+Z

# 글로벌 설정 모드로 이동 (특권 모드에서)
configure terminal
conf t
```

## 기본 장비 설정

### 초기 설정
```cisco
# 호스트명 변경
Router(config)# hostname R1

# Enable 패스워드 설정
R1(config)# enable password cisco123
R1(config)# enable secret cisco123  # 암호화됨 (권장)

# 콘솔 패스워드 설정
R1(config)# line console 0
R1(config-line)# password console123
R1(config-line)# login

# VTY 라인 설정 (Telnet/SSH)
R1(config)# line vty 0 4
R1(config-line)# password vty123
R1(config-line)# login

# 배너 설정
R1(config)# banner motd #
*************************************************
* Authorized Access Only                        *
* Unauthorized access is prohibited             *
*************************************************
#
```

### 시간 및 시간대 설정
```cisco
# 시간대 설정
R1(config)# clock timezone KST 9

# 수동 시간 설정
R1# clock set 15:30:00 Dec 15 2024

# NTP 설정
R1(config)# ntp server pool.ntp.org
```

## 인터페이스 설정

### 기본 인터페이스 설정
```cisco
# 이더넷 인터페이스
R1(config)# interface fastethernet 0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# description "LAN Interface"
R1(config-if)# no shutdown

# 시리얼 인터페이스
R1(config)# interface serial 0/0/0
R1(config-if)# ip address 10.1.1.1 255.255.255.252
R1(config-if)# description "WAN to R2"
R1(config-if)# clock rate 64000  # DCE인 경우만
R1(config-if)# no shutdown
```

### 루프백 인터페이스
```cisco
# 논리적 인터페이스 (항상 up 상태)
R1(config)# interface loopback 0
R1(config-if)# ip address 1.1.1.1 255.255.255.255
R1(config-if)# description "Router ID"
```

### 인터페이스 범위 설정
```cisco
# 여러 인터페이스 동시 설정
SW1(config)# interface range fastethernet 0/1-5
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 10
```

## 정보 확인 명령어

### 시스템 정보
```cisco
# 하드웨어 및 소프트웨어 정보
Router# show version

# 실행 중인 설정
Router# show running-config
Router# show run

# 저장된 설정  
Router# show startup-config
Router# show start

# 시스템 시간
Router# show clock

# 부팅 정보
Router# show boot
```

### 인터페이스 정보
```cisco
# 모든 인터페이스 요약
Router# show ip interface brief
Router# show interfaces status  # 스위치

# 특정 인터페이스 상세 정보
Router# show interfaces fastethernet 0/0

# 인터페이스 통계
Router# show interfaces fastethernet 0/0 | include packets
Router# show interfaces counters  # 스위치
```

### 네트워크 정보
```cisco
# 라우팅 테이블
Router# show ip route

# ARP 테이블
Router# show arp
Router# show ip arp

# CDP 정보
Router# show cdp neighbors
Router# show cdp neighbors detail

# LLDP 정보 (활성화된 경우)
Router# show lldp neighbors
```

## 라우팅 관련 명령어

### 정적 라우팅
```cisco
# 정적 경로 설정
Router(config)# ip route 192.168.2.0 255.255.255.0 192.168.1.2
Router(config)# ip route 0.0.0.0 0.0.0.0 192.168.1.1  # 기본 경로

# 정적 경로 확인
Router# show ip route static
```

### 동적 라우팅 프로토콜
```cisco
# RIP 설정
Router(config)# router rip
Router(config-router)# version 2
Router(config-router)# network 192.168.1.0
Router(config-router)# no auto-summary

# OSPF 설정
Router(config)# router ospf 1
Router(config-router)# router-id 1.1.1.1
Router(config-router)# network 192.168.1.0 0.0.0.255 area 0

# EIGRP 설정
Router(config)# router eigrp 100
Router(config-router)# network 192.168.1.0 0.0.0.255
```

## 스위치 관련 명령어

### VLAN 설정
```cisco
# VLAN 생성
SW1(config)# vlan 10
SW1(config-vlan)# name SALES

# 포트 VLAN 할당
SW1(config)# interface fastethernet 0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10

# 트렁크 설정
SW1(config)# interface fastethernet 0/24
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk allowed vlan 10,20,30
```

### VLAN 확인
```cisco
# VLAN 정보
SW1# show vlan brief
SW1# show vlan id 10

# 트렁크 포트 정보
SW1# show interfaces trunk

# MAC 주소 테이블
SW1# show mac address-table
SW1# show mac address-table vlan 10
```

## 연결성 테스트

### Ping 테스트
```cisco
# 기본 ping
Router# ping 8.8.8.8

# 확장 ping
Router# ping
Protocol [ip]: 
Target IP address: 192.168.2.1
Repeat count [5]: 10
Datagram size [100]: 1500
Timeout in seconds [2]: 5
Extended commands [n]: y
Source address or interface: 192.168.1.1

# 특정 인터페이스에서 ping
Router# ping 8.8.8.8 source loopback 0
```

### Traceroute
```cisco
# 경로 추적
Router# traceroute 8.8.8.8

# 확장 traceroute
Router# traceroute
Protocol [ip]: 
Target IP address: 8.8.8.8
Source address: 192.168.1.1
Probe count [3]: 3
Minimum Time to Live [1]: 1
Maximum Time to Live [30]: 30
```

### Telnet 테스트
```cisco
# Telnet 연결 테스트
Router# telnet 192.168.1.2

# 특정 포트 테스트
Router# telnet 192.168.1.2 80
```

## 파일 관리

### 설정 저장 및 복원
```cisco
# 실행 설정을 시작 설정으로 저장
Router# copy running-config startup-config
Router# copy run start
Router# write memory
Router# wr

# 설정 백업 (TFTP)
Router# copy running-config tftp://192.168.1.100/router-backup.cfg

# 설정 복원
Router# copy tftp://192.168.1.100/router-backup.cfg running-config
```

### 플래시 메모리 관리
```cisco
# 플래시 내용 확인
Router# show flash
Router# dir flash:

# 파일 삭제
Router# delete flash:old-file.bin

# 파일 복사
Router# copy tftp://192.168.1.100/new-ios.bin flash:
```

## 보안 관련 명령어

### 패스워드 보안
```cisco
# 패스워드 암호화
Router(config)# service password-encryption

# 강력한 패스워드 정책
Router(config)# security passwords min-length 8

# 로그인 실패 제한
Router(config)# login block-for 300 attempts 3 within 60
```

### SSH 설정
```cisco
# 도메인 이름 설정
Router(config)# ip domain-name company.com

# RSA 키 생성
Router(config)# crypto key generate rsa
How many bits in the modulus [512]: 1024

# SSH 활성화
Router(config)# ip ssh version 2
Router(config)# line vty 0 4
Router(config-line)# transport input ssh
```

### ACL 설정
```cisco
# Standard ACL
Router(config)# access-list 10 permit 192.168.1.0 0.0.0.255
Router(config)# access-list 10 deny any

# Extended ACL
Router(config)# access-list 100 permit tcp any host 192.168.2.10 eq 80
Router(config)# access-list 100 deny ip any any

# ACL 적용
Router(config)# interface fastethernet 0/0
Router(config-if)# ip access-group 10 in
```

## 모니터링 및 디버깅

### 로그 관리
```cisco
# 로깅 설정
Router(config)# logging on
Router(config)# logging buffered 16384
Router(config)# logging console warnings
Router(config)# logging host 192.168.1.100

# 로그 확인
Router# show logging
Router# show logging | include ERROR
```

### 디버그 명령어
```cisco
# IP 라우팅 디버그
Router# debug ip routing
Router# debug ip packet

# 인터페이스 디버그
Router# debug interface

# 프로토콜별 디버그
Router# debug ip rip
Router# debug ip ospf events
Router# debug ip eigrp

# 디버그 비활성화
Router# no debug all
Router# undebug all
```

### 성능 모니터링
```cisco
# CPU 사용률
Router# show processes cpu
Router# show processes cpu history

# 메모리 사용량
Router# show memory
Router# show memory summary

# 인터페이스 통계 초기화
Router# clear counters
Router# clear counters fastethernet 0/0
```

## 도움말 및 편의 기능

### 명령어 도움말
```cisco
# 사용 가능한 명령어 목록
Router# ?

# 특정 명령어 옵션
Router# show ?
Router# ping ?

# 부분 명령어 완성
Router# sh<Tab>  # show로 자동 완성
Router# conf<Tab> t  # configure terminal
```

### 편의 기능
```cisco
# 명령어 이력
Router# show history

# 터미널 설정
Router# terminal length 0      # 페이징 비활성화
Router# terminal monitor       # 디버그 메시지 보기

# 명령어 단축
Router# sh ip int br          # show ip interface brief
Router# conf t                # configure terminal
```

## 문제해결 명령어

### 일반적인 문제 진단
```cisco
# 연결성 문제
Router# ping 8.8.8.8
Router# traceroute 8.8.8.8
Router# show ip route
Router# show ip arp

# 인터페이스 문제
Router# show interfaces
Router# show ip interface brief
Router# show controllers serial 0/0/0

# 프로토콜 문제
Router# show ip protocols
Router# show ip ospf neighbor
Router# show ip eigrp neighbors
```

### 설정 비교
```cisco
# 실행 설정과 저장 설정 비교
Router# show archive config differences system:running-config system:startup-config
```

## 모범 사례

### 명령어 사용 팁
```
1. Tab 키로 자동 완성 활용
2. ? 키로 도움말 확인
3. 명령어 단축형 사용
4. history 명령어로 이전 명령 재사용
5. 중요한 설정 전 백업
6. 변경 후 즉시 저장
```

### 보안 고려사항
```
1. 강력한 패스워드 사용
2. SSH 사용, Telnet 비활성화
3. 불필요한 서비스 비활성화
4. 정기적인 로그 모니터링
5. 적절한 권한 관리
```

## 관련 주제
- [[문제해결 방법론]] - 체계적 문제해결
- [[라우팅 기초]] - 라우팅 명령어
- [[VLAN 구성]] - 스위치 명령어
- [[네트워크 보안]] - 보안 관련 명령어

## 태그
#ccna #cisco #ios #cli #명령어 #설정
