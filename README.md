# RE:NET Enterprise Network Lab

## 프로젝트 개요

RE:NET Enterprise Network Lab은 네트워크 엔지니어 취업 포트폴리오를 목적으로 진행하는 기업형 네트워크 인프라 구축 프로젝트입니다.

가상의 중소기업 환경을 기준으로 부서별 VLAN 분리, Inter-VLAN Routing, OSPF 내부 라우팅, BGP 기반 ISP 이중화, NAT/Firewall 정책, VPN 원격 접속, 모니터링, 자동화, 장애 대응 문서화를 단계적으로 구현합니다.

단순히 네트워크 장비를 연결하는 것이 아니라, 실제 네트워크 엔지니어 업무에서 필요한 설계, 구축, 운영, 장애 분석 과정을 경험하는 것을 목표로 합니다.

---

## 프로젝트 목표

이 프로젝트의 목표는 다음과 같습니다.

- 기업 네트워크 구조 설계 능력 습득
- IP 주소 설계 및 서브넷팅 이해
- VLAN 기반 부서망 분리
- Inter-VLAN Routing 구성
- OSPF 기반 내부 동적 라우팅 구성
- BGP 기반 외부 ISP 이중화 시나리오 구성
- NAT 및 Firewall 정책 설계
- 게스트망, 서버존, 관리망 분리
- VPN을 통한 원격 관리자 접속 구성
- Prometheus/Grafana 기반 네트워크 모니터링
- Ansible 기반 네트워크 설정 백업 및 자동화
- 장애 상황을 직접 만들고 원인 분석 및 복구 과정 문서화

---

## 가상 회사 시나리오

RE:NET Corp는 약 100명 규모의 중소기업입니다.

회사 내부에는 개발팀, 인사/관리팀, 게스트 사용자, 서버존, 네트워크 관리망이 존재합니다.

각 부서는 VLAN으로 분리되며, 게스트망은 내부 서버와 관리망에 접근할 수 없습니다.

관리자는 VPN을 통해 외부에서도 관리망에 접속할 수 있도록 구성할 예정입니다.

---

## 네트워크 구성 요소

| 구성 요소 | 역할 |
|---|---|
| Edge Router | 외부 인터넷 및 ISP와 연결되는 경계 라우터 |
| Firewall/NAT | 내부망 보호, NAT, 접근 제어 정책 적용 |
| Core Router 1 | 내부 핵심 라우터, OSPF 라우팅 담당 |
| Core Router 2 | 이중화 라우터, 장애 발생 시 우회 경로 제공 |
| L3 Core Switch | VLAN 간 라우팅 및 게이트웨이 역할 |
| Access Switch 1 | 개발팀, 인사팀 사용자 단말 연결 |
| Access Switch 2 | 게스트망 사용자 단말 연결 |
| Server Switch | 서버존 및 관리망 서버 연결 |
| Web Server | 내부 서비스 테스트용 웹 서버 |
| DNS/DHCP Server | 내부 DNS 및 DHCP 서비스 제공 |
| Monitoring Server | 네트워크 상태 수집 및 시각화 |

---

## VLAN 설계

| VLAN ID | 이름 | 용도 | 네트워크 대역 |
|---:|---|---|---|
| 10 | DEV | 개발팀 네트워크 | 10.10.10.0/24 |
| 20 | HR | 인사/관리팀 네트워크 | 10.10.20.0/24 |
| 30 | GUEST | 게스트 네트워크 | 10.10.30.0/24 |
| 40 | SERVER | 서버존 | 10.10.40.0/24 |
| 99 | MGMT | 네트워크 관리망 | 10.10.99.0/24 |

---

## 주요 구현 예정 기능

### 1. 네트워크 설계

- 전체 토폴로지 설계
- IP 주소 계획 작성
- VLAN 설계
- 보안 정책 설계

### 2. 라우팅 및 스위칭

- VLAN 구성
- Inter-VLAN Routing 구성
- OSPF 내부 라우팅 구성
- BGP ISP 이중화 구성

### 3. 보안 및 외부 연결

- NAT 구성
- Firewall 정책 구성
- 게스트망 내부 접근 차단
- VPN 원격 접속 구성

### 4. 네트워크 서비스

- DHCP 서버 구성
- DNS 서버 구성
- Web Server 구성

### 5. 운영 및 자동화

- Prometheus/Grafana 모니터링 구성
- 장애 감지 대시보드 구성
- Ansible 기반 설정 백업
- Ansible 기반 상태 점검 자동화

### 6. 장애 대응

- VLAN 설정 오류 대응
- OSPF Neighbor Down 대응
- BGP ISP Failover 대응
- Firewall 정책 오류 대응
- DNS 장애 대응

---

## 프로젝트 디렉터리 구조

```text
renet-enterprise-network-lab/
├─ README.md
├─ docs/
│  ├─ 01_project_overview.md
│  ├─ 02_topology_design.md
│  ├─ 03_ip_address_plan.md
│  ├─ 04_vlan_design.md
│  └─ 05_security_policy.md
├─ topology/
│  └─ topology.md
├─ containerlab/
├─ automation/
├─ monitoring/
└─ troubleshooting/