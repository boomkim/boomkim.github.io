---
layout: post
title:  "EMR 에 올라간 Zeppelin 활용하여 Athena 이용하기. "
date:   2019-06-29
categories: Bigdata AWS EMR
---

# EMR의 Zeppelin을 통해 Athena 활용하기. 

## 들어가기 전에... 

최근에 AWS Datascience 소그룹 모임의 자료를 쭉 보던 중 아주 정리가 잘 되어있는 자료를 보았습니다. [AWS 기반 지속 가능한 데이터 분석하기
](https://github.com/awskrug/datascience-group/tree/master/workshop-sustainable_data_analysis) 라는 자료인데 [SK 빅데이터 허브](https://www.bigdatahub.co.kr/index.do)에서 제공하는 배달 업종 통화 기록을 베이스로 spark로 데이터를 변환하고 athena, Presto, tableau 를 이용해서 시각화 하는 내용을 담고 있습니다. ETL 부터 시각화까지의 내용을 아주 재밌는 데이터와 함께 잘 정리한 글입니다. (AWS 데이터사이언스 모임은 가보지 않았지만 언젠간 한번 꼭 참석하리라 마음을 먹은지 벌써 몇달째... )

한편, 위의 글에서는 Zeppelin을 Spark의 노트북으로써만 한정해서 사용하는게 조금 아쉬웠달까요. 물론 Tableau라는 훌륭한 시각화 툴을 언급했으므로 굳이 Zeppelin과 Athena를 엮을 필요는 없지만, [전에 글](https://boomkim.github.io/aws/athena/zeppelin/2019/05/24/athena-with-zeppelin.html)에서도 말했지만 s3+athena의 활용성을 높이기에 Zeppelin 만한 것도 없는것 같습니다. 

특히, 위의 글에서 처럼 Spark을 EMR로 관리하고 그 노트북으로써 Zeppelin을 활용하는 환경이라면, Athena에도  접근이 가능하게 하는 것도 활용성을 높여줄 것 같습니다. **EMR이 제공하는 Zeppelin 만으로 ETL, 분석을 쉽게할 수 있도록 노트북 환경을 만들고 여차하면 시각화까지 해결하는거죠.** (물론 Zeppelin의 시각화에는 한계가 있습니다. Zeppelin은 노트북일 뿐 BI툴이 아니니까요) 

(이 문단은 안읽으셔도 됩니다. 괜한 딴지니까요.)
사실, Zeppelin을 이런 식으로 꽤나 중요하게 사용하고 싶다면 굳이 EMR에서 제공하는 Zeppelin을 이용하지 않아도 됩니다. EMR은 비싸니까요. EC2위에 직접 Zeppelin을 설치하고 따로 관리를 하는 편이 나을수도 있습니다. EMR은 사실 항시 유지하는 클러스터를 위한 솔루션이 아니니까요. 24X7 유지하는 클러스터를 위해서 EMR을 사용한다면 Multi-Master 기능을 고려할텐데 심지어 Multi-Master모드로 EMR을 올린다면 Zeppelin을 같이 사용할 수 없습니다. 이것저것 고려하니 복잡해졌네요. 그냥 AWS에서 EMR 프로비저닝 할때 Athena와의 연계를 미리 고려해서 Zeppelin에 Interpreter 하나만 추가해놔주면 고민이 쉽게 해결될것 같네요. 

이유야 어떻든 간에, 이 글의 목표는 **EMR에 올려진 Zeppelin에 Athena를 연결**시키는 겁니다. EC2에 Zeppelin이 올라가 있다고 해도 크게 달라지진 않을 것 같습니다. **Role을 이용한 권한 연결**이 핵심이니까요. 서론이 길었습니다. 얼른 본론으로 갑시다. 

*** 

## EMR의 Zeppelin에 Athena Interpreter 추가하기

*이번 글에서는 EMR에 Spark와 Zeppelin이 올라가 있다고 가정합니다.* 

EMR에서 제공하는 Zeppelin에 Athena를 연결하려면 크게는 다음과 같은 작업을 해야합니다. 

* IAM Role 수정 
* Zeppelin에 JDBC Interpreter 설치 
* Athena 용 JDBC 다운로드 
* Athena에 맞게 JDBC Interpreter 추가

그럼 하나씩 살펴보겠습니다. 

### 1. IAM Role 수정

EMR의 Master에는 기본적으로 DynamoDB, Glue, Kinesis, RDS, S3, SNS, SQS 등에 관한 권한이 설정되어 있습니다. (아래 그림 참고)

![그림1](/images/EMR-EC2-Role.png)

그런데 이번에는 Athena와 연동을 시켜줄 것이기 때문에, Athena를 컨트롤할 수 있는 권한을 추가해줘야합니다. 
뭔가 Default는 건들면 안될 것 같으니 다음과 같이  새로 IAM Role을 추가합니다. 

* Role -> Create Role 클릭
* 이 역할을 사용할 서비스에서 EC2 선택후 [Next: Permissions] 클릭 
* 'AmazonElasticMapReduceforEC2Role' 와 'AmazonAthenaFullAccess' 를 찾아 선택
* 태그는 무시 (원한다면 당연히 입력하셔도 됩니다.)
* Role 이름에 'EMR_EC2_AthenaRole' 입력 후 [Create Role] 클릭 

이제 Role이 완성되었으니 이를 원래 있던 Role대신 넣어줘야 합니다. Zeppelin은 EMR의 마스터 노드에 있으니 Master노드를 찾아 변경해 Role을 변경해줍니다. 

* EMR -> 해당 Cluster 클릭 -> Hardware -> Master 의 ID 클릭 -> EC2 Instance ID 클릭 (EC2 콘솔로 이동하게 됩니다)
* EC2 콘솔에서 Actions -> Instance Settings -> Attach/Replace IAM Role 클릭 
* IAM role 에서 'EMR_EC2_AthenaRole' 선택 후 [Apply] 클릭 

자, 이제 EMR의 마스터노드에서는 Athena를 마음껏 컨트롤할 수 있게 되었습니다. 

그럼 이제 Zeppelin을 세팅해 봅시다.

### 2. Zeppelin에 JDBC Interpreter 설치 

이상하게 EMR의 Zeppelin에는 JDBC Interpreter가 설치되어 있지 않습니다. 일단 Spark 만 컨트롤할 수 있게 가볍게 세팅이 되어 있는것 같네요. 그러니 우선 JDBC Interpreter를 설치해 줍시다. 

일단, 마스터 노드에 ssh 로 접근해줍니다. (Windows는 putty 등 사용)

 ```bash
 ssh -i {나의pemkey.pem} hadoop@{Masternode EC2  Public DNS}
 ````

그 다음엔 일단 Zeppelin을 멈추고, 

```bash
sudo stop zeppelin
``` 

jdbc interpreter를 설치하고,

```bash
sudo /usr/lib/zeppelin/bin/install-interpreter.sh -n jdbc
```

Zeppelin을 다시 시작해줍니다. 

```bash
sudo start zeppelin
```

~~*참 쉽죠?*~~


### 3. Athena 용 JDBC 다운로드 

그럼 이제 Zeppelin이 사용할 Athena용 JDBC 드라이버를 받아봅시다. 

우선 jar 파일이 들어갈 경로를 정해봅시다. 저는 `/usr/local/jar` 로 정했습니다. 

```bash
sudo mkdir /usr/local/jar
sudo cd /usr/local/jar
```

wget을 이용해서 jdbc 파일을 받고, 

```bash
sudo wget https://s3.amazonaws.com/athena-downloads/drivers/JDBC/SimbaAthenaJDBC_2.0.7/AthenaJDBC42_2.0.7.jar
```
마지막으로, zeppelin이 실행시킬 수 있게 권한을 바꿔줍니다.

```bash
sudo chmod 755 AthenaJDBC42_2.0.7.jar
```

### 4. Athena에 맞게 JDBC Interpreter 추가

자, 이제 마지막입니다. JDBC Interpreter만 Athena에 맞게 세팅해서 구성해주면 끝납니다. 

우측상단에 있는 Interpreter를 클릭후 Interpreter 창에서 Create를 누릅니다. 

![그림2](/images/zeppelin-interpreter.png)

그 후 다음과 같이 세팅합니다.
* `Interpreter Name` 은 적당히 'athena' 로 정해주고 
* `Interpreter Group` 은 JDBC 를 선택합니다. 

Properties 에서 
* `default.password`, `default.user`는 빈칸으로
* `default.driver`는 `com.simba.athena.jdbc.Driver`
* `default.url`은 다음과 같이 세팅합니다. 
  * `jdbc:awsathena://athena.ap-northeast-2.amazonaws.com:443;S3OutputLocation=s3://aws-athena-query-results-{계정번호12자리}-ap-northeast-2;Schema=default;AwsCredentialsProviderClass=com.simba.athena.amazonaws.auth.InstanceProfileCredentialsProvider; ` (서울리전 기준, 나머지 리전의 경우 리전에 맞춰서 세팅해주면 됩니다.)

Artifact는 jar파일을 받은 경로를 입력하면 됩니다. 

`/usr/local/jar/AthenaJDBC42_2.0.7.jar`

*Save를 누르면 드디어! Athena를 사용할 Interpreter가 생성이 됩니다.*

***

## 테스트 

그럼 Athena랑 연결이 잘 되었는지 테스트해 봅시다. 

Notebook-> Create New Note 에서 Default Interpreter를 Athena 로 선택 후 생성합니다. 

![그림3](/images/zeppelin-create-notebook.png)

athena에 등록된 테이블에 쿼리를 날려봤습니다. 

![그림4](/images/emr-zeppelin-athena-final-example.png)

잘 되네요. ~~(꼭 얄밉게 잘된것만 보여주는 나쁜 사람들이 있습니다.)~~

*** 

## 정리 

지금까지 EMR에서 설치해주는 Zeppelin에 Athena를 얹어봤습니다. 이를 통해서 

* Spark로 ETL해서 S3로 넣고 
* Athena로 분석
* 부족하지만 시각화 

까지 Zeppelin이라는 하나의 노트북을 환경 통해 좀 편하게 할 수 있지 않을까 해서 포스팅을 써봤습니다. 

BI를 고려한다면 Zeppelin 안써도 됩니다. BI툴 쓰세요 ㅎㅎㅎ. 그리고 Athena 안쓰고 Spark 내에서 다 ETL, 분석 까지 가능하다면 굳이 Athena와 연결하는데 에너지 쓸 이유는 없습니다. 

다만, BigQuery를 필두로 한 SQL을 통한 분석이 요새 대세인것 같기도 하고 실제로 데이터분석가들이 SQL을 많이 사용하는것 같기도 해서 Athena의 사용성이 좋아지길 기대하고 있습니다. 또 Athena와의 연동이 보시다시피 그렇게 어렵지는 않잖아요? 

한편으로는 오픈소스 Zeppelin이 더 잘되길 바라는 마음도 있습니다. Zeppelin 화이팅! 







