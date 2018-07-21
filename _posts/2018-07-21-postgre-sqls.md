---
layout: post
title:  "자주쓰는 postgresql 쿼리"
date:   2018-07-21
categories: postgresql
---

`set search_path to abc_schema`

: 스키마 설정. <br>

`describe table_a;`

: describe 하면 해당 테이블에 관련된 인덱스 까지 다 나와서 좋다. <br>

`drop index indexname;`

: 인덱스를 날려버린다. <br>

`alter table postgre_c drop constraint postgre_c_pkey;`

: Primary key는 drop index 명령어로 안날아가서 이렇게 날려야 한다. <br>

`ALTER TABLE distributors ADD PRIMARY KEY (dist_id);`

: primary key 추가할땐 이렇게. <br>

`create index idx1 on table1 (column1)`

: 인덱스 추가는 이렇게 <br>

`truncate table_a;`

: 테이블을 날려버릴땐 이렇게. 자동으로 vacuum까지 된다. <br>