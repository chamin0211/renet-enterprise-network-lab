# 02. Topology Design

## 전체 네트워크 구조

RE:NET Corp의 네트워크는 Edge, Firewall, Core, Access, Server 영역으로 나누어 설계한다.

전체 구조는 다음과 같다.

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