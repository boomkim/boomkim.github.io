---
layout: post
title:  "Amazon Elasticsearch Dedicated Master의 스토리지 비용은?"
date:   2019-06-07
categories: AWS Elasticsearch
---

# Amazon Elasticsearch Dedicated Master의 스토리지 비용은?

Amazon Elasticsearch 에는 Dedicated Master를 설정해줄 수 있습니다. 마스터노드의 기능만 하는 노드를 따로 빼서 클러스터의 안정성을 높이는 방안입니다. Elastic의 문서에도 클러스터의 사이즈가 큰 경우 마스터 노드를 분리하는 것을 권장하고 있네요. (참고: https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html)

오늘 내용은 Dedicated Master에 대한 디테일한 내용이 아니라 비용에 관한 간단한 얘기입니다. 

결론부터 말씀드리면 **Dedicated Master는 노드당 비용만 받고 스토리지 비용은 과금되지 않습니다.**

Amazon Elasticsearch를 Production 타입으로 만들면 아래 그림처럼 Data Instances, Dedicated master instances, Storage 등을 선택할 수 있습니다. 

![screenshot01](/images/elasticsearch-master-freestorage.png)

그런데 이 Storage 설정 부분은 Data node에만 적용이 됩니다. 여기에서 설정한 `Storage 양 X 데이터 노드의 수`가 클러스터의 총 용량이 되는거죠. 

그럼 Dedicated master instance 에는? Storage 관련해서 설정해주는 부분이 눈씻고 찾아봐도 없습니다. 
그래서 aws에 support case를 열어 물어보았더니 아래와 같은 답변이 왔습니다. 

> I would like to mention that Storage settings do not apply to any dedicated master nodes in the cluster.
> Hence, the pricing for dedicated master instances would be based on their instance type and not for the storage.

즉, Dedicated master instance는 그냥 instance 요금만 받는다는 겁니다. 하긴 Dedicated master instance 만해도 기본 인스턴스를 3개씩 써야되는데 스토리지 조금정도는 서비스해줄 수 있겠네요. 

별로 중요한 정보는 아니지만, aws 문서에 잘 나와있지 않아서 한번 포스팅해봅니다. 

