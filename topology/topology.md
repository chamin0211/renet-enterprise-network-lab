# Network Topology

## RE:NET Enterprise Network Lab Topology

본 문서는 RE:NET Enterprise Network Lab의 전체 네트워크 토폴로지를 정리한 문서이다.

이 토폴로지는 가상의 중소기업 네트워크를 기준으로 Edge, Firewall, Core, Access, Server 영역으로 나누어 설계하였다.

---

## 전체 토폴로지

```text
                         [ Remote Admin ]
                               |
                            VPN Tunnel
                               |
                        [ Internet / ISP ]
                               |
                          [ Edge Router ]
                               |
                         [ Firewall/NAT ]
                               |
                    +----------+----------+
                    |                     |
              [ Core Router 1 ]     [ Core Router 2 ]
                    |                     |
                    +----------+----------+
                               |
                         [ L3 Core Switch ]
                               |
        +----------------------+----------------------+
        |                      |                      |
 [ Access Switch 1 ]    [ Access Switch 2 ]    [ Server Switch ]
        |                      |                      |
  개발팀 / 인사팀          게스트망              서버존 / 관리망
```

---

## 네트워크 영역 설명

| 영역 | 설명 |
|---|---|
| Internet / ISP | 외부 인터넷 및 ISP 연결 구간 |
| Edge Router | 외부망과 내부망의 경계 라우터 |
| Firewall/NAT | 방화벽 정책과 NAT를 담당하는 보안 구간 |
| Core Router | 내부 라우팅과 장애 우회를 담당하는 핵심 라우팅 구간 |
| L3 Core Switch | VLAN 간 라우팅과 Gateway 역할 수행 |
| Access Switch | 사용자 단말이 연결되는 스위치 구간 |
| Server Switch | 서버와 관리 장비가 연결되는 구간 |
| Remote Admin | VPN을 통해 원격으로 접속하는 관리자 |

---

## VLAN 배치

| VLAN ID | 이름 | 배치 위치 | 네트워크 대역 |
|---:|---|---|---|
| 10 | DEV | Access Switch 1 | 10.10.10.0/24 |
| 20 | HR | Access Switch 1 | 10.10.20.0/24 |
| 30 | GUEST | Access Switch 2 | 10.10.30.0/24 |
| 40 | SERVER | Server Switch | 10.10.40.0/24 |
| 99 | MGMT | Server Switch / Network Devices | 10.10.99.0/24 |

---

## 주요 트래픽 흐름

### 1. 내부 사용자 인터넷 접속

```text
DEV/HR User
 → Access Switch
 → L3 Core Switch
 → Core Router
 → Firewall/NAT
 → Edge Router
 → Internet
```

### 2. 내부 사용자 서버 접속

```text
DEV/HR User
 → Access Switch
 → L3 Core Switch
 → Server Switch
 → Web/DNS Server
```

### 3. 게스트 사용자 인터넷 접속

```text
Guest User
 → Access Switch 2
 → L3 Core Switch
 → Firewall/NAT
 → Edge Router
 → Internet
```

게스트 사용자는 내부 서버존과 관리망으로 접근할 수 없다.

### 4. 원격 관리자 접속

```text
Remote Admin
 → VPN Tunnel
 → Firewall/VPN
 → MGMT VLAN
 → Network Devices
```

원격 관리자는 VPN 인증 후 관리망에 접근할 수 있다.

---

## 장애 대응 실습 포인트

본 토폴로지는 다음 장애 상황을 실습할 수 있도록 설계하였다.

| 장애 상황 | 확인할 내용 |
|---|---|
| Access Switch VLAN 설정 오류 | 특정 부서 사용자의 통신 불가 |
| Trunk 설정 오류 | 여러 VLAN의 통신 장애 |
| L3 Gateway 설정 오류 | Inter-VLAN Routing 실패 |
| Core Router 장애 | OSPF 경로 우회 여부 |
| Firewall 정책 오류 | 특정 서비스 접속 실패 |
| NAT 설정 오류 | 내부 사용자의 인터넷 접속 실패 |
| DNS Server 장애 | 도메인 조회 실패 |
| VPN 장애 | 원격 관리자 접속 실패 |

---

## 향후 구현 계획

이 토폴로지를 기준으로 다음 기능을 단계적으로 구현한다.

1. VLAN 및 IP 주소 구성
2. Inter-VLAN Routing 구성
3. OSPF 내부 라우팅 구성
4. BGP 기반 ISP 이중화 구성
5. NAT 및 Firewall 정책 구성
6. VPN 원격 접속 구성
7. DHCP/DNS/Web Server 구성
8. Prometheus/Grafana 모니터링 구성
9. Ansible 자동화 구성
10. 장애 대응 시나리오 문서화

---

## 정리

본 토폴로지는 단순한 연결 구조가 아니라, 네트워크 엔지니어에게 필요한 설계, 구축, 운영, 장애 대응 능력을 학습하기 위한 구조이다.

향후 실습 단계에서는 이 토폴로지를 기반으로 실제 가상 네트워크 환경을 구성하고, 각 기능을 검증한다.