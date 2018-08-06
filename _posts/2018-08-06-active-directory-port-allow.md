---
layout: post
title:  "[ActiveDirectory] 삽질중... dns 포트 허용"
date:   2018-08-06
categories: activedirectory dns aws 
---

[Active Directory](http://dianememory.tistory.com/5) 글을 보고 AWS 상에서 AD를 구성하는 것을 테스트해보고 있다. 하지만 이 글은 AWS의 인프라를 기준으로 작성되어 있지 한다. 여기저기 찾아봤으나 aws 상에서 AD를 구성하는 예가 많지 않다. 사실 쉽게 AWS에서 quickstart로 나와있어서 굳이 구성 예제가 필요하지 않았던 것 같다. 그래도 사정상 테스트를 해야했기에... 일단은 부딪혀보기로 했다. 

첫번재 AD를 만드는 데에는 아무 문제가 없었으나 두번째 AD를 만드는 과정에서 문제가 생겼다. example.com 으로 도메인을 만들었는데 두번째 윈도우 서버에서 그 도메인을 찾지를 못한다. nslookup으로 질의해보니 dns가 응답하지 않는다. 뭐가 문제인지 모르겠다. tcp 포트도 다 열려있는데... 그래서 그냥 모든 트래픽을 허용해버렸다. 된다. 

모든 트래픽을 열어주는건 찜찜해서 dns가 어떤 포트를 쓰는지 궁금해졌다. 뒤져보니 나랑 비슷한 질문을 한 글을 발견했다. ([질문: NSLookup, how to set to TCP from UDP?](https://social.technet.microsoft.com/Forums/windows/en-US/955244f1-867a-4f3b-a0a3-321f05982fa0/nslookup-how-to-set-to-tcp-from-udp?forum=win10itpronetworking))

결론: **그냥 udp 53 포트를 열어주면 되는 문제**였다... 질문을 열어보면 **dns는 주로 udp 53 포트를 사용하고 512 byte가 넘을때만 tcp를 사용한다**고 친절하고 내공있는 답변이 적혀있다. 세상은 넓고 고수는 많다. 어쨌든 AWS 상에서 수동으로 AD를 구성하고, 이전하는 것 까지 테스트를 해보고 긴 포스트를 올려볼 예정이다. 물론, 예정일 뿐이다. 

위키피디아에 [DNS 항목](https://en.wikipedia.org/wiki/Domain_Name_System#DNS_Protocol_transport)에도 다음과 같이 적혀있다. 
>> DNS primarily uses the User Datagram Protocol (UDP) on port number 53 to serve requests.[3] DNS queries consist of a single UDP request from the client followed by a single UDP reply from the server. When the length of the answer exceeds 512 bytes and both client and server support EDNS, larger UDP packets are used. Otherwise, the query is sent again using the Transmission Control Protocol (TCP). TCP is also used for tasks such as zone transfers. Some resolver implementations use TCP for all queries.

(지금보니 그 고수의 댓글ㄴ 위키피디아에서 그냥 퍼온 것...)