cat > docs/06_basic_routing_lab.md <<'EOF'
# 06. Basic Routing Lab

## 1. Lab Objective

이번 실습의 목표는 Containerlab을 이용해 서로 다른 네트워크 대역에 위치한 DEV 호스트와 SERVER 호스트를 구성하고, 가운데에 위치한 `core-r1` 노드가 라우터 역할을 수행하도록 설정하는 것이다.

이를 통해 다음 항목을 검증한다.

- 서로 다른 IP 대역 간 통신
- 기본 게이트웨이 설정
- 라우팅 테이블 확인
- Linux 기반 라우터의 IP forwarding 설정
- ping을 이용한 연결성 검증
- traceroute를 이용한 패킷 경로 확인

---

## 2. Lab Topology

```text
DEV PC                  Router                  SERVER
dev-host  ----------  core-r1  ----------  server-host

10.10.10.10/24      10.10.10.1/24
                    10.10.40.1/24        10.10.40.10/24
이번 구성에서는 dev-host와 server-host를 서로 다른 네트워크 대역에 배치하였다.

DEV Network: 10.10.10.0/24
SERVER Network: 10.10.40.0/24

두 네트워크는 직접 통신할 수 없기 때문에, 가운데에 위치한 core-r1이 Layer 3 라우터 역할을 수행한다.

3. IP Address Plan
Device	Interface	IP Address	Role
dev-host	eth1	10.10.10.10/24	DEV PC
core-r1	eth1	10.10.10.1/24	DEV Gateway
core-r1	eth2	10.10.40.1/24	SERVER Gateway
server-host	eth1	10.10.40.10/24	SERVER
4. Containerlab Topology File

실습을 위해 다음 Containerlab 토폴로지 파일을 작성하였다.

containerlab/renet-routing-basic.clab.yml

해당 파일에서는 총 3개의 Linux 컨테이너 노드를 구성하였다.

dev-host
core-r1
server-host

core-r1에는 두 개의 인터페이스를 설정하여 DEV 네트워크와 SERVER 네트워크 양쪽에 연결되도록 구성하였다.

5. Routing Design
dev-host

dev-host는 DEV 네트워크에 속하며, 다른 네트워크로 가는 트래픽은 기본 게이트웨이인 10.10.10.1로 전달한다.

default via 10.10.10.1 dev eth1
server-host

server-host는 SERVER 네트워크에 속하며, 다른 네트워크로 가는 트래픽은 기본 게이트웨이인 10.10.40.1로 전달한다.

default via 10.10.40.1 dev eth1
core-r1

core-r1은 두 네트워크에 직접 연결되어 있다.

10.10.10.0/24 dev eth1
10.10.40.0/24 dev eth2

또한 Linux 컨테이너가 라우터처럼 동작할 수 있도록 IP forwarding을 활성화하였다.

net.ipv4.ip_forward = 1
6. Verification
6.1 IP Address Verification

각 노드의 인터페이스 IP 설정을 확인하였다.

ip -4 addr show eth1

확인 결과 다음과 같이 IP가 정상적으로 설정되었다.

dev-host eth1      = 10.10.10.10/24
core-r1 eth1       = 10.10.10.1/24
core-r1 eth2       = 10.10.40.1/24
server-host eth1   = 10.10.40.10/24
6.2 Routing Table Verification

각 호스트의 라우팅 테이블을 확인하였다.

dev-host의 기본 게이트웨이:

default via 10.10.10.1 dev eth1

server-host의 기본 게이트웨이:

default via 10.10.40.1 dev eth1

core-r1의 직접 연결 네트워크:

10.10.10.0/24 dev eth1 proto kernel scope link src 10.10.10.1
10.10.40.0/24 dev eth2 proto kernel scope link src 10.10.40.1

이를 통해 dev-host와 server-host가 서로 다른 네트워크로 트래픽을 보낼 때 core-r1을 게이트웨이로 사용하도록 설정되었음을 확인하였다.

6.3 IP Forwarding Verification

core-r1에서 IP forwarding 설정을 확인하였다.

sysctl net.ipv4.ip_forward

결과:

net.ipv4.ip_forward = 1

해당 결과를 통해 core-r1이 한 인터페이스로 들어온 패킷을 다른 인터페이스로 전달할 수 있음을 확인하였다.

6.4 Gateway Ping Test

dev-host에서 기본 게이트웨이인 core-r1의 DEV 쪽 인터페이스로 ping 테스트를 수행하였다.

ping -c 4 10.10.10.1

결과:

4 packets transmitted, 4 received, 0% packet loss

이를 통해 dev-host와 core-r1 간 L3 연결이 정상임을 확인하였다.

6.5 End-to-End Ping Test

dev-host에서 server-host로 ping 테스트를 수행하였다.

ping -c 4 10.10.40.10

결과:

4 packets transmitted, 4 received, 0% packet loss

이를 통해 서로 다른 네트워크 대역인 10.10.10.0/24와 10.10.40.0/24 간 통신이 정상적으로 이루어짐을 확인하였다.

또한 응답 결과에서 ttl=63이 확인되었다. 동일 네트워크 게이트웨이 응답에서는 ttl=64가 확인되었고, 서버 응답에서는 TTL 값이 1 감소하였다. 이를 통해 패킷이 core-r1 라우터를 한 번 경유했음을 확인할 수 있었다.

6.6 Traceroute Test

dev-host에서 server-host까지의 경로를 확인하기 위해 traceroute를 수행하였다.

traceroute 10.10.40.10

결과:

1  10.10.10.1
2  10.10.40.10

Traceroute 결과를 통해 패킷이 다음 경로로 이동함을 확인하였다.

dev-host → core-r1 → server-host

즉, core-r1이 DEV 네트워크와 SERVER 네트워크 사이에서 라우터 역할을 정상적으로 수행하고 있음을 검증하였다.

7. Result Summary

이번 Basic Routing Lab에서는 Containerlab을 이용해 DEV 네트워크와 SERVER 네트워크를 분리하고, 가운데에 위치한 core-r1을 라우터로 구성하였다.

검증 결과는 다음과 같다.

각 노드의 IP 주소가 정상적으로 설정됨
dev-host와 server-host의 기본 게이트웨이가 정상적으로 설정됨
core-r1이 두 네트워크에 직접 연결되어 있음을 확인함
core-r1의 IP forwarding이 활성화됨
dev-host에서 server-host로 ping 통신 성공
traceroute를 통해 core-r1을 경유하는 경로 확인

이를 통해 Linux 기반 컨테이너를 활용하여 기본적인 Layer 3 라우팅 구조를 구현하고 검증하였다.

8. What I Learned

이번 실습을 통해 다음 내용을 학습하였다.

Containerlab을 이용한 네트워크 랩 구성 방법
Linux 컨테이너를 라우터처럼 사용하는 방법
IP 주소와 서브넷의 역할
기본 게이트웨이의 필요성
라우팅 테이블 확인 방법
IP forwarding의 의미
ping과 traceroute를 이용한 네트워크 검증 방법
TTL 값이 라우터 경유 여부를 판단하는 데 활용될 수 있다는 점
9. Portfolio Description

Containerlab 기반으로 DEV 네트워크와 SERVER 네트워크를 분리하여 구성하고, Linux 기반 core-r1 노드에 IP forwarding을 적용해 기본 Layer 3 라우팅을 구현하였다. 이후 라우팅 테이블, 기본 게이트웨이, ping, traceroute를 통해 서로 다른 네트워크 간 통신이 정상적으로 이루어지는 것을 검증하였다.
EOF


그다음 제대로 만들어졌는지 확인해.

```bash
cat docs/06_basic_routing_lab.md

이거 실행한 화면 캡처 보내줘.
내용 확인한 다음에 README.md랑 topology/topology.md에 이번 단계 완료 내용 추가하자.