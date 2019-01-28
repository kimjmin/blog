---
title: Elastic Maps Service 한국지도 추가
tags:
  - Elastic
  - Elasticsearch
  - Elastic Maps Service
categories:
  - Engineering
subtitle: Elastic Map Service 에 한국 지도가 추가되었습니다. 한국 지역을 Region Map 으로 사용하는 방법에 대해 설명합니다.
date: 2019-01-28
header-img: "maps-bg.jpeg"
---

## Elasticsearch, Kibana 에서 위치정보 사용

Elasticsearch 에서는 숫자, 텍스트 뿐 아니라 지역 정보를 가지고 활용이 가능합니다. logstash 또는 ingest pipeline 의 geopoint 필터를 사용하면 IP 주소로부터 다음과 같은 정보들의 추출이 가능합니다.

![](region-info.png)

Elasticsearch에 지역정보를 저장하고 Kibana 에서 지역 정보를 표시하는 방법은 아래의 2가지가 있습니다.

![](kibana-vis.png)

Coordinate Map은 위의 `geoip.location` 필드의 `{ "lon":113.25, "lat":23.1167 }` 같은 geo_point 형식으로 저장된 데이터를 이용해서 지도에 표시하는 방법입니다. 이 정보를 기반으로 아래와 같이 지도에 원이나 사각형, 열지도(heatmap) 형식으로 수치를 표현하는 것이 가능합니다.

![](coordinate-map.png)

Region Map은 geo_point 가 아니라 term aggregation 을 이용해서 위의 `geoip.country_code` 필드의  `KR`,`US`,`CN` 같은 keyword 형식으로 저장된 필드의 값을 가지고 미리 정의된 geo-hash 영역에 대입하여 지도에 나타내는 방식입니다.

![](region-map.png)

Region Map에서 지원되는 지도들은 Options 의 Vector Map 항목에서 선택이 가능합니다.

![](select-region.png)

지원되는 지도와 데이터의 종류들은 Elastic Maps Service - https://maps.elastic.co 페이지에서 확인이 가능합니다.

![](ems-world.png)

## Elastic Maps Service 에 한국 지도 추가

기쁘게도 이번에 Elastic Maps Service에 한국 지도 2종이 추가되었습니다.

- South Korea Provinces
- South Korea Municipalities

기본적으로 Elastic Maps Service 는 위키데이터에 있는 정보들을 활용합니다. South Korea Provinces 의 경우에는 [wikidata](https://www.wikidata.org/wiki/Q884)에 있는 정보를 활용해서 8개의 도와 서울특별시, 광역시 정도 들을 표시합니다.

![](ems-sk-province.png)

South Korea Municipalities 지도는 시,군,구 단위까지의 정보를 표시할 수 있습니다. 이 정보는 wikidata에 없기 때문에 [박은정](https://github.com/e9t) 님으로부터 처음 작성된 [South Korea Maps](https://github.com/southkorea/southkorea-maps) 의 정보를 활용하고 있습니다. 이 중 [통계청 통계지리정보서비스 - KOSTAT](http://kostat.go.kr) 자료 부분을 활용합니다. 

![](ems-south-korea.png)

데이터의 출처와 라이센스는 Elastic Maps Service 의 깃헙 
https://github.com/elastic/ems-file-service/blob/master/sources/kr/README.md 
에 명시하고 있습니다.

맵 geo.json 파일은 South Korea Maps 에 있는 실제 맵 보다 사이즈를 줄이기 위해 2차 가공을 다시 한번 했습니다. KOGL 라이센스는 2차 가공을 허용한다고 명시하고 있어서 문제는 없다고 판단했습니다. Merge 기록은 아래에 있습니다.
https://github.com/elastic/ems-file-service/pull/74

이제 Kibana의 Region Map에서 대한민국 시군구 지도를 활용할 수 있습니다. options 에서 vector map을 South Korea Municipalities 으로 선택하고 분석할 keyword 값과 일치하는 join field를 선택하면 됩니다.
![](kibana-south-korea.png)

시군구 명칭과 코드들은 https://maps.elastic.co/#file/south_korea_municipalities 에서 확인할 수 있으며 [통계청 통계지리정보서비스](https://sgis.kostat.go.kr/contents/shortcut/shortcut_05_01.jsp) 에서 코드표를 다운로드 할 수 있습니다. **알림마당 > 자료신청 > 자료 다운로드** 페이지 하단에 **코드표 및 이용설명서** 를 다운로드 하시면 됩니다.