---
title: 서울시 지하철 승하차인원 대시보드 v3 - 1. 데이터 색인
tags:
  - Elastic
  - Engineering
  - Seoul Metro
  - Kibana
  - Dashboard
subtitle: 서울 지하철 승하차인원 Kibana 대시보드를 2022년 새롭게 다시 만들어봤습니다. 소스 파일도 쉽게 배포합니다.
header-img: seoul-metro-line2.jpg
categories:
  - Engineering
date: 2022-07-02
---

> 서울시 지하철 승하차인원 대시보드 v3 - 1. 데이터 색인
> [서울시 지하철 승하차인원 대시보드 v3 - 2. 대시보드 생성](/2022/07/2022-07-seoul-metro-v3-2)
> 
<iframe width="560" height="315" src="https://www.youtube.com/embed/W_oU9JF5fsE" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

제가 Elastic Stack 을 처음 시작할 때 부터 사골처럼 우려먹고 있는 예제가 있습니다. 그런데 아직도 이것보다 좋은 예제 데이터셋을 찾지를 못했네요.

바로 서울시 지하철 승하차인원 대시보드입니다.

###### 2015년 EMOCON 에서 처음 발표한 버전 (Elasticsearch 2.x, Kibana 4.x)
- [ELK 스택을 사용한 서울시 지하철 대시보드 만들기](https://youtu.be/ec-XzM6_CgU)

###### 2019 년에 3편으로 나누어 다시 녹화한 시리즈 버전 (Elastic Stack 7.x)
- [Elastic Stack을 이용한 서울시 지하철 대시보드 다시 만들기 #1](https://youtu.be/ypsEZXVYLo4)
- [Elastic Stack을 이용한 서울시 지하철 대시보드 다시 만들기 #2](https://youtu.be/KzxpvgO8h5k)
- [Elastic Stack을 이용한 서울시 지하철 대시보드 다시 만들기 #3](https://youtu.be/h1EAH4hTsiw)

이 대시보드를 Elastic Stack 새 버전으로 다시 만들어보려고 합니다. 내용이 길어질 것 같아 이 포스트에서는 데이터 색인 부분만 다루도록 하겠습니다.

## 데이터 소스 파일

그 동안 소스 데이터는 서울시 공공데이터 포털에서 제공하는
- 서울시 역코드로 지하철역 위치 조회
- 서울교통공사 지하철 역명 다국어 표기 정보
- 서울교통공사 연도별 일별 시간대별 역별 승하차 인원

등의 json, csv 데이터를 받아 여기에서 Elasticsearch 에 색인하기 적합한 모양으로 [변환하는 프로그램](https://github.com/eskrug/elastic-demos/blob/master/seoul-metro-logs/01-data-process-ingest.md) 을 만들어 사용하고 있었습니다.
그런데 언제부터인가 서울시 공공데이터 포털에서 위 데이터들이 사라져서 찾을수가 없습니다. 또 [서울교통공사 홈페이지](http://www.seoulmetro.co.kr/kr/board.do?menuIdx=551)에서 제공하는 데이터도 어떤 년도는 xlsx 어떤 년도는 csv 처럼 파일 형식이 통일되어 있지도 않고, 컬럼도 어떤년도는 날짜가 A열에 있고 어떤 년도는 A열에 역 코드가 있는 등 데이터 좀 뒤죽박죽 아쉽게 되어 있었습니다.

![](metro-source-provide.jpg)

그래서 제가 이전에 만들었던 데이터들을 기반으로 하여 좀 더 다양한 데이터들을 찾아서 추가하고, 형식을 통일한 뒤 배포하기 쉬운 형태로 제작해서 직접 배포하기로 결정했습니다.

소스 파일은 아래의 캐글(Kaggle) 페이지에 있습니다. 내려받으려면 캐글 계정이 필요합니다.
https://www.kaggle.com/datasets/kimjmin/seoul-metro-usage
![서울 지하철 승하차 인원 Seoul Metro Usage Kaggle Page](kaggle-seoul-metro.png)

서울 지하철 승하차인원 대시보드는 그냥 보여주는 용도가 아니라, 항상 Elastic 을 처음 시작하시는 분들이 처음부터 쉽게 따라서 만들어 볼 수 있는 튜토리얼을 제공하는 것이 목적이기 때문에 소스 파일에 다 넣어놓지 않고 색인 때 다양한 과정을 통해 데이터를 제대로 만들어 가도록 했습니다. (캐글에서 제공하는 용량이 한정적이기도 했고요)

제공하는 파일들은 다음과 같습니다.

##### [seoul-metro-station-info.csv](https://www.kaggle.com/datasets/kimjmin/seoul-metro-usage?select=seoul-metro-station-info.csv)
서울 지하철 역별 메타정보 파일입니다. 사이즈는 약 85KB 이고 1~8호선 총 286 개의 역 정보가 저장되어 있습니다.
각 행의 정보는 다음과 같습니다.
- `station.code` : 역 내부 코드 - 각 로그 파일의 station_code 와 연결되는 키값 (238)
- `station.fr_code` : 역 공개 코드 (211-2)
- `line.num` : 호선 number (2)
- `line.name` : 호선 text (2호선)
- `line.name_sub` : 세부 호선 text (을지로순환선) - 2호선 처럼 한 호선이 을지로순환선, 성수지선 처럼 또 나눠진 것을 구분하기 위한 정보입니다.
- `line.station_seq` : 호선별 역 순번 (23) - 나중에 지도에 각 호선을 연결하여 그릴 때 사용하기 위한 호선별 순번입니다. **station.code** 는 역이 생긴 순서대로 매겨지기 때문에 예를 들어 두 역 사이에 나중에 역이 추가된다면 순번이 **24 - 29 - 25** 처럼 되어 버려서 지도에 그릴때 직선이 뒤죽박죽이 되어 버립니다.
- `station.name_full` : 역명 전체 (서울대입구(관악구청))
- `station.name` : 역명 "|" 로 구분 (서울대입구|관악구청) - 나중에 이 역을 ["서울대입구", "관악구청"] 같이 배열로 저장하기 위해 구분 한 것입니다.
- `station.name_chc` : 역명 한자 (서울大入口(冠岳區廳))
- `station.name_chn` : 역명 중국어 (首尔大学(冠岳区厅))
- `station.name_en` : 역명 영어 (Seoul Nat‘l Univ. (Gwanak-gu Office))
- `station.name_jp` : 역명 일본어 (ソウルデイック)
- `geo.latitude` : 좌표계 위도 (37.481247)
- `geo.longitude` : 좌표계 경도 (126.952739)
- `geo.sigungu_code` : 시군구코드 (11210) - [Elastic Map Service](https://maps.elastic.co/#file/south_korea_municipalities) 를 이용해서 표현하기 위한 값입니다.
- `geo.sigungu_name` : 시군구명 (관악구)
- `geo.addres_road` : 역 도로명 주소 (서울특별시 관악구 남부순환로 지하1822(봉천동))
- `geo.address_land` : 역 지번 주소 (서울특별시 관악구 봉천동 979-2 서울대입구역(2호선))
- `geo.phone` : 역 대표 전화번호 (02-6110-2281)

##### [seoul-metro-2015.logs.csv](https://www.kaggle.com/datasets/kimjmin/seoul-metro-usage?select=seoul-metro-2015.logs.csv) ~ [seoul-metro-2021.logs.csv](https://www.kaggle.com/datasets/kimjmin/seoul-metro-usage?select=seoul-metro-2021.logs.csv)
년도별 지하철 승하차 인원 집계 파일입니다. 각 파일의 사이즈는 100MB 를 넘지 않도록 했습니다. 이렇게 하면 Filebeats 나 Logstash, Elastic Agent 를 사용하지 않고 Kibana 에서 직접 파일 업로드 기능을 이용해서 색인 할 수 있습니다. (제한 100MB)
각 행의 정보는 다음과 같습니다.
- `timestamp` : 타임스탬프, 매 시작 정각값이며 ISO8601 형식으로 되어 있습니다 (2015-01-02T00:00:00.000+09:00)
- `station_code` : 역 내부 코드 - **seoul-metro-station-info.csv** 의 **station.code** 에 대입합니다. (151)
- `people_in` : 승차인원 (281)
- `people_out` : 하차인원 (311)

이제 [캐글 페이지](https://www.kaggle.com/datasets/kimjmin/seoul-metro-usage) 에서 Download 버튼을 눌러 소스들을 다운로드 합니다.
![](file-download.png)


## seoul-metro-station-info.csv 색인

이제 먼저 seoul-metro-station-info.csv 파일을 색인하겠습니다. 먼저 사용할 Elasticsearch 와 Kibana 를 준비합니다. Elastic Cloud 를 사용하여도 되고 직접 설치한 클러스터가 있으면 그것을 사용해도 됩니다.
간편하게 Kibana 의 Data Visualizer 의 파일 업로드 기능을 이용해서 색인 하겠습니다. Machine Learning 메뉴에서 File 을 선택하면 됩니다. 참고로 머신러닝 기능이지만 Basic 무료 라이센스에서 사용 가능합니다.

![](data-visualizer.png)

이제 seoul-metro-station-info.csv 파일을 드래그 & 드롭 하거나 파일찾기를 이용해서 업로드를 하면 Data Visualizer 가 파일을 읽어들여 필드명과 매핑을 적절하게 설정 해 줍니다. 업로드를 한 다음에 import 버튼을 클릭합니다.

![](data-visualizer-import.png)

우선은 기본 설정으로 색인을 한 뒤 필요한 부분은 **_reindex API** 를 이용해서 다시 하도록 하겠습니다. Data View 는 생성하지 않도록 **create data view 는 체크를 해제** 하고 색인할 인덱스 이름은 **seoul-metro-station-info-temp** 로 하겠습니다.
이제 import 버튼을 눌러 업로드 한 파일의 색인을 시작합니다.

![](data-visualizer-import-2.png)

285 개의 서울시 역 정보 도큐먼트 색인이 모두 끝났습니다.

![](data-visualizer-import-3.png)

Dev Tools 에서 데이터가 제대로 들어갔는지 확인 해 보겠습니다.

![](seoul-metro-station-info-temp-search.png)

#### seoul-metro-station-info 매핑 설정

데이터는 잘 들어갔지만 매핑을 손보아야 합니다. **seoul-metro-station-info** 인덱스를 만들면서 매핑을 아래와 같이 추가합니다.
```javascript
PUT seoul-metro-station-info
{
  "mappings": {
    "properties": {
      "geo": {
        "properties": {
          "addres_road": { "type": "text" },
          "address_land": { "type": "text" },
          "latitude": { "type": "float" },
          "longitude": { "type": "float" },
          "phone": { "type": "text" },
          "sigungu_code": { "type": "keyword" },
          "sigungu_name": { "type": "keyword" },
          "location": { "type": "geo_point" }
        }
      },
      "line": {
        "properties": {
          "name": { "type": "keyword" },
          "name_sub": { "type": "keyword" },
          "num": { "type": "byte" },
          "station_seq": { "type": "byte" }
        }
      },
      "station": {
        "properties": {
          "code": { "type": "short" },
          "fr_code": { "type": "keyword" },
          "name": { "type": "keyword" },
          "name_chc": { "type": "keyword" },
          "name_chn": { "type": "keyword" },
          "name_en": { "type": "text" },
          "name_full": { "type": "keyword" },
          "name_jp": { "type": "keyword" }
        }
      }
    }
  }
}
```

#### 필드 구조 정리를 위한 ingest pipeline

루트에 있는 필드들을 geo, line, station 필드의 하위 필드로 적절하게 나누어 저장을 할 것입니다. **seoul-metro-station-info-temp** 인덱스의 도큐먼트들을 **seoul-metro-station-info** 로 재색인 할 때 사용할 ingest pipeline 을 다음과 같이 입력합니다.
```javascript
PUT _ingest/pipeline/seoul-metro-station-pipe
{
  "processors": [
    { "set": { "field": "_id", "value": "{{station_code}}" } },
    { "set": { "field": "geo.location.lon", "value": "{{geo_longitude}}" } },
    { "set": { "field": "geo.location.lat", "value": "{{geo_latitude}}" } },
    { "convert": { "field": "geo.location.lon", "type": "float" } },
    { "convert": { "field": "geo.location.lat", "type": "float" } },
    { "split": { "field": "station_name", "separator": "\\|" } },
    { "split": { "field": "line_name_sub", "separator": "\\|" } },
    {"rename": { "field": "geo_addres_road", "target_field": "geo.addres_road" } },
    {"rename": { "field": "geo_address_land", "target_field": "geo.address_land" } },
    {"rename": { "field": "geo_latitude", "target_field": "geo.latitude" } },
    {"rename": { "field": "geo_longitude", "target_field": "geo.longitude" } },
    {"rename": { "field": "geo_phone", "target_field": "geo.phone" } },
    {"rename": { "field": "geo_sigungu_code", "target_field": "geo.sigungu_code" } },
    {"rename": { "field": "geo_sigungu_name", "target_field": "geo.sigungu_name" } },
    {"rename": { "field": "line_name", "target_field": "line.name" } },
    {"rename": { "field": "line_name_sub", "target_field": "line.name_sub" } },
    {"rename": { "field": "line_num", "target_field": "line.num" } },
    {"rename": { "field": "line_station_seq", "target_field": "line.station_seq" } },
    {"rename": { "field": "station_code", "target_field": "station.code" } },
    {"rename": { "field": "station_fr_code", "target_field": "station.fr_code" } },
    {"rename": { "field": "station_name", "target_field": "station.name" } },
    {"rename": { "field": "station_name_chc", "target_field": "station.name_chc" } },
    {"rename": { "field": "station_name_chn", "target_field": "station.name_chn" } },
    {"rename": { "field": "station_name_en", "target_field": "station.name_en" } },
    {"rename": { "field": "station_name_jp", "target_field": "station.name_jp" } },
    {"rename": { "field": "station_name_full", "target_field": "station.name_full" } }
  ]
}
```

이제 **seoul-metro-station-info-temp** 인덱스의 도큐먼트들을 **seoul-metro-station-info** 로 재색인 합니다.
```javascript
POST _reindex
{
  "source": {
    "index": "seoul-metro-station-info-temp"
  },
  "dest": {
    "index": "seoul-metro-station-info",
    "pipeline": "seoul-metro-station-pipe"
  }
}
```

아래와 같이 285 개의 도큐먼트가 created 되었다는 메시지가 나오면 성공입니다.
![](reindexed.png)

재색인 작업이 끝났으면 **seoul-metro-station-info-temp** 인덱스와 **seoul-metro-station-pipe** 인제스트 파이프라인은 삭제하셔도 됩니다.
```javascript
DELETE seoul-metro-station-info-temp
DELETE _ingest/pipeline/seoul-metro-station-pipe
```

## 데이터 확장을 위한 스크립트 작업

승하차인원 집계 로그 데이터를 색인하면서 다양한 변환 작업을 해 줄 것이기 때문에 이를 위한 준비를 먼저 해보도록 하겠습니다. 

#### hour_and_week 스크립트

나중에 만들 대시보드에 아래와 같이 요일별, 시간대별 값을 보는 차트를 만들 것입니다.
![](hours_weekdays.png)

그러기 위해서는 쿼리 시점에서 timestamp 필드값에서 요일과 시각 정보를 만드는 것 보다 미리 별도의 시각값과 요일값 필드를 만들어 각 도큐먼트에 넣어 두는 것이 성능이나 자원 활용 면에서 여러가지로 유용합니다. timestamp 필드로부터 요일과 시각 정보를 추출해서 저장하는 스크립트를 만들고 `_scripts` 에 `hour_and_week` 라는 이름으로 미리 저장을 하겠습니다.

```javascript
PUT _scripts/hour_and_week
{
  "script": {
    "lang": "painless",
    "source": """def ts=ctx[params['dateTimeField']];
def sdf=new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSS");
def date=sdf.parse(ts);
def cal=Calendar.getInstance();
cal.setTime(date);

ctx[params['hourOfDayField']]=cal.get(Calendar.HOUR_OF_DAY);

def dowNum=cal.get(Calendar.DAY_OF_WEEK)-1;
def dowEn=["Sun","Mon","Tue","Wed","Thu","Fri","Sat"][dowNum];
def dowKr=["일","월","화","수","목","금","토"][dowNum];

ctx[params['dayOfWeekField']]=["num":dowNum, "en":dowEn, "kr":dowKr];"""
  }
}
```
이 스크립트에 사용되는 파라메터 3가지는 다음과 같습니다.
- dateTimeField : 시각과 요일 정보를 추출할 date 타입의 필드명
- hourOfDayField : 시각 값을 저장할 필드명
- dayOfWeekField : 날짜 값을 저장할 필드명

이 스크립트를 테스트 하기 위해서 임시로 ingest pipeline 을 만들어봅니다. 이름은 temp_hourAndWeek 으로 했습니다.
```javascript
PUT _ingest/pipeline/temp_hourAndWeek
{
  "processors": [
    {
      "script": {
        "id": "hour_and_week",
        "params": {
          "dateTimeField": "timestamp",
          "hourOfDayField": "hour_of_day",
          "dayOfWeekField": "day_of_week"
        }
      }
    }
  ]
}
```
timestamp 필드에 date 타입의 값이 있으면 hour_of_day 와 day_of_week 에 값을 넣도록 합니다. 이제 이 ingest pipeline 을 _simulate 를 이용해서 테스트 해 봅니다.
```javascript
POST _ingest/pipeline/temp_hourAndWeek/_simulate
{
  "docs": [
    {
      "_source": {
        "timestamp": "2022-07-01T12:00:00.000+0900"
      }
    }
  ]
}
```
위와 같이 입력하면 아래와 같은 결과가 리턴됩니다.
```javascript
{
  "docs" : [
    {
      "doc" : {
        "_index" : "_index",
        "_id" : "_id",
        "_source" : {
          "hour_of_day" : 12,
          "timestamp" : "2022-07-01T12:00:00.000+0900",
          "day_of_week" : {
            "en" : "Fri",
            "num" : 5,
            "kr" : "금"
          }
        },
        "_ingest" : {
          "timestamp" : "2022-07-02T05:25:30.495605395Z"
        }
      }
    }
  ]
}

```
hour_of_day 필드에 시각 값인 12, day_of_week 필드에 하위 필드로 해당 날짜의 요일인 금요일이 영어, 한글 그리고 순번으로 입력 된 것을 확인할 수 있습니다. 순번을 넣은 이유는 나중에 시각화를 하고 정렬할 때 필요합니다. 저 값이 없으면 요일을 가나다 또는 알파벳 순으로 밖에 나열을 못합니다. (예: 금목수월일토화)

#### enrich pipeline process

승하차인원 집계 로그 파일이 색인될 때 앞서 만든 **seoul-metro-station-info** 인덱스의 정보를 가져와 조인 할 수 있도록 enrich 프로세서를 포함하는 인제스트 파이프라인을 만들겠습니다. 자세한 사용 방법에 대한 설명은 공식 도큐먼트의 [Enrich your data](https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest-enriching-data.html) 페이지를 참고하시기 바랍니다.

먼저 enrich policy 를 만들어야 합니다. seoul-metro-station-info 인덱스에서 `station.code` 필드와 일치하는 도큐먼트를 가져와 병합하는 **seoul-metro-info_policy** 를 만들고 활성(**_execute**) 해 줍니다.
```javascript
PUT /_enrich/policy/seoul-metro-info_policy
{
  "match": {
    "indices": "seoul-metro-station-info",
    "match_field": "station.code",
    "enrich_fields": [ "line", "station", "geo" ]
  }
}

POST /_enrich/policy/seoul-metro-info_policy/_execute
```

방금 만든 seoul-metro-info_policy 의 enrich 프로세서를 포함하는 인제스트 파이프라인을 만들고 문서를 테스트 해 봅니다.
```javascript
PUT _ingest/pipeline/seoul-metro-logs-pipe
{
  "processors": [
    {
      "enrich": {
        "policy_name": "seoul-metro-info_policy",
        "field": "station_code",
        "target_field": "info"
      }
    }
  ]
}

POST _ingest/pipeline/seoul-metro-logs-pipe/_simulate
{
  "docs": [
    {
      "_source": {
        "@timestamp": "2015-01-01T05:00:00.000+09:00",
        "station_code": 150,
        "people_in": 441,
        "people_out": 392
      }
    }
  ]
}
```

아래와 같이 데이터가 확장 되었으면 성공입니다.
![](enriched-log.png)

#### enrich, hour_and_week, date 파이프라인

이제 앞에서 만든 enrich 프로세서와 hour_and_week 스크립트, 그리고 그 외 필요한 프로세서들을 포함하는 **seoul-metro-logs-pipe** 파이프라인을 만듭니다. 앞에 만든 파이프라인과 이름이 중복되어도 덧씌워지기 때문에 상관 없습니다.

```
PUT _ingest/pipeline/seoul-metro-logs-pipe
{
  "description": "Ingest pipeline for seoul-metro-logs-%{+YYYY} index",
  "processors": [
    {
      "enrich": {
        "policy_name": "seoul-metro-info_policy",
        "field": "station_code",
        "target_field": "info"
      }
    },
    {
      "script": {
        "id": "hour_and_week",
        "params": {
          "dateTimeField": "timestamp",
          "hourOfDayField": "hour_of_day",
          "dayOfWeekField": "day_of_week"
        }
      }
    },
    { "rename": { "field": "info.geo.sigungu_name", "target_field": "geo.sigungu_name" } },
    { "rename": { "field": "info.geo.sigungu_code", "target_field": "geo.sigungu_code" } },
    { "rename": { "field": "info.geo.location", "target_field": "geo.location" } },
    { "rename": { "field": "info.station", "target_field": "station" } },
    { "rename": { "field": "info.line", "target_field": "line" } },
    { "date": { "field": "timestamp", "formats": [ "ISO8601" ], "timezone" : "Asia/Seoul" } },
    { "remove": { "field": [ "info", "station_code", "timestamp" ] } }
  ]
}
```
###### seoul-metro-logs-pipe 파이프라인은 다음과 같은 작업들을 합니다.
- `enrich` : seoul-metro-info_policy 인덱스에서 가져온 정보들을 info 필드의 하위에 넣습니다.
- `script` : @timestamp 필드로부터 hour_of_day, day_of_week 정보를 추출해서 입력합니다.
- `rename` : enrich 에서 가져온 info.geo, info.line, info.station 필드들을 바깥으로 옮깁니다.
- `remove` : 필요 없어진 info 필드와 중복 값을 가진 station_code 를 삭제합니다.

이제 다시 도큐먼트를 넣고 테스트를 해 봅니다.

![](pipe-simul.png)

## seoul-metro-%{+YYYY}.logs.csv 색인

#### seoul-metro-station-info 매핑 설정

이제 **seoul-metro-logs\*** 형식을 가진 인덱스가 색인될 때 자동으로 매핑을 적용할 인덱스 템플릿을 만들겠습니다.

```javascript
PUT _index_template/seoul-metro-logs_template
{
  "index_patterns": [ "seoul-metro-logs*" ],
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },
        "year": { "type": "integer" },
        "people_in": { "type": "integer" },
        "people_out": { "type": "integer" },
        "hour_of_day": { "type": "byte" },
        "day_of_week": {
          "properties": {
            "en": { "type": "keyword" },
            "kr": { "type": "keyword" },
            "num": { "type": "byte" }
          }
        },
        "geo": {
          "properties": {
            "sigungu_code": { "type": "keyword" },
            "sigungu_name": { "type": "keyword" },
            "location": { "type": "geo_point" }
          }
        },
        "line": {
          "properties": {
            "name": { "type": "keyword" },
            "name_sub": { "type": "keyword" },
            "num": { "type": "byte" },
            "station_seq": { "type": "byte" }
          }
        },
        "station": {
          "properties": {
            "code": { "type": "short" },
            "fr_code": { "type": "keyword" },
            "name": { "type": "keyword" },
            "name_chc": { "type": "keyword" },
            "name_chn": { "type": "keyword" },
            "name_en": { "type": "keyword" },
            "name_full": {
              "type": "keyword",
              "fields": {
                "nori": { "type": "text", "analyzer": "nori" }
              }
            },
            "name_jp": { "type": "keyword" }
          }
        }
      }
    }
  }
}
```

#### logstash 필터 설정

실시간 로그 수집은 Filebeats 또는 Elatic Agent 등이 잘 되어 있지만 간단한 커스텀 데이터를 색인하는 데에는 Logstash 가 아직까지는 수월합니다. Logstash 로 파일에서 데이터를 수집해서 csv 형식을 파싱하고 elasticsearch 로 내보내도록 다음과 같이 config 설정 파일을 만들어주도록 하겠습니다.

처음에는 output 을 `stdout { }` 으로 테스트 하면서 데이터가 제대로 확장되었는지 확인하고 나중에 `elasticsearch { }` 로 변경하는 것이 좋습니다.

```ruby
input {
  file {
    path => "/Users/kimjmin/elastic/source/seoul-metro/seoul-metro-*.logs.csv"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}

filter {
  # csv 파싱
  csv {
    source => "message"
    skip_header => true
    columns => [ "timestamp", "station_code", "people_in", "people_out" ]
  }
  # timestamp 필드로부터 year 값 추출.
  mutate {
    copy => { "timestamp" => "year" }
  }
  mutate {
    split => { "year" => "-" }
  }
  mutate {
    replace => { "year" => "%{[year][0]}" }
  }
  # 숫자 필드 타입 변경
  mutate {
    convert => {
      "station_code" => "integer"
      "people_in" => "integer"
      "people_out" => "integer"
      "year" => "integer"
    }
  }
  # 사용하지 않는 필드 삭제
  mutate {
    remove_field => ["@version", "event", "log", "host", "message"]
  }
}

output {
  # stdout { }
  elasticsearch {
    cloud_id => "seoul-metro-test:YXNpYS1ub3J0aGVhc3QzLmdj..."
    cloud_auth => "ingest:password"
    index => "seoul-metro-logs-%{[year]}"
    pipeline => "seoul-metro-logs-pipe"
  }
}
```

전체 파이프라인을 거치면서 데이터는 다음과 같이 변경됩니다.

![](data-pipelined.png)

이제 Logstash 를 실행하고 데이터가 정상적으로 색인되었는지 확인합니다.

![](seoul-metro-logs-search.png)