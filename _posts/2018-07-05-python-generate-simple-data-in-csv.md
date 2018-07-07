---
layout: post
title: "파이썬으로 간단한 테스트 데이터 생성하기 (csv)"
date: 2018-07-05
categories: python bigdata
---

테스트를 위한 데이터가 필요해서 데이터를 만들어야 할 때가 있다. 적은 양의 데이터는 수동으로 일일히 만들수도 있지만 데이터 양이 많아지면(1000건만 넘어가도...) 하나씩 만드는 건 너무 노가다.

그럴 때 데이터가 복잡하지 않고 단순한 구성일 경우에는 파이썬을 이용해 간단하게 생성할 수 있다. 

### random 모듈을 이용해 간단하게 짠 코드

```python
import csv
import random
import datetime
import timeit

start = timeit.default_timer()
file1 = open('mysql_d.csv','w', encoding='utf-8', newline='')
wr = csv.writer(file1)

extensions = ['hello','hi','bonjour','amazon','elastic','compute','gsneotek','everglow','line','kakaotalk','sql','test']
texts = ['Amazon Elastic Compute Cloud (Amazon EC2) provides scalable computing capacity in the Amazon Web Services (AWS) cloud. Using Amazon EC2 eliminates your need to invest in hardware up front so you can develop and deploy applications faster. You can use Amazon EC2 to launch as many or as few virtual servers as you need configure security and networking and manage storage. Amazon EC2 enables you to scale up or down to handle changes in requirements or spikes in popularity reducing your need to forecast traffic.','First you need to get set up to use Amazon EC2. After you are set up you are ready to complete the Getting Started tutorial for Amazon EC2. Whenever you need more information about an Amazon EC2 feature you can read the technical documentation.','You can provision Amazon EC2 resources such as instances and volumes directly using Amazon EC2. You can also provision Amazon EC2 resources using other services in AWS. For more information see the following documentation:','If you prefer to build applications using language-specific APIs instead of submitting a request over HTTP or HTTPS AWS provides libraries sample code tutorials and other resources for software developers. These libraries provide basic functions that automate tasks such as cryptographically signing your requestsetrying requests and handling error responses making it is easier for you to get started. For more information see AWS SDKs and Tools.']
words = texts[0].split()

for number in range(0,1000):
	val_ints = []
	val_varchars = []
	val_texts = []
	val_dates = []
	val_doubles = []

	for k in range(0,3):
		val_ints.append(random.randrange(0,10000))

	for k in range(0,13):
		val_varchars.append(random.choice(extensions))

	# for k in range(0,3):
	# 	val_texts.append(random.choice(texts))

	for k in range(0,4):
		val_dates.append(datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S'))

	# for k in range(0,2):
	# 	val_doubles.append(random.random())

	wr.writerow(val_ints + val_varchars + val_dates)

file1.close()
stop = timeit.default_timer()
print (stop - start)

```

### 간단하게 설명하자면, 

**각 데이터에 맞는 값을 random모듈을 통해 생성하고 csv형태에 맞게 출력한다.**

너무 쉬워서 설명이 저게 끝이다. 여기에 코드를 추가해서 데이터에 변화를 줄 수도 있다. random모듈에 내장되어 있는 정규분포함수를 이용해 정규분포에 따르는 데이터를 만들수도 있고 랜덤함수 여러개를 사용해서 값을 좀더 풍성하게 바꿀 수도 있다.

---

### 사이드 프로젝트...?

데이터 생성기를 만드는 건 간단하지만 활용도는 꽤 높은 것 같다. 위 코드는 그냥 하드 코딩이어서 재사용하려면 일일이 수정해 줘야 한다. 그런데 앞으로 쓸 일이 많아서 재사용할수 있는 형태로 만들어 보려고 한다. 

1. 커맨드라인으로 커스터마이징이 가능한 파이썬 프로그램 
2. 웹을 통해 만들고자 하는 데이터의 정보/포맷을 입력하면 csv파일을 다운로드를 받을 수 있는 서비스 

2번의 경우 고민할 것이 많아지겠지만(연산을 어디에서 할 것인지, 서버를 둘 것인지, 어떤 언어를 사용할 것인지 등등...) 사이드 프로젝트로 적당한 난이도와 규모가 아닐까 싶다. 필요한 사람도 꽤 있을 것 같고...

---

#### 추가사항 

데이터를 만들다보니 많은 데이터를 만들경우 위 코드를 사용하면 시간이 너무 오래 걸린다(20개 컬럼 1억 5천만 줄 생성시 1시간 30분 이상 소요). cpu사양도 높으면 좋겠지만 파이썬은 기본적으로 한개의 쓰레드에서만 동작하니 아무리 cpu의 코어 수, 쓰레드 수가 많아도 쓰질 않으니 소용이 없다. 그래서 쓰레딩을 도입해보려고 한다. 다음 포스팅에서는 간단하게 쓰레딩을 도입해서 쓰레딩 도입 한 것과 안 한 것을 비교해 봐야겠다.

 *+ 만약에 이 코드를 c나 java, go등의 언어로 짜면 성능이 얼마나 개선될까?*
