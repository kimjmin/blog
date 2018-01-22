---
title: Elasticsearch 6.x 에서의 join 사용
tags:
  - Elasticsearch
categories:
  - Elasticsearch
subtitle: Elasticsearch 6.x 에서 join 관계 설정 및 Logstash 색인 설정.
date: 2018-01-19
header-img: "join-data.png"
---

## 5.x 이전의 도큐먼트 간 parent - child 구조

Elasticsearch 에서는 도큐먼트들 간에 연결을 맺을 수 있는 몇가지 기능들을 제공하고 있습니다. 대표적으로는 nested type 이 있으며, 5.x 이전 버전에서는 parent-child 구조의 정의를 할 수 있었습니다.

5.x 이전의 parent - child 구조는 인덱스 내부의 타입을 parent, 그리고 child 타입으로 나눠서 생성하고 child 에 속한 도큐먼트들이 색인될 때 해당 도큐먼트의 parent 를 명시 해서 저장하는 방식으로 사용했습니다. 자세한 내용은 아래를 문서를 참고하세요.
https://www.elastic.co/guide/en/elasticsearch/reference/5.6/mapping-parent-field.html#_parent_child_restrictions

다음은 `stackoverflow` 라는 인덱스에 `question` 타입과 `answer` 타입을 각각 parent - child 구조로 저장 한 예제 입니다.
```
{
  "stackoverflow": {
    "mappings": {
      "question": {
        "properties": {
          "accepted_answer_id": {
            "type": "long"
          }
          ... 중략 ...
        }
      },
      "answer": {
        "_parent": {
          "type": "question"
        },
        "_routing": {
          "required": true
        },
        "properties": {
          "id": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

## 6.x 에서의 join 데이터 타입 설정

6.0 부터는 한 인덱스에 하나의 타입만 생성할 수 있도록 강제됩니다. 그래서 parent - child 구조는 더 이상 사용이 불가능하고, 대신 join 이라는 데이터 타입을 이용해서 도큐먼트들 간의 관계를 정의하게 됩니다. join 데이터 타입에 대해서는 아래 문서를 참고합니다.
https://www.elastic.co/guide/en/elasticsearch/reference/6.1/parent-join.html

먼저 관계를 설정하기 위한 join 필드를 새로 추가합니다. 저는 `qna_join` 이라는 필드를 join 필드로 설정했습니다.
```
PUT stackoverflow
{
  "mappings": {
    "doc": {
      "properties": {
        "qna_join": {
          "type": "join",
          "relations": {
            "question": "answer" 
          }
        }
      }
    }
  }
}
```

그리고 question 도큐먼트에는 `qna_join` 필드 안의 `name` 값을 `question` 으로, answer 도큐먼트에는 `answer` 와 parent에 해당하는 question 도큐먼트의 id 값을 넣어줍니다. 
```
POST stackoverflow/doc/25691276?routing=25691276
{
  "title": "Import CSV into MySQL - Offset by 1 Column",
  "accepted_answer_id": 25691509,
  ... 중략 ...
  "id": 25691276,
  "view_count": 15,
  "qna_join": {
    "name": "question"
  }
}

POST stackoverflow/doc/25691509?routing=25691276
{
  "comment_count": 0,
  "owner": {
    "location": "Sao Paulo, Brazil",
    "id": 3337405,
    "display_name": "vinibarr"
  },
  "comments": [],
  "creation_date": "2014-09-05T18:01:23.033",
  "id": 25691509,
  "body": """
<p>You can load your data specifing the order columns that you're going to use into your table:
... 중략 ...
""",
  "qna_join": {
    "name": "answer",
    "parent":"25691276"
  }
}
```

중요한 것은 parent, child 두개 도큐먼트는 항상 동일한 routing 값을 넣어줘야 합니다. 같은 routing 값을 가진 도큐먼트는 같은 샤드에 저장이 됩니다.
이렇게 저장한 후 `has_parent` 쿼리를 이용해서 `question` 도큐먼트의 id 필드 값을 이용해서 그 도큐먼트와 연결된 `answer` 도큐먼트를 가져오는 쿼리를 실행 해 봅니다.
```
GET stackoverflow/_search
{
  "query": {
    "has_parent": {
      "parent_type": "question",
      "query": {
        "term": {
          "id": {
            "value": "25691276"
          }
        }
      }
    }
  }
}
```
위 쿼리를 실행하면 아래와 같이 `answer` 도큐먼트를 결과로 가져옵니다.
```
{
  "took": 12,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1,
    "hits": [
      {
        "_index": "stackoverflow",
        "_type": "doc",
        "_id": "25691509",
        "_score": 1,
        "_routing": "1",
        "_source": {
          "comment_count": 0,
          "owner": {
            "location": "Sao Paulo, Brazil",
            "id": 3337405,
            "display_name": "vinibarr"
          },
          "comments": [],
          "creation_date": "2014-09-05T18:01:23.033",
          "id": 25691509,
          "body": """
<p>You can load your data specifing the order columns that you're going to use into your table:
    ... 중략 ...
""",
          "qna_join": {
            "name": "answer",
            "parent": "25691276"
          }
        }
      }
    ]
  }
}
```

## Logstash 색인 설정

이제 5.x 에서 parent - child 타입으로 나뉘어 있던 데이터를 6.x 로 재 색인을 해야 합니다. 먼저 데이터를 타입별로 구분해야 하므로 저는 question 타입과 answer 타입의 데이터들을 각각 `/Users/kimjmin/elastic/source/stackoverflow/` 경로 아래에 `question.json`, `answer.json` 이라는 파일들로 저장 했습니다.
이제 logstash 설정 파일을 작성하겠습니다. path 로 부터 파일 이름에 있는 `question` 그리고 `answer` 를 추출하여 `qna_join.name` 에 해당하는 값을 넣어주도록 했습니다.

 우선 파일 이름을 기준으로 `question`, `answer` 태그를 만들도록 합니다.

```
input {
  file {
    path => "/Users/kimjmin/elastic/source/stackoverflow/*.json"
    sincedb_path => "/dev/null"
    start_position => "beginning"
    codec => "json"
  }
}

filter {
  # "/" 기준으로 path를 배열로 분리하여 [6]번째 값인 "question.json" 또는 "answer.json"을 path_array 에 저장.
  mutate {
    split => { "path" => "/" }
    add_field => { "path_array" => "%{path[6]}" }
  }

  mutate {
    # "." 기준으로 path_array를 배열로 분리하여 [0]번째 값인 "question" 또는 "answer"을 qna_join.name, doc_type 에 저장.
    split => { "path_array" => "." }
    add_field => { "[qna_join][name]" => "%{path_array[0]}" }
    add_field => { "doc_type" => "%{path_array[0]}" }
  }

  mutate {
    remove_field => ["host","@version","path","path_array","@timestamp"]
  }

  ... 중략 ...

}
```

현재 구성중인 stackoverflow 는 parent가 question, child 가 answer 구조로 되어 있습니다. 하지만 도큐먼트 내용을 보면 question 도큐먼트에는 answer 도큐먼트와 연결되는 `accepted_answer_id` 필드가 있지만 answer 에는 question 도큐먼트를 확인하는 필드가 없습니다. answer 도큐먼트가 색인 될 때 연결되는 question 도큐먼트의 id 값을 가져오기 위해 Logstash 의 filter 내부에 `elasticsearch` 필터를 추가합니다.
https://www.elastic.co/guide/en/logstash/6.1/plugins-filters-elasticsearch.html

```
filter {

  ... 중략 ...

  if [doc_type] == "question" {
    # routing 을 위해 question 도큐먼트의 id를 question_id 필드에 저장
    mutate{
      add_field => { "question_id" => "%{id}" }
    }
  } else if [doc_type] == "answer" {
    # routing 을 위해 answer에 해당하는 question 도큐먼트의 id 를 가져와서 question_id 필드에 저장
    elasticsearch {
      hosts => ["localhost:9200"]
      index => "stackoverflow"
      query => "doc_type:question AND accepted_answer_id:%{id}"
      fields => { "id" => "[qna_join][parent]" }
    }
    
    mutate{
      add_field => { "question_id" => "%{[qna_join][parent]}" }
    }
  }

}
```

이제 데이터를 elasticsearch로 저장하도록 output 을 입력합니다. parent - child 구조의 도큐먼트는 같은 샤드에 저장하기 위해 항상 같은 rounting 값을 적어줘야 합니다. routing 값을 question 도큐먼트의 id 값인 `question_id` 필드 값으로 지정합니다. 만약에 stackoverflow 인덱스에 샤드가 1개만 있다고 하면 routing 에 모든 도큐먼트에 적용되는 임의의 값을 넣어도 됩니다.

```
output {
  elasticsearch {
    index => "stackoverflow"
    document_type => "doc"
    document_id => "%{id}"
    routing => "%{question_id}"
  }
}
```

이제 데이터를 색인합니다.

> 중요! 반드시 `question` 도큐먼트들을 먼저 색인 한 뒤에 `answer` 도큐먼트들을 색인해야 합니다.

데이터 색인이 끝난 뒤 앞에서 실행했던 `has_parent` 쿼리를 이용해서 데이터가 정상적으로 나오는지 확인 해 봅니다.

```
GET stackoverflow/_search
{
  "query": {
    "has_parent": {
      "parent_type": "question",
      "query": {
        "term": {
          "id": {
            "value": "25691276"
          }
        }
      }
    }
  }
}

{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1,
    "hits": [
      {
        "_index": "stackoverflow",
        "_type": "doc",
        "_id": "25691509",
        "_score": 1,
        "_routing": "1",
        "_source": {
          "comment_count": 0,
          "owner": {
            "location": "Sao Paulo, Brazil",
            "id": 3337405,
            "display_name": "vinibarr"
          },
          "comments": [],
          "creation_date": "2014-09-05T18:01:23.033",
          "id": 25691509,
          "body": """
<p>You can load your data specifing the order columns that you're going to use into your table:
    ... 중략 ...
""",
          "qna_join": {
            "name": "answer",
            "parent": "25691276"
          }
        }
      }
    ]
  }
}
```

이처럼 `join` 타입의 필드를 사용해서 과거처럼 parent-child 구조를 만들 수 있으며, logstash의 `elasticsearch` 필터를 사용해서 데이터를 색인할 때 elasticsearch 에 있는 데이터를 가져와서 도큐먼트에 추가할 수 있습니다.