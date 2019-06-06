---
layout: post
title:  "AWS Athena, CF Log 분석을 위한 쿼리 예시"
date:   2019-06-07
categories: AWS Athena Cloudfront
---

# AWS Athena, Cloudfront Log 분석을 위한 쿼리 예시

Cloudfront는 고맙게도 Edge단에서 발생하는 처리 로그들을 모아서 제공해 줍니다. CF Log는 기본적으로 S3에 gz 형태로 압축되엇 제공이 되고 
다음과 같은 DDL을 통해 테이블을 정의하면 Athena에서 조회할 수 있습니다. 

```SQL
CREATE EXTERNAL TABLE IF NOT EXISTS default.cloudfront_logs (
  `date` DATE,
  time STRING,
  location STRING,
  bytes BIGINT,
  request_ip STRING,
  method STRING,
  host STRING,
  uri STRING,
  status INT,
  referrer STRING,
  user_agent STRING,
  query_string STRING,
  cookie STRING,
  result_type STRING,
  request_id STRING,
  host_header STRING,
  request_protocol STRING,
  request_bytes BIGINT,
  time_taken FLOAT,
  xforwarded_for STRING,
  ssl_protocol STRING,
  ssl_cipher STRING,
  response_result_type STRING,
  http_version STRING,
  fle_status STRING,
  fle_encrypted_fields INT
)
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY '\t'
LOCATION 's3://CloudFront_bucket_name/AWSLogs/ACCOUNT_ID/'
TBLPROPERTIES ( 'skip.header.line.count'='2' )
``` 
출처: https://docs.aws.amazon.com/ko_kr/athena/latest/ug/cloudfront-logs.html

위의 쿼리문에서 다른것은 수정할 것 없이 `LOCATION` 뒤의 s3 버킷 주소만 자신의 버킷에 맞게 수정해주면 됩니다. 

이렇게 테이블 정의를 하고 나면 필요에 따라 쿼리를 날려 CF로그를 분석 할 수 있습니다. CF Cache Statistics 등에서도 통계를 제공해주기 때문에 통계에 관한 쿼리가 별로 필요하지 않을 수도 있지만, 따로 대시보드를 구성해 시각화 툴과 연계한다던가 할때는 필요할만한 것들을 모아봤습니다. Troubleshooting이 필요할 때는 그때그때 상황에 맞게 쿼리를 해야해서 예시를 들기가 어렵네요. 아래 예시 쿼리들은 참고만 하시고 필요에 맞게 변경해서 사용하시기 바랍니다. 

# 쿼리 예시 

## 일별 사용량 (GB)
```SQL
SELECT 
	date, sum(bytes)/1024/1024/1024 as GB  
FROM 
	"sample_db"."cloudfront_logs" 
GROUP BY
	date;
```

## CloudFront 응답 별 카운트 
``` SQL
SELECT 
	response_result_type, count(*) as cnt 
FROM 
	"sample_db"."cloudfront_logs" 
GROUP BY 
	response_result_type
ORDER BY 
	cnt DESC;
```

## 에러의 응답 코드 별 조회 
``` SQL
SELECT 
	status, count(*) as cnt
FROM 
	"sample_db"."cloudfront_logs"
WHERE 
	result_type = 'Error'
GROUP BY 
	status
ORDER BY 
	cnt DESC;
```

## 상위 20개 에러(4XX,5XX) uri 
``` SQL
SELECT 
	uri, count(*) as cnt 
FROM 
	"sample_db"."cloudfront_logs"
WHERE 
	result_type = 'Error'
GROUP BY 
	uri
ORDER BY 
	cnt DESC
limit 20;
```

## 상위 20개 요청ip count
``` SQL
SELECT 
	request_ip, COUNT(*) AS cnt 
FROM 
	cloudfront_logs
GROUP BY 
	request_ip
ORDER BY 
	cnt DESC;
```

## 상위 20 Referrer Count
``` SQL
SELECT 
	referrer, count(*) as cnt 
FROM 
	"sample_db"."cloudfront_logs"
GROUP BY 
	referrer
ORDER BY 
	cnt DESC 
LIMIT 20;
```


# 시계열 분석 예시

시계열 분석을 위해서는 기본 테이블에 저장된 date, time의 데이터를 concat(), from_iso8601_timestamp(), date_trunc() 등을 이용해 변환해주어야 합니다.  
Athena는 Select 절에서 Alias가 잘 만들어지지 않아서 cte를 쓰는 편이 편합니다. 


## CTE 예시 
```SQL
WITH cte AS (
	SELECT from_iso8601_timestamp(concat(concat(date_format(date, '%Y-%m-%d'), 'T'), time)) AS ts
    FROM "sample_db"."cloudfront_logs"
)
```

## 5월 29일(UTC 기준) 시간별 요청수, 오름차순 
``` SQL 
WITH cte AS (
	SELECT from_iso8601_timestamp(concat(concat(date_format(date, '%Y-%m-%d'), 'T'), time)) AS ts
    FROM "sample_db"."cloudfront_logs"
)SELECT 
	date_trunc('hour',ts) as TIME, count(*) as CNT
FROM cte
WHERE 
	ts >= from_iso8601_timestamp('2019-05-29T00:00:00') AND 
	ts < from_iso8601_timestamp('2019-05-30T00:00:00')
GROUP BY 
	date_trunc('hour',ts)
ORDER BY 
	date_trunc('hour',ts);
```

## 5월 29일(UTC 기준) 시간별 전송량(Gb)
``` SQL
WITH cte AS (
	SELECT from_iso8601_timestamp(concat(concat(date_format(date, '%Y-%m-%d'), 'T'), time)), bytes AS ts
    FROM "sample_db"."cloudfront_logs"
)SELECT 
	date_trunc('hour',ts) as TIME, sum(bytes)/1024/1024/1024 as Gb
FROM cte
WHERE 
	ts >= from_iso8601_timestamp('2019-05-29T00:00:00')AND 
	ts < from_iso8601_timestamp('2019-05-30T00:00:00')
GROUP BY 
	date_trunc('hour',ts)
ORDER BY 
	date_trunc('hour',ts);
``` 

## 5월 29일(UTC 기준) 시간별 에러수 
``` SQL
WITH cte AS (
	SELECT from_iso8601_timestamp(concat(concat(date_format(date, '%Y-%m-%d'), 'T'), time)) AS ts
    FROM "sample_db"."cloudfront_logs"
)SELECT 
	date_trunc('hour',ts) as TIME, count(*) as CNT
FROM 
	cte
WHERE 
	ts >= from_iso8601_timestamp('2019-05-29T00:00:00') AND 
	ts < from_iso8601_timestamp('2019-05-30T00:00:00') AND 
	result_type = 'Error'
GROUP BY 
	date_trunc('hour',ts)
ORDER BY 
	date_trunc('hour',ts);
``` 

## 시간별 평균 전송시간
``` SQL 
WITH cte AS (
	SELECT from_iso8601_timestamp(concat(concat(date_format(date, '%Y-%m-%d'), 'T'), time)) AS ts, time_taken
    FROM "sample_db"."cloudfront_logs"
)SELECT 
	date_trunc('hour',ts) as TIME, avg(time_taken) as avg_time
FROM cte
WHERE 
	ts >= from_iso8601_timestamp('2018-05-29T00:00:00') AND 
	ts < from_iso8601_timestamp('2019-05-30T00:00:00')
GROUP BY 
	date_trunc('hour',ts)
ORDER BY 
	date_trunc('hour',ts);
``` 

## 주의할 점 
Athena는 스캔한 데이터의 양에 비례해서 과금이 됩니다. 그런데 Cloudfront Log는 보통 양이 상당히 많은데도 데이터가 파티셔닝되지 않고 하나의 버킷에 통째로 저장이 되죠. <b>결국 Cloudfront의 Log 양이 상당해지면 Athena가 스캔하는 데이터의 양도 늘어나고 그러면 성능도 느려지겠죠.</b>

따라서 데이터의 양이 많아지는 경우, 파티셔닝을 고려해봐야 합니다. 여기 [AWS 블로그](https://aws.amazon.com/ko/blogs/big-data/analyze-your-amazon-cloudfront-access-logs-at-scale/)에서 Cloudfront로그를 파티셔닝하고, 파싱해서 더 많은 정보를 부여 (예를들어, bot인지 아닌지 여부를 판단해서 isBot이라는 column을 추가)하는 예제를 만들어 놓았네요. 따라해보진 않았지만 꽤 유용할것 같습니다. 글이 포스팅된지 꽤 된것 같은데 한국어로 번역된글도 없는걸 보니 한번 따라해보면서 번역글을 작성해도 괜찮겠네요. 