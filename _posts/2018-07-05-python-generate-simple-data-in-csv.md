---
layout: post
title: "파이썬으로 간단한 테스트 데이터(csv)파일 생성하기"
date: 2018-07-05
---

테스트를 위한 데이터가 필요해서 데이터를 만들어야 할 때가 있다. 적은 양의 데이터는 수동으로 일일히 만들수도 있지만 데이터 양이 많아지면(1000건만 넘어가도...) 하나씩 만드는 건 너무 노가다.

그럴 때 데이터가 복잡하지 않고 단순한 구성일 경우에는 파이썬을 이용해 간단하게 생성할 수 있다. 

```python
import csv
import random
import datetime

start = timeit.default_timer()
file1 = open('testdata.csv','w', encoding='utf-8', newline='')
wr = csv.writer(file1)

texts = ['Amazon Elastic Compute Cloud (Amazon EC2) provides scalable computing capacity in the Amazon Web Services (AWS) cloud. Using Amazon EC2 eliminates your need to invest in hardware up front so you can develop and deploy applications faster. You can use Amazon EC2 to launch as many or as few virtual servers as you need configure security and networking and manage storage. Amazon EC2 enables you to scale up or down to handle changes in requirements or spikes in popularity reducing your need to forecast traffic.','First you need to get set up to use Amazon EC2. After you are set up you are ready to complete the Getting Started tutorial for Amazon EC2. Whenever you need more information about an Amazon EC2 feature you can read the technical documentation.','You can provision Amazon EC2 resources such as instances and volumes directly using Amazon EC2. You can also provision Amazon EC2 resources using other services in AWS. For more information see the following documentation:','If you prefer to build applications using language-specific APIs instead of submitting a request over HTTP or HTTPS AWS provides libraries sample code tutorials and other resources for software developers. These libraries provide basic functions that automate tasks such as cryptographically signing your requestsetrying requests and handling error responses making it is easier for you to get started. For more information see AWS SDKs and Tools.']
words = texts[0].split()

for number in range(0,1000):
	a = random.randrange(1,10001)
	b = random.randrange(1,10001)
	c = random.choice(words)
	d = random.choice(words)
	e = random.choice(words)
	f = random.choice(words)
	g = random.choice(words)
	h = random.choice(words)
	i = random.choice(words)
	j = random.choice(words)
	k = random.choice(words)
	l = random.choice(words)
	m = random.choice(words)
	n = random.choice(words)
	o = random.choice(words)
	p = random.choice(words)
	q = random.choice(words)
	r = random.choice(words)
	s = random.choice(words)
	t = random.choice(words)
	u = random.choice(words)
	v = random.choice(words)
	w = random.choice(words)
	x = random.choice(words)
	y = random.choice(texts)
	z = random.choice(texts)
	b1 = random.choice(texts)
	b2 = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
	b3 = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
	b4 = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
	wr.writerow([a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p, q, r, s, t, u ,v, w, x, y, z, b1, b2, b3, b4])

file1.close()
```