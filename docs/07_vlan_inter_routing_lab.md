# 07. VLAN and Inter-VLAN Routing Lab

## 1. Lab Objective

이번 실습의 목표는 Containerlab을 이용해 VLAN 기반 네트워크 분리 구조를 구성하고, 서로 다른 VLAN 간 통신이 `core-r1` 라우터를 통해 이루어지는지 검증하는 것이다.

이전 Basic Routing Lab에서는 `dev-host`와 `server-host`가 `core-r1`에 직접 연결된 구조였다.

이번 실습에서는 중간에 `access-sw1` 스위치 역할의 노드를 추가하고, VLAN 10과 VLAN 40을 분리한 뒤 Router-on-a-Stick 방식으로 Inter-VLAN Routing을 구성하였다.

검증 목표는 다음과 같다.

- VLAN 10과 VLAN 40 분리
- Access Port와 Trunk Port 구성
- Linux bridge 기반 VLAN filtering 설정
- Router-on-a-Stick 구조 구성
- VLAN 서브인터페이스 생성
- Inter-VLAN Routing 검증
- ping과 traceroute를 통한 통신 경로 확인

---

## 2. Lab Topology

```text
              core-r1
                |
            trunk port
                |
dev-host --- access-sw1 --- server-host
 VLAN 10                    VLAN 40
```

구성 흐름은 다음과 같다.

```text
dev-host
  |
access-sw1 eth1  → VLAN 10 access port
  |
access-sw1 eth3  → VLAN 10, VLAN 40 trunk port
  |
core-r1 eth1.10  → VLAN 10 gateway
core-r1 eth1.40  → VLAN 40 gateway
  |
access-sw1 eth2  → VLAN 40 access port
  |
server-host
```

---

## 3. VLAN and IP Address Plan

| Device | Interface | VLAN | IP Address | Role |
|---|---|---:|---|---|
| dev-host | eth1 | 10 | 10.10.10.10/24 | DEV Host |
| server-host | eth1 | 40 | 10.10.40.10/24 | SERVER Host |
| access-sw1 | eth1 | 10 | - | VLAN 10 Access Port |
| access-sw1 | eth2 | 40 | - | VLAN 40 Access Port |
| access-sw1 | eth3 | 10, 40 | - | Trunk Port |
| core-r1 | eth1.10 | 10 | 10.10.10.1/24 | VLAN 10 Gateway |
| core-r1 | eth1.40 | 40 | 10.10.40.1/24 | VLAN 40 Gateway |

---

## 4. Containerlab Topology File

실습을 위해 다음 Containerlab 토폴로지 파일을 작성하였다.

```text
containerlab/renet-vlan-routing.clab.yml
```

해당 파일에서는 총 4개의 Linux 컨테이너 노드를 구성하였다.

- `dev-host`
- `server-host`
- `access-sw1`
- `core-r1`

각 노드의 역할은 다음과 같다.

| Node | Role |
|---|---|
| dev-host | VLAN 10에 속한 DEV PC |
| server-host | VLAN 40에 속한 SERVER |
| access-sw1 | VLAN access/trunk 포트를 구성하는 스위치 |
| core-r1 | VLAN 간 라우팅을 수행하는 라우터 |

---

## 5. VLAN Design

`access-sw1`은 Linux bridge를 사용하여 스위치처럼 동작하도록 구성하였다.

```text
br0 = Linux bridge
```

그리고 VLAN filtering을 활성화하여 포트별 VLAN을 분리하였다.

```text
eth1 = VLAN 10 access port
eth2 = VLAN 40 access port
eth3 = VLAN 10, VLAN 40 trunk port
```

VLAN 설정 확인 명령어는 다음과 같다.

```bash
bridge vlan show
```

확인 결과:

```text
port    vlan-id
eth1    10 PVID Egress Untagged
eth2    40 PVID Egress Untagged
eth3    10
        40
```

이를 통해 `dev-host`는 VLAN 10에, `server-host`는 VLAN 40에 연결되었으며, `core-r1`과 연결된 `eth3` 포트는 VLAN 10과 VLAN 40을 모두 전달하는 trunk 포트로 동작함을 확인하였다.

---

## 6. Router-on-a-Stick Design

`core-r1`은 `access-sw1`과 하나의 물리 인터페이스 `eth1`로 연결된다.

이 인터페이스 위에 VLAN별 서브인터페이스를 생성하였다.

```text
core-r1 eth1
├── eth1.10 → VLAN 10 Gateway → 10.10.10.1/24
└── eth1.40 → VLAN 40 Gateway → 10.10.40.1/24
```

확인 명령어:

```bash
ip -4 addr
```

확인 결과:

```text
eth1.10
inet 10.10.10.1/24 scope global eth1.10

eth1.40
inet 10.10.40.1/24 scope global eth1.40
```

이를 통해 `core-r1`이 VLAN 10과 VLAN 40의 기본 게이트웨이 역할을 수행하도록 구성되었음을 확인하였다.

---

## 7. Host IP and Routing Verification

### dev-host

`dev-host`는 VLAN 10에 속하며, IP 주소는 다음과 같다.

```text
10.10.10.10/24
```

라우팅 테이블 확인 결과:

```text
default via 10.10.10.1 dev eth1
10.10.10.0/24 dev eth1 proto kernel scope link src 10.10.10.10
```

이를 통해 `dev-host`가 다른 네트워크로 가는 트래픽을 VLAN 10 게이트웨이인 `10.10.10.1`로 전달하도록 설정되었음을 확인하였다.

### server-host

`server-host`는 VLAN 40에 속하며, IP 주소는 다음과 같다.

```text
10.10.40.10/24
```

라우팅 테이블 확인 결과:

```text
default via 10.10.40.1 dev eth1
10.10.40.0/24 dev eth1 proto kernel scope link src 10.10.40.10
```

이를 통해 `server-host`가 다른 네트워크로 가는 트래픽을 VLAN 40 게이트웨이인 `10.10.40.1`로 전달하도록 설정되었음을 확인하였다.

---

## 8. IP Forwarding Verification

`core-r1`이 VLAN 간 패킷을 전달하려면 IP forwarding이 활성화되어 있어야 한다.

확인 명령어:

```bash
sysctl net.ipv4.ip_forward
```

결과:

```text
net.ipv4.ip_forward = 1
```

해당 결과를 통해 `core-r1`이 한 VLAN에서 들어온 패킷을 다른 VLAN으로 전달할 수 있음을 확인하였다.

---

## 9. Inter-VLAN Ping Test

`dev-host`에서 `server-host`로 ping 테스트를 수행하였다.

```bash
ping -c 4 10.10.40.10
```

결과:

```text
64 bytes from 10.10.40.10: icmp_seq=1 ttl=63
64 bytes from 10.10.40.10: icmp_seq=2 ttl=63
64 bytes from 10.10.40.10: icmp_seq=3 ttl=63
64 bytes from 10.10.40.10: icmp_seq=4 ttl=63

4 packets transmitted, 4 received, 0% packet loss
```

이를 통해 VLAN 10에 속한 `dev-host`와 VLAN 40에 속한 `server-host` 간 통신이 정상적으로 이루어짐을 확인하였다.

`ttl=63`으로 확인된 점을 통해 응답 패킷이 `core-r1` 라우터를 한 번 경유했음을 확인할 수 있었다.

---

## 10. Traceroute Test

`dev-host`에서 `server-host`까지의 경로를 확인하기 위해 traceroute를 수행하였다.

```bash
traceroute 10.10.40.10
```

결과:

```text
traceroute to 10.10.40.10
1  10.10.10.1
2  10.10.40.10
```

경로는 다음과 같다.

```text
dev-host → access-sw1 → core-r1 → access-sw1 → server-host
```

Traceroute 결과에서 첫 번째 홉이 `10.10.10.1`로 확인되었다.

이는 `dev-host`의 기본 게이트웨이인 `core-r1 eth1.10`을 경유했다는 의미이다.

따라서 VLAN 10과 VLAN 40 사이의 트래픽이 `core-r1`을 통해 라우팅되고 있음을 확인하였다.

---

## 11. Result Summary

이번 VLAN and Inter-VLAN Routing Lab에서는 Linux 컨테이너 기반으로 VLAN 분리 구조를 구성하고, Router-on-a-Stick 방식으로 VLAN 간 통신을 구현하였다.

검증 결과는 다음과 같다.

- `access-sw1`에서 VLAN 10과 VLAN 40이 분리됨
- `access-sw1 eth1`은 VLAN 10 access port로 설정됨
- `access-sw1 eth2`는 VLAN 40 access port로 설정됨
- `access-sw1 eth3`은 VLAN 10과 VLAN 40을 전달하는 trunk port로 설정됨
- `core-r1`에 VLAN 서브인터페이스 `eth1.10`, `eth1.40`이 생성됨
- `core-r1`이 VLAN 10과 VLAN 40의 게이트웨이 역할을 수행함
- `dev-host`와 `server-host`의 기본 게이트웨이가 정상적으로 설정됨
- `dev-host`에서 `server-host`로 ping 통신 성공
- traceroute를 통해 `core-r1`을 경유하는 경로 확인

이를 통해 VLAN으로 분리된 네트워크 간 통신이 라우터를 통해 정상적으로 이루어지는 것을 검증하였다.

---

## 12. What I Learned

이번 실습을 통해 다음 내용을 학습하였다.

- VLAN을 이용한 네트워크 분리 구조
- Access Port와 Trunk Port의 차이
- Linux bridge의 VLAN filtering 설정 방법
- Router-on-a-Stick 구조
- VLAN 서브인터페이스 생성 방법
- Inter-VLAN Routing 동작 방식
- `bridge vlan show` 명령어를 이용한 VLAN 설정 확인
- `ping`과 `traceroute`를 이용한 VLAN 간 통신 검증

---

## 13. Portfolio Description

Containerlab 기반으로 VLAN 10 DEV 네트워크와 VLAN 40 SERVER 네트워크를 분리하고, Linux bridge 기반 `access-sw1`에서 access/trunk 포트를 구성하였다. 이후 `core-r1`에 VLAN 서브인터페이스를 생성하여 Router-on-a-Stick 방식의 Inter-VLAN Routing을 구현하였다. 최종적으로 ping과 traceroute를 통해 서로 다른 VLAN 간 통신이 `core-r1`을 경유하여 정상적으로 이루어지는 것을 검증하였다.
