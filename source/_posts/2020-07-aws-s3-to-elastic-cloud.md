---
title: Elastic Cloud 활용 1 - Amazon S3 에서 Elastic Cloud 로 데이터 수집하기
tags:
  - Elasticsearch
  - Elastic
  - AWS
  - Elastic Cloud
  - Logstash
  - S3
subtitle: Amazon S3 저장소에 있는 Static 웹페이지의 접속 로그를 Logstash 를 이용해서 파싱하고 Elastic Cloud 로 데이터를 수집하는 방법에 대해 설명합니다.
header-img: datacenters.jpg
categories:
  - Engineering
date: 2020-07-13
---

드디어 Elastic Cloud 의 서울 리전이 생긴다는 기쁜 소식입니다.

![Elastic Cloud 서울 리전 출시](es-seoul-blog.png)

Elastic Cloud 는 https://cloud.elastic.co 에서 쉽게 가입하고 시작할 수 있습니다. 서울 리전 런칭을 위해 사전 신청과 런칭 행사도 준비하고 있으니 관심 있는 분들은 아래 링크에서 신청 하시면 됩니다.🤓
https://events.elastic.co/2020-elasticcloud-seoul-promo

Elastic Cloud 활용에 유용한 정보들을 시리즈로 작성 해 보려고 합니다. 오늘은 첫번째인 Logstash를 이용한 AWS S3 에서 Elastic Cloud 로 데이터 수집하기 입니다.

> 1 - Amazon S3 에서 Elastic Cloud 로 데이터 수집하기
> [2 - Elastic Cloud 마이그레이션](/2020/07/2020-07-elastic-cloud-migration)

데이터 스토리지인 Amazon S3는 AWS 에서 가장 많이 쓰이는 서비스 중 하나입니다. 이번 포스트에서는 S3의 정적 웹사이트 호스팅 기능을 이용하는 웹 페이지 로그를 Logstash로 파싱해서 Elastic Cloud 로 수집하는 방법에 대해 설명하도록 하겠습니다.

Logstash 에 대해서 잘 모르시는 분들은 Elastic 공식 홈페이지에 있는 [Logstash 시작하기](https://www.elastic.co/kr/webinars/getting-started-logstash) 영상을 먼저 보시기 바랍니다.

> 이 포스트를 작성하고 있는 중에 Elastic 공식 홈페이지의 [Filebeat와 Elastic Stack을 사용해 S3의 AWS 로그 가져오기](https://www.elastic.co/kr/blog/getting-aws-logs-from-s3-using-filebeat-and-the-elastic-stack) 블로그가 한글로 번역이 되었습니다. 중복되는 내용이 있지만 같은 내용도 여러 번 보는 것이 도움이 되니 같이 보시면 좋을 것 같습니다.

그리고 앞으로는 가능하면 Elastic 관련 블로그는 활용 영상과 함께 올리도록 하겠습니다. 영상은 블로그 맨 아래에 있습니다.

### Amazon S3 정적 웹사이트의 접속 로그

블로그를 쉽게 쓰시려는 분들은 [네이버](https://section.blog.naver.com), [티스토리](https://www.tistory.com/), [미디엄](https://medium.com/) 같은 가입형 서비스를 편리하게 이용이 가능합니다. 또는 직접 서버를 호스팅 해서 [텍스트큐브](http://www.textcube.org), [워드프레스](https://wordpress.org/) 같은 설치형 블로그를 운영하시는 분들도 많이 계실겁니다. 그리고 비교적(?) 최근에는 마크다운을 이용해서 포스트를 작성하고 HTML로 빌드를 하여 업로드 하는 정적 호스팅형 블로그를 이용하시는 분들도 계실텐데요, 정적 호스팅 블로그 중 가장 널리 쓰이는 것은 아마도 Ruby 기반의 [지킬(jekyll)](https://jekyllrb.com) 일 것이고요, 저는 Node.js 기반인 [헥소(Hexo)](https://hexo.io/)를 사용하고 있습니다.

정적 블로그 호스팅은 데이터베이스나 미들웨어가 필요 없고 웹 서버만 있으면 되기 때문에 비교적 저렴한 비용으로 호스팅이 가능합니다. 깃헙에서도 무료로 정적 웹 페이지 생성 기능을 제공하고, Amazon S3 에서도 버킷에 있는 html 파일들로 정적 웹사이트를 호스팅 하는 기능을 제공합니다. 제 블로그도 S3의 정적 웹사이트 호스팅 기능을 이용하고 있습니다.

Amazon S3 의 접속 기록을 다른 버킷으로 저장하는 기능이 있습니다. 이 기능을 이용하면 S3의 정적 웹 호스팅 웹사이트의 접속 로그의 분석이 가능합니다. 저는 kimjmin.net 이라는 버킷에 블로그의 소스 파일을 올려놓고 kimjmin.net.logs 라는 버킷으로 접속 로그를 수집하고 있습니다.

![S3 정적 웹사이트 로그 수집 설정](static-access-log-bucket.png)

### Logstash S3 input

Logstash 는 100여개에 가까운 거의 모든 서비스와 시스템의 입력과 출력을 지원합니다. Logstash의 [S3 input plugin](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-s3.html) 기능을 이용해서 데이터를 수집 해 보도록 하겠습니다.

설정 가능한 옵션들이 많이 있지만, S3 input 에서 주로 사용하는 설정은 아래와 같습니다.
```
input {
  s3 {
    region => "ap-northeast-2"  
    bucket => "kimjmin.net.logs"
    prefix => "logstash-ingest-data/json/"
    access_key_id => "AKIAJ2ZIBNECS98IGD4A"
    secret_access_key => "TrcfgIAqw7V+fASZ23MrunoVSPef6dsfT4m/d8P7"
    interval => 10
  }
}
```

- **region** : 리전 코드. 서울은 `ap-northeast-2`
- **bucket** : 버킷명
- **prefix** : 버킷의 하위 경로
- **access_key_id** : AWS 액세스 키 ID
- **secret_access_key** : AWS 비밀 액세스 키
- **interval** : 리프레시 시간. 단위 - 초. 디폴트 - 60

먼저 S3 접속이 되는지 간단히 테스트를 위해 `stdout{ }` 으로 출력을 설정하고 Logstash 를 실행 해 보면 다음과 같이 S3 버킷으로 부터 데이터가 수집되는 것을 볼 수 있습니다.
```
input {
  s3 {
    ...
  }
}

output {
  stdout { }
}
```
![Logstash S3 수집 결과](s3.png)

> <font color="red">버킷 용량이 크면 처음 출력이 나타나기 까지도 오랜 시간이 걸리기 때문에 먼저 적은 용량의 샘플 데이터를 임시 버킷에 복사 해 놓고 테스트 하는것이 좋습니다.</font>

### Logstash filter 를 이용한 데이터 파싱

Logstash 는 `input`, `filter`, `output` 세 단계로 이루어집니다. Elastic 서울 오피스에서 제가 진행했던 세미나를 들으신 분들은 아마 실습 때 많이 해 보셨을텐데요, filter 의 `grok` 기능을 이용해서 메시지 스트림을 여러 필드로 분리하고, `geoip`, `useragent` 등을 이용해서 데이터를 더욱 확장할 수 있습니다. [Logstash 시작하기](https://www.elastic.co/kr/webinars/getting-started-logstash) 영상에서도 설명하고 있습니다.

##### Grok
아래와 같은 일반적인 아파치 로그를 수집한다고 할 때 
```json
14.49.42.25 - - [12/May/2019:01:24:44 +0000] "GET /articles/ppp-over-ssh/ HTTP/1.1" 200 18586 "-" "Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.2b1) Gecko/20091014 Firefox/3.6b1 GTB5"
```
보통은 해당 데이터는 message 필드에 문자열로 입력이 됩니다.
![](apache-log.png)

grok 필터를 이용해서 message 필드 패턴을 `COMBINEDAPACHELOG` 로 지정하면 다음과 같이 데이터를 파싱할 수 있습니다.
```json
filter {
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
}
```

![](apache-grok.png)

아파치 로그 처럼 Amazon S3 의 정적 웹 페이지 접속 로그도 지원하는 grok 패턴이 있는데, 바로 `S3_ACCESS_LOG` 입니다. logstash 설정에 다음의 filter 를 추가합니다.
```json
input {
  s3 {
    ...
  }
}

filter {
  grok {
    match => { "message" => "%{S3_ACCESS_LOG}"}
  }
}

output {
  stdout { }
}
```
이제 다시 Logstash로 S3 의 로그를 수집 해 보며 아래와 같이 데이터가 파싱된 것을 볼 수 있습니다.

![](s3-grok.png)

##### GeoIP

위 로그에 clientip 라는 필드에 접속한 클라이언트의 ip 주소가 있습니다. Logstash 는 `geoip` 필터를 이용해서 ip 주소로 부터 접속한 클라이언트의 위치 정보 (국가, 도시, 위도 및 경도 등)를 가져올 수 있습니다. 다음과 같이 filter 에 geoip 를 추가하면 geoip 라는 필드가 생성되고 하위 필드로 다양한 위치 정보들이 추가됩니다.
```json
...
filter {
  grok {
    match => { "message" => "%{S3_ACCESS_LOG}"}
  }
  geoip {
    source => "clientip"
  }
}
...
```

![](s3-geoip.png)

당연하지만 geoip 는 grok 아래에 와야 정상적으로 작동합니다.

##### User Agent

다시 위의 Grok 예제를 보면 agent 라는 이름의 필드에 클라이언트의 브라우저, OS, 디바이스 정보들이 문자열로 있는 것이 보입니다. `useragent` 필터를 이용해서 이 값을 구분 해 줄 수 있습니다. 다음과 같이 useragent 필터를 추가하면 user_agent 필드에 클라이언트 정보가 추가됩니다.
```json
...
filter {
  grok ...
  geoip ...
  useragent {
    source => "agent"
    target => "user_agent"
  }
}
...
```
![](s3-useragent.png)

##### Date

로그 데이터를 살펴보면 `@timestamp` 와 `timestamp` 필드 두 개가 있는 것을 확인할 수 있습니다. 

![](s3-date-1.png)

@timestamp 필드는 Elasticsearch 가 인식할 수 있는 날짜/시간 형태로 되어 있고, timestamp 필드는 문자열 값이며 Elasticsearch 가 디폴트로 이해할 수 있는 포맷도 아닙니다. 그러나 @timestamp 는 데이터를 읽어드리는 시점에 Logstash 가 기록한 값, 즉 로그가 출력되고 있는 현재 시간이고, 실제로 저 로그가 S3에 저장된 시간은 timestamp 필드에 있는 값입니다. 따라서 실제로 우리에게 필요한 값인 timestamp 필드를 Elasticsearch 의 날짜 형태로 바꿔서 저장 할 필요가 있습니다. 이 때 사용할 수 있는 것이 date 필터 입니다. 다음과 같이 date 필터를 추가하면 @timestamp 의 값이 timestamp 에 있는 날짜로 바뀌게 됩니다.

```json
...
filter {
  grok ...
  geoip ...
  useragent ...
  date {
    match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
  }
}
...
```

![](s3-date-2.png)

### Elastic Cloud 로 데이터 수집

이제 지금까지 만든 Logstash 설정을 이용해서 Elastic Cloud 로 Amazon S3 데이터를 색인 해 보겠습니다. Elasticsearch 로 데이터를 보내려면 output 구문에 elasticsearch { } 를 추가합니다. stdout { } 을 삭제하지 않으면 elasticsearch 와 콘솔 화면에 모두 출력할 수도 있습니다.
```
output {
  # stdout { }
  elasticsearch {
    hosts => ["localhost:9200"]
    user => "elastic"
    password => "changeme"
    index => "kimjmin-net-logs-%{+yyyy.MM.dd}"
  }
}
```

보통은 hosts, user, password 설정을 사용하지만, Elastic Cloud를 사용하면 이 설정 대신 `cloud_id` 그리고 `cloud_auth` 설정을 이용할 수 있습니다. cloud_id 는 Elastic Cloud 관리 화면에서 확인이 가능합니다.

![](cloud-auth.png)

```
output {
  elasticsearch {
    cloud_id: ":YXAtbm9ydGhlYXN0LTEuYXdzLmZvdW5kLmlvJDA5ZGUzMjVlMmU0YTRiMzJiZWM0NWRhNmMzOTJlZDdkJDZmNjExYTMzZjZkNTRhZDQ5ZGQyYWM2NGUzMjg0YjQ3"
    cloud_auth: "elastic:changeme"
    index => "kimjmin-net-logs-%{+yyyy.MM.dd}"
  }
}
```

이제 Elastic Cloud 에 있는 Kibana 에서 데이터가 입력 된 것을 확인합니다.

![](kibana-s3-data.png)

Elastic Cloud 로 데이터가 잘 수집이 되었습니다. 다음 포스트에서는 Logstash 를 Kibana 에서 관리할 수 있도록 설정하는 방법과 S3 접속 로그를 이용해서 Kibana 에서 대시보드를 만드는 부분을 다루어 보도록 하겠습니다.

<iframe width="560" height="315" src="https://www.youtube.com/embed/_kF0cLGpu7w" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>