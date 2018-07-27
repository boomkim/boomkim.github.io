---
layout: post
title:  "[docker] overlay network bandwidth test"
date:   2018-07-27
categories: docker network multihost
---

앞의 포스트에 이어 overlay network를 테스트해보다가 bandwidth가 궁금해졌다. 아무래도 네트워크 레이어를 하나 더 만드는 것이니 네트워크 성능이 노드들 끼리의 통신보다는 안 좋을 것이라 예상됐다. 찾아보니 예전 docker에서는 많게는 50% 정도까지 성능저하가 있었던 것 같다. 

테스트 환경: AWS EC2 t2.medium <-> t2.small

테스트 방법: iperf3 을 사용했다. 

테스트 내용 

1. ec2끼리 대역폭 테스트 

```
[ec2-user@ip-10-0-0-31 ~]$ iperf3 -c 10.0.0.209 -t 30
Connecting to host 10.0.0.209, port 5201
[  4] local 10.0.0.31 port 56086 connected to 10.0.0.209 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-1.00   sec  83.7 MBytes   702 Mbits/sec    0    362 KBytes       
[  4]   1.00-2.00   sec  75.9 MBytes   636 Mbits/sec    0    362 KBytes       
[  4]   2.00-3.00   sec  87.3 MBytes   732 Mbits/sec   43    659 KBytes       
[  4]   3.00-4.00   sec  88.1 MBytes   739 Mbits/sec    0    659 KBytes       
[  4]   4.00-5.00   sec  87.5 MBytes   734 Mbits/sec   60    560 KBytes       
[  4]   5.00-6.00   sec  93.5 MBytes   784 Mbits/sec    0    560 KBytes       
[  4]   6.00-7.00   sec  92.5 MBytes   776 Mbits/sec    0    560 KBytes       
[  4]   7.00-8.00   sec  93.0 MBytes   780 Mbits/sec    0    560 KBytes       
[  4]   8.00-9.00   sec  89.6 MBytes   752 Mbits/sec    0    560 KBytes       
[  4]   9.00-10.00  sec  90.3 MBytes   757 Mbits/sec    0    720 KBytes       
[  4]  10.00-11.00  sec  90.7 MBytes   761 Mbits/sec    0    720 KBytes       
[  4]  11.00-12.00  sec  91.3 MBytes   766 Mbits/sec    0    720 KBytes       
[  4]  12.00-13.00  sec  95.1 MBytes   798 Mbits/sec    0    720 KBytes       
[  4]  13.00-14.00  sec  90.6 MBytes   760 Mbits/sec    0    720 KBytes       
[  4]  14.00-15.00  sec  91.2 MBytes   765 Mbits/sec    0    720 KBytes       
[  4]  15.00-16.00  sec  90.0 MBytes   755 Mbits/sec    0    720 KBytes       
[  4]  16.00-17.00  sec  92.3 MBytes   774 Mbits/sec    0    775 KBytes       
[  4]  17.00-18.00  sec  94.1 MBytes   789 Mbits/sec   87    624 KBytes       
[  4]  18.00-19.00  sec  91.2 MBytes   765 Mbits/sec    0    675 KBytes       
[  4]  19.00-20.00  sec  91.9 MBytes   771 Mbits/sec    0    732 KBytes       
[  4]  20.00-21.00  sec  91.3 MBytes   766 Mbits/sec    0    732 KBytes       
[  4]  21.00-22.00  sec  88.5 MBytes   742 Mbits/sec    0    732 KBytes       
[  4]  22.00-23.00  sec  89.9 MBytes   754 Mbits/sec    0    732 KBytes       
[  4]  23.00-24.00  sec  85.9 MBytes   720 Mbits/sec    0    732 KBytes       
[  4]  24.00-25.00  sec  92.6 MBytes   777 Mbits/sec    0    732 KBytes       
[  4]  25.00-26.00  sec  92.6 MBytes   777 Mbits/sec    0    732 KBytes       
[  4]  26.00-27.00  sec  90.9 MBytes   762 Mbits/sec   65    585 KBytes       
[  4]  27.00-28.00  sec  89.0 MBytes   747 Mbits/sec    0    585 KBytes       
[  4]  28.00-29.00  sec  90.0 MBytes   755 Mbits/sec    0    663 KBytes       
[  4]  29.00-30.00  sec  93.1 MBytes   781 Mbits/sec    0    663 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-30.00  sec  2.64 GBytes   756 Mbits/sec  255             sender
[  4]   0.00-30.00  sec  2.64 GBytes   756 Mbits/sec                  receiver

iperf Done.

```

대략 560Mbits/s 정도의 대역폭 

2. docker에서 overlay 모드로 만들었을 때의 노드끼리 대역폭 테스트

```
[ec2-user@ip-10-0-0-31 ~]$ docker exec -it 89c08bbec6d7 sh
# iperf3 -c 192.168.0.155 -t 30
Connecting to host 192.168.0.155, port 5201
[  4] local 192.168.0.154 port 47676 connected to 192.168.0.155 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-1.00   sec  92.0 MBytes   771 Mbits/sec    0   1.51 MBytes       
[  4]   1.00-2.00   sec  85.0 MBytes   713 Mbits/sec    0   1.51 MBytes       
[  4]   2.00-3.00   sec  86.4 MBytes   725 Mbits/sec    0   1.51 MBytes       
[  4]   3.00-4.00   sec  85.3 MBytes   716 Mbits/sec    0   1.59 MBytes       
[  4]   4.00-5.00   sec  84.7 MBytes   710 Mbits/sec    0   1.59 MBytes       
[  4]   5.00-6.00   sec  87.5 MBytes   734 Mbits/sec    0   1.59 MBytes       
[  4]   6.00-7.00   sec  87.8 MBytes   737 Mbits/sec    0   1.59 MBytes       
[  4]   7.00-8.00   sec  87.6 MBytes   735 Mbits/sec    0   1.59 MBytes       
[  4]   8.00-9.00   sec  85.5 MBytes   717 Mbits/sec    0   1.59 MBytes       
[  4]   9.00-10.00  sec  79.2 MBytes   665 Mbits/sec  250    631 KBytes       
[  4]  10.00-11.00  sec  75.5 MBytes   633 Mbits/sec    0    714 KBytes       
[  4]  11.00-12.00  sec  81.4 MBytes   683 Mbits/sec    0    795 KBytes       
[  4]  12.00-13.00  sec  86.2 MBytes   723 Mbits/sec    0    871 KBytes       
[  4]  13.00-14.00  sec  86.2 MBytes   723 Mbits/sec    0    943 KBytes       
[  4]  14.00-15.00  sec  83.5 MBytes   700 Mbits/sec    0   1008 KBytes       
[  4]  15.00-16.00  sec  83.9 MBytes   704 Mbits/sec    0   1.04 MBytes       
[  4]  16.00-17.00  sec  85.6 MBytes   718 Mbits/sec    0   1.10 MBytes       
[  4]  17.00-18.00  sec  88.8 MBytes   744 Mbits/sec    0   1.16 MBytes       
[  4]  18.00-19.00  sec  88.1 MBytes   739 Mbits/sec    0   1.21 MBytes       
[  4]  19.00-20.00  sec  87.9 MBytes   738 Mbits/sec    0   1.26 MBytes       
[  4]  20.00-21.00  sec  83.3 MBytes   699 Mbits/sec    0   1.31 MBytes       
[  4]  21.00-22.00  sec  84.5 MBytes   709 Mbits/sec    0   1.35 MBytes       
[  4]  22.00-23.00  sec  85.1 MBytes   714 Mbits/sec    0   1.39 MBytes       
[  4]  23.00-24.00  sec  87.8 MBytes   736 Mbits/sec    0   1.44 MBytes       
[  4]  24.00-25.00  sec  90.0 MBytes   755 Mbits/sec    0   1.48 MBytes       
[  4]  25.00-26.00  sec  86.1 MBytes   723 Mbits/sec    0   1.51 MBytes       
[  4]  26.00-27.00  sec  83.2 MBytes   698 Mbits/sec    0   1.51 MBytes       
[  4]  27.00-28.00  sec  81.5 MBytes   684 Mbits/sec    0   1.51 MBytes       
[  4]  28.00-29.00  sec  84.9 MBytes   712 Mbits/sec    0   1.51 MBytes       
[  4]  29.00-30.00  sec  84.1 MBytes   706 Mbits/sec  214    595 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-30.00  sec  2.50 GBytes   715 Mbits/sec  464             sender
[  4]   0.00-30.00  sec  2.50 GBytes   715 Mbits/sec                  receiver
```

약간의 손실만 있는걸로... 

