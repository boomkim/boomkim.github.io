---
layout: post
title: "[Python] 파이썬으로 간단한 테스트 데이터 생성하기 (csv)"
date: 2018-07-05
categories: python bigdata
comments: true
---

테스트를 위한 데이터가 필요해서 데이터를 만들어야 할 때가 있다. 적은 양의 데이터는 수동으로 일일히 만들수도 있지만 데이터 양이 많아지면(1000건만 넘어가도...) 하나씩 만드는 건 너무 노가다.

그럴 때 데이터가 복잡하지 않고 단순한 구성일 경우에는 파이썬을 이용해 간단하게 생성할 수 있다. 

### *random 모듈*을 이용해 간단하게 짠 코드

아래의 코드는 *int형 데이터 3개, varchar 형 13개, datetime형 4개로 이루어진 레코드 1000개* 를 만들어서 csv 파일로 저장한다. 

```python
import csv
import random
import datetime
import timeit
import argparse


class Argparser:
    def __init__(self):
        self.parser = argparse.ArgumentParser()
        self.init_parser()

    def init_parser(self):
        parser = self.parser
        parser.add_argument("-i", "--integers", help="how many integer value do you want to generate")
        parser.add_argument("-ri", "--range-integer", help="describe range of integer")
        parser.add_argument("-vc", "--varchars", help="how many varchar value do you want to generate")
        parser.add_argument("-t", "--texts", help="how many text value do you want to generate")
        parser.add_argument("-dt", "--dates", help="how many datetime value do you want to generate")
        parser.add_argument("-d", "--doubles", help="how many double value do you want to generate")
        parser.add_argument("-r", "--rows", help="describe number of rows")
        parser.add_argument("-ta", "--textarrays", help="how many text arrays...")
        parser.add_argument("-o", "--output", help="describe file name")
        self.parser = parser


class Generator:
    def __init__(self, args):
        self.rows = int(args.rows) if args.rows else 1000
        self.filename = args.output if args.output else "out.csv"
        self.integers = int(args.integers) if args.integers else 5
        self.varchars = int(args.varchars) if args.varchars else 5
        self.texts = int(args.texts) if args.texts else 5
        self.doubles = int(args.doubles) if args.doubles else 5
        self.dates = int(args.dates) if args.dates else 1
        self.textarrays = int(args.textarrays) if args.textarrays else 0
        self.text_pool = ['Amazon Elastic Compute Cloud (Amazon EC2)',
                          'First you need to get set up to use Amazon EC2.', 
                          'You can provision Amazon EC2 resources such as instances and volumes.',
                          'If you prefer to build applications using language-sp.']
        self.word_pool = self.text_pool[0].split()

    def generate(self):
        f = open(self.filename, 'w', encoding='utf-8', newline='')
        wr = csv.writer(f)
        for i in range(0, self.rows):
            idx = [i + 1]
            val_ints = []
            val_varchars = []
            val_texts = []
            val_doubles = []
            val_dates = []
            val_textarrays = []

            for j in range(0, self.integers):
                val_ints.append(random.randrange(0, 10000))
            for j in range(0, self.varchars):
                val_varchars.append(random.choice(self.word_pool))
            for j in range(0, self.texts):
                val_texts.append(random.choice(self.text_pool))
            for j in range(0, self.textarrays):
                val_textarrays.append('{\"abc\",\"def\",\"ghi\"}')
            for j in range(0, self.dates):
                val_dates.append(datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S'))
            for j in range(0, self.doubles):
                val_doubles.append(random.random())
            wr.writerow(idx + val_ints + val_varchars + val_texts + val_textarrays + val_dates + val_doubles)
        f.close()


if __name__ == "__main__":
    parser = Argparser()
    args = parser.parser.parse_args()
    print("Generating file...")
    start = timeit.default_timer()
    generator = Generator(args)
    generator.generate()
    stop = timeit.default_timer()
    print(generator.filename + " is created. ")
    print("It took " + str(stop - start) + " seconds to generate data.")
```

### 간단하게 설명하자면... 

**각 데이터에 맞는 값을 random모듈을 통해 생성하고 csv형태에 맞게 출력한다.**

* `random.randrange()`를 이용해 int를 만들고
* `random.choice()`를 통해 문자열을 만들고 
* `datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')` 을 이용해 현재 시간을 스트링으로 만든다

너무 쉬워서 설명이 이게 끝이다. 여기에 코드를 추가해서 데이터에 변화를 줄 수도 있다. random모듈에 내장되어 있는 정규분포함수를 이용해 정규분포에 따르는 데이터를 만들수도 있고 랜덤함수 여러개를 사용해서 값을 좀더 풍성하게 바꿀 수도 있다.
<br><br>

---

<br>
### 사이드 프로젝트...?

데이터 생성기를 만드는 건 간단하지만 활용도는 꽤 높은 것 같다. 위 코드는 그냥 하드 코딩이어서 재사용하려면 일일이 수정해 줘야 한다. 그런데 앞으로 쓸 일이 많아서 재사용할수 있는 형태로 만들어 보려고 한다. 

1. 커맨드라인으로 커스터마이징이 가능한 파이썬 프로그램 
2. 웹을 통해 만들고자 하는 데이터의 정보/포맷을 입력하면 csv파일을 다운로드를 받을 수 있는 서비스 

2번의 경우 고민할 것이 많아지겠지만(연산을 어디에서 할 것인지, 서버를 둘 것인지, 어떤 언어를 사용할 것인지 등등...) 사이드 프로젝트로 적당한 난이도와 규모가 아닐까 싶다. 필요한 사람도 꽤 있을 것 같고...
<br><br>

---

<br>
#### 추가사항...

데이터를 만들다보니 많은 데이터를 만들경우 위 코드를 사용하면 시간이 너무 오래 걸린다(20개 컬럼 1억 5천만 줄 생성시 1시간 30분 이상 소요). cpu사양도 높으면 좋겠지만 파이썬은 기본적으로 한개의 쓰레드에서만 동작하니 아무리 cpu의 코어 수, 쓰레드 수가 많아도 쓰질 않으니 소용이 없다. 그래서 **쓰레딩을 도입**해보려고 한다. 다음 포스팅에서는 간단하게 쓰레딩을 도입해서 쓰레딩 도입 한 것과 안 한 것을 비교해 봐야겠다.

 *+ 만약에 이 코드를 c나 Java, go등의 언어로 짜면 성능이 얼마나 개선될까?*
