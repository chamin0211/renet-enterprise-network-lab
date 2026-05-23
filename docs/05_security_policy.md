# 05. Security Policy

## 보안 정책 설계 목적

본 문서는 RE:NET Enterprise Network Lab에서 적용할 기본 보안 정책을 정리한 문서이다.

기업 네트워크에서는 모든 통신을 허용하는 방식이 아니라, 필요한 통신만 허용하고 불필요한 접근은 차단해야 한다.

본 프로젝트에서는 최소 권한 원칙을 기준으로 VLAN 간 접근 제어, 게스트망 격리, 서버존 보호, 관리망 보호, 외부 접근 제한 정책을 설계한다.

---

## 보안 정책 기본 원칙

본 프로젝트의 보안 정책은 다음 원칙을 따른다.

1. 기본적으로 불필요한 통신은 차단한다.
2. 업무에 필요한 서비스만 허용한다.
3. 게스트망은 내부망에 접근할 수 없도록 격리한다.
4. 관리망은 관리자만 접근할 수 있도록 제한한다.
5. 서버존은 필요한 포트만 허용한다.
6. 외부 인터넷에서 내부망으로 직접 접근하는 것은 제한한다.
7. 원격 관리는 VPN을 통해서만 허용한다.

---

## 네트워크 구간별 보안 정책

| 구간 | 보안 정책 |
|---|---|
| DEV VLAN | 업무에 필요한 서버 접근만 허용 |
| HR VLAN | 업무에 필요한 서버 접근만 허용 |
| GUEST VLAN | 인터넷만 허용, 내부망 접근 차단 |
| SERVER VLAN | 필요한 서비스 포트만 허용 |
| MGMT VLAN | 관리자만 접근 가능 |
| Internet | 내부 관리망 접근 차단 |
| VPN User | 인증된 사용자만 관리망 접근 허용 |

---

## VLAN 간 접근 제어 정책

| 출발지 | 목적지 | 서비스 | 허용 여부 | 설명 |
|---|---|---|---|---|
| DEV | Web Server | HTTP/HTTPS | 허용 | 개발팀의 내부 웹 서비스 접근 |
| DEV | DNS Server | DNS | 허용 | 도메인 조회 필요 |
| DEV | DHCP Server | DHCP | 허용 | IP 자동 할당 필요 |
| DEV | Monitoring Server | HTTP/HTTPS | 차단 | 일반 사용자는 모니터링 접근 불필요 |
| DEV | MGMT | SSH/SNMP | 차단 | 네트워크 장비 보호 |
| HR | Web Server | HTTP/HTTPS | 허용 | 업무 시스템 접근 |
| HR | DNS Server | DNS | 허용 | 도메인 조회 필요 |
| HR | DHCP Server | DHCP | 허용 | IP 자동 할당 필요 |
| HR | DEV | Any | 차단 | 부서 간 불필요한 접근 차단 |
| GUEST | Internet | HTTP/HTTPS/DNS | 허용 | 게스트 인터넷 사용 |
| GUEST | SERVER | Any | 차단 | 내부 서버 보호 |
| GUEST | MGMT | Any | 차단 | 네트워크 장비 보호 |
| GUEST | DEV/HR | Any | 차단 | 내부 사용자망 보호 |
| MGMT | Network Devices | SSH/SNMP | 허용 | 관리자 운영 목적 |
| MGMT | SERVER | SSH/HTTP/HTTPS | 허용 | 서버 관리 목적 |
| SERVER | MGMT | Any | 차단 | 서버에서 장비 관리망 직접 접근 제한 |
| Internet | Web Server | HTTP/HTTPS | 제한 허용 | 외부 공개 서비스가 필요한 경우 |
| Internet | MGMT | Any | 차단 | 외부에서 관리망 직접 접근 금지 |
| VPN User | MGMT | SSH | 허용 | 원격 관리자 접속 |

---

## 게스트망 보안 정책

GUEST VLAN은 외부 방문자 또는 개인 기기가 사용하는 네트워크이다.

게스트망은 보안 위험이 높은 구간으로 간주한다.

따라서 게스트망에서는 인터넷 접근만 허용하고, 내부 서버존, 개발팀, 인사팀, 관리망으로의 접근은 모두 차단한다.

### 허용

- GUEST → Internet
- GUEST → DNS

### 차단

- GUEST → DEV
- GUEST → HR
- GUEST → SERVER
- GUEST → MGMT

게스트망 격리는 본 프로젝트에서 가장 중요한 보안 정책 중 하나이다.

---

## 서버존 보안 정책

SERVER VLAN에는 Web Server, DNS Server, DHCP Server, Monitoring Server, Log Server가 위치한다.

서버존은 사용자망보다 중요한 자원이 위치하는 구간이므로 필요한 서비스만 허용한다.

| 서버 | 허용 서비스 | 접근 허용 대상 |
|---|---|---|
| Web Server | HTTP/HTTPS | DEV, HR, 필요 시 Internet |
| DNS Server | DNS | DEV, HR, GUEST |
| DHCP Server | DHCP | DEV, HR, GUEST |
| Monitoring Server | HTTP/HTTPS, SNMP | MGMT |
| Log Server | Syslog | Network Devices, Server |

서버존에 대한 접근은 서비스 단위로 제한한다.

예를 들어 DEV VLAN 사용자는 Web Server에 접근할 수 있지만, Monitoring Server에는 접근할 수 없다.

---

## 관리망 보안 정책

MGMT VLAN은 네트워크 장비를 관리하기 위한 전용 네트워크이다.

라우터, 스위치, 방화벽의 관리 IP는 MGMT VLAN에 배치한다.

관리망 접근은 관리자 또는 VPN 사용자로 제한한다.

### 허용

- MGMT → Router SSH
- MGMT → Switch SSH
- MGMT → Firewall SSH
- MGMT → Network Devices SNMP
- VPN User → MGMT SSH

### 차단

- DEV → MGMT
- HR → MGMT
- GUEST → MGMT
- Internet → MGMT

관리망이 일반 사용자망에 노출되면 장비 설정 변경, 정보 유출, 서비스 장애로 이어질 수 있기 때문에 엄격하게 보호한다.

---

## 외부 인터넷 접근 정책

내부 사용자는 Firewall/NAT를 통해 외부 인터넷에 접근한다.

내부 사설 IP는 외부에서 직접 라우팅되지 않으므로 NAT를 통해 공인 IP로 변환한다.

### 내부에서 외부로 나가는 트래픽

| 출발지 | 목적지 | 정책 |
|---|---|---|
| DEV | Internet | 허용 |
| HR | Internet | 허용 |
| GUEST | Internet | 허용 |
| SERVER | Internet | 제한 허용 |
| MGMT | Internet | 제한 허용 |

### 외부에서 내부로 들어오는 트래픽

| 출발지 | 목적지 | 정책 |
|---|---|---|
| Internet | Web Server | HTTP/HTTPS만 제한 허용 |
| Internet | DNS Server | 차단 |
| Internet | MGMT | 차단 |
| Internet | DEV/HR | 차단 |

외부에서 내부로 들어오는 트래픽은 기본적으로 차단하며, 공개가 필요한 Web Server만 제한적으로 허용한다.

---

## VPN 보안 정책

원격 관리자는 VPN을 통해 회사 내부 관리망에 접근한다.

VPN을 사용하면 외부 인터넷에서 관리망으로 직접 접근하지 않고, 인증된 터널을 통해 안전하게 접속할 수 있다.

| 사용자 | 접근 가능 구간 | 허용 서비스 |
|---|---|---|
| Remote Admin | MGMT VLAN | SSH |
| Remote Admin | Monitoring Server | HTTP/HTTPS |
| Remote Admin | Network Devices | SSH/SNMP |

VPN 사용자는 인증된 관리자만 허용한다.

일반 외부 사용자는 VPN에 접근할 수 없다.

---

## 방화벽 정책 설계

Firewall은 내부망과 외부망 사이에서 트래픽을 제어한다.

본 프로젝트에서는 다음과 같은 역할을 수행한다.

- 내부망에서 외부 인터넷으로 나가는 NAT 처리
- 외부에서 내부망으로 들어오는 트래픽 차단
- 게스트망의 내부 접근 차단
- 서버존 접근 포트 제한
- VPN 트래픽 허용
- 관리망 보호

방화벽 정책은 기본적으로 다음 방식으로 설계한다.

```text
기본 정책: Deny All
필요한 통신만 Allow