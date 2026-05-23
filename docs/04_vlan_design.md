# 04. VLAN Design

## VLAN 설계 목적

본 문서는 RE:NET Enterprise Network Lab에서 사용할 VLAN 구조와 VLAN별 통신 정책을 정리한 문서이다.

VLAN은 하나의 물리적 네트워크를 여러 개의 논리적 네트워크로 분리하기 위해 사용한다.

이를 통해 부서별 네트워크를 분리하고, 보안 정책을 독립적으로 적용할 수 있다.

---

## VLAN 구성 개요

RE:NET Corp의 네트워크는 다음과 같은 VLAN으로 구성한다.

| VLAN ID | 이름 | 용도 | 네트워크 대역 |
|---:|---|---|---|
| 10 | DEV | 개발팀 네트워크 | 10.10.10.0/24 |
| 20 | HR | 인사/관리팀 네트워크 | 10.10.20.0/24 |
| 30 | GUEST | 게스트 네트워크 | 10.10.30.0/24 |
| 40 | SERVER | 서버존 | 10.10.40.0/24 |
| 99 | MGMT | 네트워크 관리망 | 10.10.99.0/24 |

---

## VLAN별 역할

### VLAN 10 - DEV

DEV VLAN은 개발팀 사용자를 위한 네트워크이다.

개발팀 사용자는 내부 Web Server, DNS Server 등 업무에 필요한 서버에 접근할 수 있다.

다만 네트워크 장비의 관리망이나 인사팀 네트워크에는 기본적으로 접근할 수 없도록 제한한다.

---

### VLAN 20 - HR

HR VLAN은 인사/관리팀 사용자를 위한 네트워크이다.

인사/관리팀 사용자는 내부 업무 시스템과 DNS Server에 접근할 수 있다.

개발팀 네트워크나 관리망에는 기본적으로 접근하지 못하도록 제한한다.

---

### VLAN 30 - GUEST

GUEST VLAN은 외부 방문자 또는 개인 기기를 위한 네트워크이다.

게스트 사용자는 인터넷 접근만 허용한다.

내부 서버존, 개발팀, 인사팀, 관리망으로의 접근은 차단한다.

---

### VLAN 40 - SERVER

SERVER VLAN은 내부 서버가 위치하는 네트워크이다.

Web Server, DNS Server, DHCP Server, Monitoring Server, Log Server가 이 영역에 배치된다.

사용자망에서 서버존으로 접근할 때는 필요한 서비스만 허용한다.

---

### VLAN 99 - MGMT

MGMT VLAN은 네트워크 장비를 관리하기 위한 전용 네트워크이다.

라우터, 스위치, 방화벽 등의 관리 IP가 이 VLAN에 배치된다.

SSH, SNMP 등 관리 목적의 접근만 허용하고, 일반 사용자와 게스트 사용자의 접근은 차단한다.

---

## VLAN별 Gateway

각 VLAN의 Gateway는 L3 Core Switch 또는 라우팅 장비에서 담당한다.

| VLAN ID | 이름 | Gateway |
|---:|---|---|
| 10 | DEV | 10.10.10.1 |
| 20 | HR | 10.10.20.1 |
| 30 | GUEST | 10.10.30.1 |
| 40 | SERVER | 10.10.40.1 |
| 99 | MGMT | 10.10.99.1 |

---

## Access Port 설계

Access Port는 사용자 단말이나 서버가 연결되는 포트이다.

각 포트는 하나의 VLAN에 소속된다.

| 연결 대상 | VLAN | 설명 |
|---|---:|---|
| 개발팀 PC | 10 | DEV VLAN에 연결 |
| 인사팀 PC | 20 | HR VLAN에 연결 |
| 게스트 사용자 | 30 | GUEST VLAN에 연결 |
| Web/DNS/DHCP Server | 40 | SERVER VLAN에 연결 |
| 네트워크 장비 관리 인터페이스 | 99 | MGMT VLAN에 연결 |

---

## Trunk Port 설계

Trunk Port는 여러 VLAN 트래픽을 하나의 링크로 전달하기 위해 사용한다.

스위치와 스위치 사이, L3 Core Switch와 Access Switch 사이의 연결은 Trunk로 구성한다.

| 연결 구간 | 포트 유형 | 허용 VLAN |
|---|---|---|
| L3 Core Switch ↔ Access Switch 1 | Trunk | 10, 20, 99 |
| L3 Core Switch ↔ Access Switch 2 | Trunk | 30, 99 |
| L3 Core Switch ↔ Server Switch | Trunk | 40, 99 |
| L3 Core Switch ↔ Core Router | Routed Link 또는 Trunk | 구성 방식에 따라 결정 |

---

## VLAN 통신 정책

VLAN 간 통신은 기본적으로 제한하고, 필요한 통신만 허용한다.

| 출발 VLAN | 목적 VLAN | 허용 여부 | 설명 |
|---|---|---|---|
| DEV | SERVER | 제한 허용 | Web, DNS 등 필요한 서비스만 허용 |
| HR | SERVER | 제한 허용 | 업무에 필요한 서비스만 허용 |
| GUEST | Internet | 허용 | 게스트 사용자의 인터넷 사용 |
| GUEST | SERVER | 차단 | 내부 서버 보호 |
| GUEST | MGMT | 차단 | 네트워크 장비 보호 |
| DEV | HR | 차단 | 부서 간 불필요한 접근 차단 |
| HR | DEV | 차단 | 부서 간 불필요한 접근 차단 |
| MGMT | 전체 VLAN | 허용 | 관리자 운영 목적 |
| SERVER | MGMT | 차단 | 서버에서 장비 관리망으로 직접 접근 제한 |

---

## Inter-VLAN Routing 설계

서로 다른 VLAN 간 통신이 필요한 경우 Inter-VLAN Routing을 사용한다.

예를 들어 DEV VLAN의 사용자가 SERVER VLAN의 Web Server에 접근하려면 L3 Core Switch 또는 라우터에서 VLAN 간 라우팅을 수행해야 한다.

하지만 모든 VLAN 간 통신을 허용하지는 않는다.

라우팅은 가능하더라도 Firewall 또는 ACL 정책을 통해 필요한 통신만 허용한다.

---

## VLAN 설계 이유

### 1. 보안 강화

게스트망, 사용자망, 서버망, 관리망을 분리하여 불필요한 접근을 차단한다.

특히 GUEST VLAN은 내부 서버와 관리망에 접근할 수 없도록 설계한다.

---

### 2. 장애 범위 축소

VLAN을 분리하면 특정 부서나 구간에서 문제가 발생하더라도 전체 네트워크에 미치는 영향을 줄일 수 있다.

예를 들어 게스트망에서 브로드캐스트 트래픽이 증가하더라도 개발팀이나 서버존에는 직접적인 영향을 줄일 수 있다.

---

### 3. 관리 편의성 향상

VLAN 번호와 IP 대역을 일치시켜 관리자가 네트워크 구조를 쉽게 파악할 수 있도록 설계하였다.

예를 들어 VLAN 10은 10.10.10.0/24, VLAN 20은 10.10.20.0/24 대역을 사용한다.

---

### 4. 정책 적용 용이

VLAN 단위로 방화벽 정책이나 ACL을 적용할 수 있기 때문에 보안 정책을 명확하게 관리할 수 있다.

예를 들어 GUEST VLAN에서 SERVER VLAN으로 가는 트래픽은 전체 차단하고, DEV VLAN에서 Web Server로 가는 HTTP/HTTPS 트래픽만 허용할 수 있다.

---

## 정리

본 VLAN 설계는 기업 네트워크에서 필요한 부서별 분리, 서버존 보호, 게스트망 격리, 관리망 보호를 목표로 한다.

향후 실제 구성 단계에서는 이 문서를 기준으로 Access Port, Trunk Port, Inter-VLAN Routing, ACL 정책을 구현한다.