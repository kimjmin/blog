---
title: Elastic의 새로운 고급 필드들
tags:
  - Elasticsearch
  - Elastic
  - fields
subtitle: Elastic 에 비교적 최근 추가된 고급 기능의 필드들을 살펴보고 어떻게 활용하는지에 대해 설명합니다.
header-img: newspaper-coffee.jpg
categories:
  - Engineering
date: 2021-04-14
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/9_MkRTXH0QU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Elastic은 버전업이 될 때 마다 나오는 새로운 기능들이 많습니다. 워낙 기본 기능도 다양하기 때문에 기본 기능 활용의 학습에만도 많은 시간을 들이게 되는데, 이번 포스트에서는 비교적 최근에 출시된 새로운 필드들에 대해 한번 살펴보고 활용 방법을 알아보도록 하겠습니다. 다음은 이 포스트에서 살펴볼 필드들입니다.
- [unsigned_long](#unsigned-long)
- [version](#version)
- [flattened](#flattened)
- [aggregate_metric_double](#aggregate-metric-double)
- [dense_vector](#dense-vector)

### [unsigned_long](https://www.elastic.co/guide/en/elasticsearch/reference/current/unsigned-long.html)
이름 그대로 양/음 부호를 없앤 long 정수 타입입니다. Java의 long 타입인 64비트로 표현 가능한 최대 정수값인 `0 ~ 18446744073709551615` 까지 숫자의 저장이 가능합니다. 1천8백경이 넘네요. 입력은 아래처럼 심플합니다.
```
PUT my_index
{
  "mappings": {
    "properties": {
      "my_counter": {
        "type": "unsigned_long"
      }
    }
  }
}
```

### [version](https://www.elastic.co/guide/en/elasticsearch/reference/current/version.html)
elasticsearch 에 시스템 메트릭 같은 정보들을 저장하게 되면 버전 정보값을 저장해야 할 일이 있습니다. 보통 버전 정보는 7.12.2 처럼 3군데 이상으로 구분되어있는 경우가 많아 소수점이 있는 숫자로 저장하기 힘들고 보통은 문자열로 저장을 많이 합니다. 그러다 보니 버전 간의 대/소를 비교하거나 정렬하는 것이 쉽지가 않았습니다. `version` 타입을 사용하면 이런 버전 형식의 값의 비교 또는 정렬이 가능합니다.
다음과 같이 version 타입의 필드인 my_version을 갖는 version-test 인덱스를 선언하고 값들을 넣습니다.
```
PUT version-test
{
  "mappings": {
    "properties": {
      "my_version": {
        "type": "version"
      }
    }
  }
}

POST version-test/_bulk
{ "index" : {}}
{"my_version": "1.5.0"}
{ "index" : {}}
{"my_version": "1.2.5"}
{ "index" : {}}
{"my_version": "2.1.7"}
{ "index" : {}}
{"my_version": "1.5.3-beta"}
{ "index" : {}}
{"my_version": "1.5.3-alpha2"}
```
이제 1.3.9 초과 1.5.3-beta 미만의 값으로 range 쿼리를 해 보면 
```
GET version-test/_search
{
  "query": {
    "range": {
      "my_version": {
        "gt": "1.3.9",
        "lt": "1.5.3-beta"
      }
    }
  }
}
```
다음과 같이 결과를 확인할 수 있습니다.
```
...
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "version-test",
        "_type" : "_doc",
        "_id" : "wlujzngBoohoz6ycrpvX",
        "_score" : 1.0,
        "_source" : {
          "my_version" : "1.5.0"
        }
      },
      {
        "_index" : "version-test",
        "_type" : "_doc",
        "_id" : "xlujzngBoohoz6ycrpvX",
        "_score" : 1.0,
        "_source" : {
          "my_version" : "1.5.3-alpha2"
        }
      }
    ]
  }
...
```
다음과 같이 my_version 필드의 오름차순으로 정렬을 해 보면
```
GET version-test/_search
{
  "sort": [
    {
      "my_version": {
        "order": "desc"
      }
    }
  ]
}
```
다음처럼 정렬이 됩니다.
```
...
{ "my_version" : "1.2.5" }
...
{ "my_version" : "1.5.0" }
...
{ "my_version" : "1.5.3-alpha2" }
...
{ "my_version" : "1.5.3-beta" }
...
{ "my_version" : "2.1.7" }
...
```
### [flattened](https://www.elastic.co/guide/en/elasticsearch/reference/current/flattened.html)
매핑에 하위 필드를 가지는 필드를 설계할 때는 보통 ojbect 또는 경우에 따라 nested 타입으로 설계를 합니다. 하지만 하위 필드로 어떤 타입의 필드가 오게 될지를 모르고, 데이터를 생성하는 개발자들이 자유롭게 형식을 만들 수 있도록 하고 싶은 경우 flattened 타입으로 선언이 가능합니다.
```
PUT flatten-label
{
  "mappings": {
    "properties": {
      "labels": {
        "type": "flattened"
      }
    }
  }
}
```
다음과 같이 labels 필드 안에 하위 필드로 다양한 값들을 넣어보겠습니다.
```
PUT flatten-label/_doc/1
{
  "labels": {
    "size": "XL",
    "color": "red",
    "price": 9000,
    "manufactured_date": "2020-10-21"
  }
}
```
입력한 값은 모두 정상적으로 들어가며, 다시 flatten-label 인덱스의 매핑을 확인 해 보아도 하위 필드는 자동으로 생성되지 않습니다.
```
### 요청 ###
GET flatten-label/_mapping

### 결과 ###
{
  "flatten-label" : {
    "mappings" : {
      "properties" : {
        "labels" : {
          "type" : "flattened"
        }
      }
    }
  }
}
```
입력된 하위 필드의 값은 모두 정상적으로 검색이 됩니다. 2개의 문서를 추가로 넣고 검색을 해 봅니다.
```
PUT flatten-label/_doc/2
{
  "labels": {
    "color": "green",
    "price": 11000
  }
}
PUT flatten-label/_doc/3
{
  "labels": {
    "color": "blue",
    "price": 13000
  }
}

### 쿼리 ###
GET flatten-label/_search
{
  "query": {
    "match": {
      "labels.color": "red"
    }
  }
}

### 결과 ###
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.9530773,
    "hits" : [
      {
        "_index" : "flatten-label",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.9530773,
        "_source" : {
          "labels" : {
            "size" : "XL",
            "color" : "red",
            "price" : 9000,
            "manufactured_date" : "2020-10-21"
          }
        }
      }
    ]
  }
```
한가지 주의 할 점은, 하위 필드는 입력 형태가 어떻든 모두 `keyword` 형태로 저장이 됩니다. 따라서 range 같은 쿼리는 할 수가 없습니다. price 필드를 내림차순으로 정렬 해 보면 다음처럼 9000 이 12000 보다 먼저 나오게 됩니다.
```
### 쿼리 ###
GET flatten-label/_search
{
  "sort": [
    {
      "labels.price": {
        "order": "desc"
      }
    }
  ]
}

### 결과 ###
    "hits" : [
      {
        "_index" : "flatten-label",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : null,
        "_source" : {
          "labels" : {
            "size" : "XL",
            "color" : "red",
            "price" : 9000,
            "manufactured_date" : "2020-10-21"
          }
        },
        "sort" : [
          "9000"
        ]
      },
      {
        "_index" : "flatten-label",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : null,
        "_source" : {
          "labels" : {
            "color" : "blue",
            "price" : 13000
          }
        },
        "sort" : [
          "13000"
        ]
      },
      {
        "_index" : "flatten-label",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : null,
        "_source" : {
          "labels" : {
            "color" : "green",
            "price" : 11000
          }
        },
        "sort" : [
          "11000"
        ]
      }
    ]
```

### [aggregate_metric_double](https://www.elastic.co/guide/en/elasticsearch/reference/current/aggregate-metric-double.html)
도큐먼트의 숫자 타입의 필드들은 aggregation을 통해 min, max, sum, avg 등의 값을 계산할 수 있습니다. 이 계산은 raw 도큐먼트들의 필드를 쿼리 때 마다 계산할 수도 있지만, aggs 를 위한 통계성 데이터만 저장 해 놓을 때에 매우 유용하게 사용될 수 있습니다. 이 때 사용하는 필드 타입이 `aggregate_metric_double` 입니다. aggregate_metric_double 에는 다음의 두 설정 값이 있습니다.
- metrics : 하위 필드로 저장할 메트릭을 지정합니다. [`min`, `max`, `sum`, `value_count`] 가 올 수 있습니다. `avg`는 별도로 저장하지 않는데, sum 과 value_count 를 이용해서 계산하게 됩니다.
- default_metric : 쿼리 시 사용할 디폴트 하위 필드를 지정합니다. 이 필드의 값을 `exist`, `range`, `term`, `terms` 쿼리로 검색할 수 있습니다.

다음과 같이 revenue-stats 인덱스에 revenue 필드를 aggregate_metric_double 타입으로 선언합니다.
```
PUT revenue-stats
{
  "mappings": {
    "properties": {
      "revenue": {
        "type": "aggregate_metric_double",
        "metrics": [ "min", "max", "sum", "value_count" ],
        "default_metric": "max"
      }
    }
  }
}
```
이 인덱스의 2020-01, 2020-02, 2020-03 도큐먼트에 revenue 필드의 하위 필드로 min, max, sum, value_count 값을 입력합니다.
```
PUT revenue-stats/_doc/2020-01
{
  "revenue": {
    "min": 25,
    "max": 470,
    "sum": 124400,
    "value_count": 374
  }
}
PUT revenue-stats/_doc/2020-02
{
  "revenue": {
    "min": 35,
    "max": 600,
    "sum": 187465,
    "value_count": 491
  }
}
PUT revenue-stats/_doc/2020-03
{
  "revenue": {
    "min": 20,
    "max": 580,
    "sum": 159650,
    "value_count": 420
  }
}
```
이제 `revenue` 필드를 각각 min, max, avg, sum, value_count 하는 aggs 들을 쿼리 해 봅니다.<font color=red>(revenue.min 같은 하위 필드가 아닌 revenue 필드인 것에 주목하세요)</font> 
```
### 쿼리 ###
GET revenue-stats/_search
{
  "aggs": {
    "min-revenue": { "min": { "field": "revenue" } },
    "max-revenue": { "max": { "field": "revenue" } },
    "sum-revenue": { "sum": { "field": "revenue" } },
    "avg-revenue": { "avg": { "field": "revenue" } },
    "total-count": { "value_count": { "field": "revenue" } }
  }, 
  "size": 0
}

### 결과 ###
  "aggregations" : {
    "max-revenue" : {
      "value" : 600.0
    },
    "total-count" : {
      "value" : 1285
    },
    "sum-revenue" : {
      "value" : 471515.0
    },
    "min-revenue" : {
      "value" : 20.0
    },
    "avg-revenue" : {
      "value" : 366.9377431906615
    }
  }
```
aggs 는 전부 revenue 필드를 대상으로 했지만 min, max, sum 등은 revenue 아래의 하위 필드인 min, max 필드들의 값을 가지고 집계가 된 것을 확인할 수 있습니다.
다음은 range 쿼리로 revenue 값이 500 이상 600 미만인 값을 가져오는 쿼리입니다.
```
### 쿼리 ###
GET revenue-stats/_search
{
  "query": {
    "range": {
      "revenue": {
        "gte": 500,
        "lt": 600
      }
    }
  }
}

### 결과 ###
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "revenue-stats",
        "_type" : "_doc",
        "_id" : "2020-03",
        "_score" : 1.0,
        "_source" : {
          "revenue" : {
            "min" : 20,
            "max" : 580,
            "sum" : 159650,
            "value_count" : 420
          }
        }
      }
    ]
  }
```
range 검색은 revenue 필드를 대상으로 했지만 매핑에서 default 로 지정한 revenue.max 값을 검색한 결과가 나타납니다. 만약에 range 검색을 revenue.max, revenue.min 같은 필드를 대상으로 검색하게 되면 검색 결과는 나타나지 않습니다.

### [dense_vector](https://www.elastic.co/guide/en/elasticsearch/reference/current/dense-vector.html)
dense_vector 를 이용하면 필드에 다차원의 벡터 값들을 입력해서 검색을 할 수 있습니다. `dims` 속성을 이용해서 차원 수를 입력합니다. 다음은 books 인덱스에 문자열 title 필드와 3차원의 벡터를 갖는 vector_recommend 필드에 값을 입력하는 예제입니다.
```
PUT books
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "vector_recommend": {
        "type": "dense_vector",
        "dims": 3
      }
    }
  }
}
```
이제 books 인덱스에 머신러닝 관련 서적들을 입력 해 보겠습니다. vector_recommend 필드에 순서대로
[ "페이지 수", "사용자 평점", "책 가격" ]
3개의 항목값들을 배열로 입력하도록 하겠습니다. 벡터 간격이 너무 벌어지지 않도록 모든 값을 0~10 사이로 맞추기 위해 페이지 수는 1/100, 책 가격은 1/5000 을 한 값을 입력했습니다.
```
# 1차원: 페이지 수/100
# 2차원: 사용자 평점 (최대 10.0)
# 3차원: 책 가격 / 5000

POST books/_bulk
{"index": {"_id": 1}}
{ "title": "혼자 공부하는 머신러닝+딥러닝", "vector_recommend": [5.8, 10.0, 5.2] }
{"index": {"_id": 2}}
{ "title": "핸즈온 머신러닝", "vector_recommend": [9.52, 9.1, 9.8] }
{"index": {"_id": 3}}
{ "title": "파이썬 머신러닝 완벽가이드", "vector_recommend": [6.48, 9.6, 7.6] }
{"index": {"_id": 4}}
{ "title": "밑바닥부터 시작하는 딥러닝", "vector_recommend": [3.12, 9.5, 4.8] }
{"index": {"_id": 5}}
{ "title": "빅데이터분석기사 필기", "vector_recommend": [3.68, 9.6, 5.0] }
{"index": {"_id": 6}}
{ "title": "선형대수와 통계학으로 배우는 머신러닝 with 파이썬", "vector_recommend": [6.24, 9.8, 7.5] }
{"index": {"_id": 7}}
{ "title": "파이썬 라이브러리를 활용한 머신러닝", "vector_recommend": [4.8, 9.2, 6.4] }
{"index": {"_id": 8}}
{ "title": "파이썬 머신러닝 판다스 데이터 분석", "vector_recommend": [3.92, 9.2, 5.0] }
{"index": {"_id": 9}}
{ "title": "텐서플로 2와 머신러닝으로 시작하는 자연어 처리", "vector_recommend": [5.6, 8.0, 7.0] }
{"index": {"_id": 10}}
{ "title": "R 데이터 분석 머신러닝", "vector_recommend": [3.08, 8.6, 4.0] }
```
이제 [script_score](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-script-score-query.html#vector-functions) 쿼리를 이용해서, books 인덱스의 도큐먼트들 중에
- 페이지 수가 적고 : `1`
- 평점은 높고 : `10`
- 가격은 저렴한 : `1`
추천 결과값을 가져오는 쿼리를 cosineSimilarity 를 이용해서 실행 해 보겠습니다.
```
GET books/_search
{
  "query": {
    "script_score": {
      "query":{
        "match_all": {}
      },
      "script": {
        "source": "cosineSimilarity(params.query_vector, 'vector_recommend') + 1.0", 
        "params": {
          "query_vector": [1, 10, 1]
        }
      }
    }
  }
}

### 결과 ###
"hits" : [
...
"_source" : {
  "title" : "R 데이터 분석 머신러닝",
  "vector_recommend" : [3.08, 8.6, 4.0]
...
"_source" : {
  "title" : "밑바닥부터 시작하는 딥러닝",
  "vector_recommend" : [3.12, 9.5, 4.8]
...
"_source" : {
  "title" : "빅데이터분석기사 필기",
  "vector_recommend" : [3.68, 9.6, 5.0]
...
```
가장 저렴하고 페이지수가 적은 책 부터 나타납니다. 이번에는 페이지수는 중간 `5`, 가격은 중간에서 약간 위 `6` 을 기준으로 놓고 그 조건에 맞는 책을 추천받는 쿼리를 실행 해 보겠습니다.
```
GET books/_search
{
  "query": {
    "script_score": {
      "query":{
        "match_all": {}
      },
      "script": {
        "source": "cosineSimilarity(params.query_vector, 'vector_recommend') + 1.0", 
        "params": {
          "query_vector": [5, 10, 6]
        }
      }
    }
  }
}

### 결과 ###
...
"_source" : {
  "title" : "파이썬 머신러닝 판다스 데이터 분석",
  "vector_recommend" : [3.92, 9.2, 5.0]
...
"_source" : {
  "title" : "파이썬 라이브러리를 활용한 머신러닝",
  "vector_recommend" : [4.8, 9.2, 6.4]
...
"_source" : {
  "title" : "혼자 공부하는 머신러닝+딥러닝",
  "vector_recommend" : [5.8, 10.0, 5.2]
...
```

elastcsearch 에는 이밖에도 다양한 고급 속성, 기능 들이 많이 있습니다. 잘 찾아보고 나에게 도움이 되는 기능들을 잘 활용 해 보시기 바랍니다.