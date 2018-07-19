---
layout: post
title:  "RDS(MySQL, MariaDB) - Multi-AZ / Read Replica (읽기전용 복제본) 관련 정리"
date:   2018-07-13
categories: AWS RDS
---

## Multi-AZ 

![그림1](/images/rds_ha1.jpg)

* Active - Standby 구조 
* 자동 장애 복구 
* 동기식 복제 (Synchronous replication)

위의 그림과 같이 원본 서버에 문제가 생기면 동기식으로 복제된 slave 인스턴스가 master로 자동으로 승격된다. 승격은 내부적으로 마스터를 바라보던 db instance의 도메인 엔드포인트가 slave를 가리킴으로써 구현된다. 

## 읽기 복제본

![그림2](/images/rds_readrepl.png)

### 생성과정

1. 원본 스냅샷 생성
2. 스냅샷을 통해 복제본 인스턴스 생성 
3. 변경사항 발생 시 비동기식 복제 

### 기타 특징 

*읽기복제본은 multi-az에 비해 자잘한 기능들이 많다.*

* 순환복제 지원 X 

a에서 복제본 b를 만들고 b에서 다시 a로 복제하는 것이 불가능하다. a에서 b라는 복제본을 만들면 b에서는 다시 새로운 복제본(b', b'' 등)을 만드는 것은 가능하다. 

* 체인식 복제 가능 (단 체인당 4대까지 )

위에서 말했듯이 a에서 복제본 a'를 생성하고 a'에서 a''를 생성하는 것이 가능하고 한 체인에 총 4대의 인스턴스까지 연결할 수 있다. 즉, a - a' - a'' - a''' 까지 가능하다는 말. 

* mysql - binlog mariadb - gtid

mysql은 복제시 원본에서 binarylog를 받아 복제본에다가 쓰는 작업을 통해 복제작업을 한다. 즉 복제본에서도 똑같이 쓰기 작업이 이루어져야 복제가 완료된다. mariadb는 gtid라는 방식을 사용한다고 한다. *gtid가 어떤 방식인지는 다음에 알아보는걸로...*

* read-only:0 으로 설정시 인덱스 설정등 쓰기 작업 가능 

읽기복제본이지만 쓰기가 아예 안되는 것은 아닌가 보다. 읽기 복제본의 읽기 성능 향상 등을 위해 read-only 기능을 끌 수 있게 되어있다. 이 기능을 껐을 때 쓰기가 어디까지 되는지는 테스트 해봐야 알것 같다. 

* 복제본 자체를 multi-az 로 이중화 구성 가능

말 그대로 인듯. 

* 외부에서 복제/ 외부로 복제 가능 

온프레미스에 있는 db의 복제본을 rds에 올려 사용할 수 있고 반대로 rds에 있는 db를 온프레미스에 복제해서 사용할 수도 있다. 이중화 구성, DR 구성을 위한 옵션인 것 같다. 물론 부하 분산도 되고. 

## Multi-AZ vs 읽기 복제본

AWS의 자료를 바탕으로 표로 정리해 보았다. 

Multi-AZ | 읽기 복제본
-------- | --------- 
동기식 복제 | 비동기식 복제 
active-standby | 읽기 부하 분산
secondary(standby)노드에서 복제 가능 | 기본값으로 백업 설정 x 
최소 두개의 가용영역 | 단일리전/cross-AZ/cross-region 가능 
업데이트 시 primary 에 적용 | 독립적으로 적용 가능 
자동 장애 복구 | 장애 시 수동으로 승격 

동기식 복제/ 비동기식 복제 차이점은 [여기](https://cloudbasic.net/white-papers/synchronous-vs-asynchronous-replication/)를 참고 

참고자료
* [AWS RDS Documentation - 읽기복제본 작업](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/UserGuide/USER_ReadRepl.html)
* [Deep Dive on Amazon RDS](https://www.slideshare.net/AmazonWebServices/dat302deep-dive-on-amazon-relational-database-service-rds?qid=5a05e0f6-fec2-4451-a0cd-e3769a0f0b7d&v=&b=&from_search=7)