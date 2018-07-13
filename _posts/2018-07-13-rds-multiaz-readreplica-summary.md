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

## 읽기 복제본

![그림2](/images/rds_readrepl.png)

### 생성과정

1. 원본 스냅샷 생성
2. 스냅샷을 통해 복제본 인스턴스 생성 
3. 비동기식 복제 

### 기타 특징 

* 순환복제 지원 X 
* 체인식 복제 가능 (단 체인당 4대까지 )
* mysql - binlog mariadb - gtid
* read-only:0 으로 설정시 인덱스 설정등 쓰기 작업 가능 
* 복제본 자체를 multi-az 로 이중화 구성 가능
* 외부에서 복제/ 외부로 복제 가능 

## Multi-AZ vs 읽기 복제본

Multi-AZ | 읽기 복제본
-------- | --------- 
동기식 복제 | 비동기식 복제 
active-standby | 읽기 부하 분산
secondary(standby)노드에서 복제 가능 | 기본값으로 백업 설정 x 
최소 두개의 가용영역 | 단일리전/cross-AZ/cross-region 가능 
업데이트 시 primary 에 적용 | 독립적으로 적용 가능 
자동 장애 복구 | 장애 시 수동으로 승격 