---
layout: post
title: Metricbeat 시작하기
subtitle: " Metricbeat 시작하기"
date: 2016-05-31 11:03:47
header-img: "IMG_6216.jpg"
tags:
  - Elasticsearch
  - Beats
categories:
  - ICT
---

# Metricbeat
metricbeat는 5.0.0 버전 부터 포함 되어 있으며 서버에 설치해서 주기적으로 서버의 다양한 프로세스 정보들을 수집하도록 만든 Beats 입니다. 기존에 있던 topbeat 역시 metricbeat에 포함되어 대치되었습니다. metricbeat에서 수집하는 데이터는 다음과 같습니다. 
- Apache
- MySQL
- Nginx
- Redis
- System
- Zookeeper


## Beat 프로젝트 내려받기.

### Elasticsearch 5.0.0-alpha2, Kibana 5.0.0-alpha2 내려받기
```
wget https://download.elasticsearch.org/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/5.0.0-alpha2/elasticsearch-5.0.0-alpha2.tar.gz
wget https://download.elastic.co/kibana/kibana/kibana-5.0.0-alpha2-darwin-x64.tar.gz
```

```
go get github.com/elastic/beats
cd $GOPATH/src/github.com/elastic/beats
cd metricbeat
```

Beat 컴파일.

```
make init
```
metricbeat.yml 등 파일 생성.

```
make
```
metricbeat 실행파일 생성.


elasticsearch, kibana 실행.

metricbeat mapping 생성
```
curl -XPUT 'http://localhost:9200/_template/metricbeat' -d@metricbeat.template.json
```

키바나 대시보드 복사.
```
../dev-tools/import_dashboards.sh -h
../dev-tools/import_dashboards.sh -d etc/kibana
```

metricbeat 실행
```
./metricbeat -e -c metricbeat.yml -d "publish"
```