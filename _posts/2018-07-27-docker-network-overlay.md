---
layout: post
title:  "[docker] multihost networking - overlay"
date:   2018-07-27
categories: docker network multihost
---

## 도커 - multi-host networking: overlay



여러 호스트를 이용하는 도커가 swarm을 이룰경우 서로 통신을 하려면 네트웍이 필요하다. 이때 각각 다른 호스트에서 생성된 컨테이너 사이에 네트워크를 생성해주는 것이 바로 overlay 네트워크 이다. 

![그림1](/images/overlay-network.png)

*각각의 호스트에서 쓰고있는 네트워크와 상관없이 컨테이너 끼리의 사설망을 따로 만들어줌.*

예를들어 각각의 호스트 A가 10.0.0.3 B가 10.0.0.5 라는 사설 ip를 사용하고 있을때 overlay network로 전혀 다른 사설대역 192.168.0.0/24를 만들 수 있다. Docker Swarm 을 구성해놓은 상태에서 새로운 서비스를 생성할 때, 네트워크를 미리 만들어 둔 overlay 네트워크를 지정하면 overlay 네트워크를 사용하는 컨테이너가 각각 올라가고 해당 서브넷 내의 ip를 할당받는다. 

*<u>이때 주의해야 할 점은 tcp 2377, 7946, udp 7946, 4789 포트가 열려 있어야 한다.</u>*

<br><br>

### overlay 네트워크 생성하고 테스트해보기. 

**1.network 생성**

네트워크를 생성하고

` docker network create --driver=overlay --subnet=192.168.0.0/24 mynet `


네트워크 생성 확인
```
[ec2-user@ip-10-0-0-209 ~]$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
f2fd318ecc13        bridge              bridge              local
c9b6d1d75287        docker_gwbridge     bridge              local
54c54d828043        host                host                local
f400wz9js3cz        ingress             overlay             swarm
h919kze0tfxw        mynet               overlay             swarm
2e51e048bc49        none                null                local
``` 
mynet 이 생성되었다. <br><br>

**2.service 생성**

`docker service create --name overlaytest --network mynet --replicas 3 tutum/hello-world ` 

tutum에서 제공하는 helloworld 이미지로 테스트 <br><br>

**3.네트워크 테스트** 

master노드에서 ps 실행 및 ip 확인
```
[ec2-user@ip-10-0-0-209 ~]$ docker ps
CONTAINER ID        IMAGE                      COMMAND                  CREATED             STATUS              PORTS               NAMES
3bf7f2ad2f5d        tutum/hello-world:latest   "/bin/sh -c 'php-fpm…"   10 seconds ago      Up 9 seconds        80/tcp              hello.3.8h6n98b0pgqxmhamhlxz0fpnr
[ec2-user@ip-10-0-0-209 ~]$ docker exec -it 3bf7f2ad2f5d sh
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:C0:A8:00:95  
          inet addr:192.168.0.149  Bcast:192.168.0.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

```

worker노드에서 ps 실행 및 ip확인 및 ping 
```
[ec2-user@ip-10-0-0-31 ~]$ docker ps
CONTAINER ID        IMAGE                      COMMAND                  CREATED              STATUS              PORTS               NAMES
5eb91ab17651        tutum/hello-world:latest   "/bin/sh -c 'php-fpm…"   About a minute ago   Up About a minute   80/tcp              hello.2.usgapik37uc3qpd7tiirrsi5u
[ec2-user@ip-10-0-0-31 ~]$ docker exec -it 5eb91ab17651 sh
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:C0:A8:00:97  
          inet addr:192.168.0.151  Bcast:192.168.0.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 
/ # ping 192.168.0.149
PING 192.168.0.149 (192.168.0.149): 56 data bytes
64 bytes from 192.168.0.149: seq=0 ttl=255 time=0.534 ms
64 bytes from 192.168.0.149: seq=1 ttl=255 time=0.528 ms
64 bytes from 192.168.0.149: seq=2 ttl=255 time=0.492 ms
64 bytes from 192.168.0.149: seq=3 ttl=255 time=0.472 ms
64 bytes from 192.168.0.149: seq=4 ttl=255 time=0.497 ms
64 bytes from 192.168.0.149: seq=5 ttl=255 time=0.511 ms
64 bytes from 192.168.0.149: seq=6 ttl=255 time=0.456 ms
64 bytes from 192.168.0.149: seq=7 ttl=255 time=0.517 ms
64 bytes from 192.168.0.149: seq=8 ttl=255 time=0.491 ms
^C
--- 192.168.0.149 ping statistics ---
9 packets transmitted, 9 packets received, 0% packet loss
round-trip min/avg/max = 0.456/0.499/0.534 ms
/ # 

```

콘테이너 끼리 사설 대역으로 통신이 되는것을 확인할 수 있다.  <br><br>


### 참고자료
* 도커 공식 홈페이지 https://docs.docker.com/network/overlay/
* 도커에서 만든 오버레이 네트워킹 데모 https://www.youtube.com/watch?v=nGSNULpHHZc

