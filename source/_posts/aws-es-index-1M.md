---
layout: post
title: 데이터 1M 입력 테스트
subtitle: "EC2 에 elasticsearch 데이터 1백만건 입력 테스트"
date: 2014-01-24
header-img: "big-data-1086802_1920.jpg"
tags:
  - AWS
  - ICT
  - Elasticsearch
categories:
  - AWS
---

EC2 인스턴스에 elasticsearch 데이터 1백만건 입력하는데 소요되는 시간을 테스트 해 보았습니다.

비용 대비 c3.large 가 제일 낫네요. 저희 서버는 c3.large 로 시스템 구축하고 EBS 나 S3에 저장용량 사용하는걸로 해야겠군요.
![](test.png)

그런데 이렇게 놓고 보니 비용 대비 m3.medium 는 c3.large 랑 가격 거즘 비슷한데 용량, 퍼포먼스는 절반 이하네요. 누가 쓸라나. -_-;

결과 도큐먼트는 여기 있습니다. 앞으로도 AWS 테스트 결과는 여기에 계속 업데이트 합니다.
[http://goo.gl/kkxLLV](http://goo.gl/kkxLLV)