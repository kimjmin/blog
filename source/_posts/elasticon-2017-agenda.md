---
title: Elastic{on}17 미리보기 
tags:
  - Elastic
  - Elasticsearch
  - 실리콘밸리
subtitle: Elastic{on} 2017 컨퍼런스 행사에서 주목할 만한 세션들 소개 
header-img: elasticon-conf-banner.jpeg
categories:
  - ICT
date: 2017-02-09 16:02:32
---

# Elastic{on}17이 한달 앞으로 다가왔습니다.

올해로 세번째를 맞는 Elastic 의 글로벌 이벤트인 Elastic{on}17이 3월 7~9일 샌프란시스코에서 열립니다.

![](elsaticon.png)

작년과 동일하게 [Pier48](https://www.google.co.kr/maps/place/Pier+48,+San+Francisco,+CA+94158+%EB%AF%B8%EA%B5%AD/@37.7756433,-122.3889372,17z/data=!3m1!4b1!4m5!3m4!1s0x808f7fdbafe3848f:0xaa0c4fa02144459!8m2!3d37.7756224!4d-122.3865937?hl=ko) 에서 열리고요, 행사 페이지와 등록은 : https://www.elastic.co/elasticon/conf/2017/sf 여기서 하시면 됩니다.

티켓 가격은 $1,495 이고요, 작년 오픈 직후에는 EarlyBird 할인, 그 다음은 New Year 할인, 지금은 Valentine 할인($1,395) 중입니다. 아마 현장 등록을 하는게 아니면 제값 내고 가는 경우는 없지 않을까 생각됩니다. 지금이라도 등록하실 분은 제게 따로 페이스북 쪽지나 트윗 등을 주시면 **추가 할인 코드** 를 드리겠습니다.

[아젠다 페이지](https://www.elastic.co/elasticon/conf/2017/sf/agenda)에 가 보시면 전체 시간표가 있고, 각각 세션 별로 설명과 URL이 구분되어 있어 공유하기 편하게 되어있네요.

![](agenda.png)

월요일은 교육이 있고 본 행사는 화요일 오후 키노트를 시작으로 본격적으로 오픈됩니다. 아마 저는 스피커가 아니어서 Demo 부스 아니면 AMA(Ask Me Anything) 부스에 배정될거라 예상되는데, 직원들 시간표도 나와 봐야 알 것 같습니다. 키노트에 발표될 키 테마가 뭘지 직원들에게도 공지는 안 되었지만 대충 뭐가 될지 예상은 됩니다. 머신러닝이랑, Elastic Cloud랑, 또 (읍...읍...)

여하튼, 이번 행사에서 관심있게 볼 만한 세션 몇가지를 블로그를 빌어 소개하려고 합니다. 세션들은 크게는 저희 내부 직원들이 소개하는 Elastic Stack의 기능들에 대한 내용들과, 외부 인사들이 소개하는 사용 사례들로 구분되어 있습니다. 외부 사례들은 `Application Search`, `Business Analytics`, `Log Analytics`, `Metrics`, `Security` 카테고리로 나뉘어 있고, 또 다시 산업 영역 별로 `교육`,`금융`,`요식업`,`건강`,`SW & Tech`,`유통`... 등으로 나뉘어 있습니다.

다음은 저희 회사에서 내부적으로 공유된 추천 세션들입니다.

#### Barclays - [The Engine for Key Security Platforms at Barclays](https://www.elastic.co/elasticon/conf/2017/sf/agenda?sess=the-engine-for-key-security-platforms-at-barclays) 
바클리스는 글로벌 금융 서비스 그룹인데 이번에 처음으로 자사의 보안 아키텍쳐에 적용된 대해 CIO가 직접 발표를 합니다. 금융 보안쪽에 관심 있는 분들이 들으시면 좋을것 같습니다.

#### Walmart - [Retail Fraud Detection and Real-Time Inventory Reporting ](https://www.elastic.co/elasticon/conf/2017/sf/agenda?sess=retail-fraud-detection-and-realtime-inventory-reporting--walmart)
월마트에서 소비자가 불법적인 방법으로 구매를 시도했던 것들을 탐지하는 fraud detection과 모니터링을 주제로 발표합니다. Dev-Ops 분들이 관심가질 만한 주제인 것 같습니다.

#### Tinder - [Using the Elastic Stack to Make Connections Around the World](https://www.elastic.co/elasticon/conf/2017/sf/agenda?sess=tinder-using-the-elastic-stack-to-make-connections-around-the-world)
틴더는 온라인으로 데이트 상대를 매칭시켜 찾아주는 서비스입니다. 틴더의 백엔드의 추천 엔진에서 Elasticsearch가 어떻게 동작하고있는지 한번 들어보는것도 재미있을 것 같습니다.

#### NVIDIA - [NVIDIA's User Experience Streaming Analytics](https://www.elastic.co/elasticon/conf/2017/sf/agenda?sess=data-intelligence-with-the-elastic-stack--scale-nvidias-user-experience-streaming-analytics)
엔비디아에서는 지포스의 프레임 스트리밍 데이터를 Elasticsearch 에 넣고 분석하고 있습니다. 고성능 게임 개발을 하는 분들이라면 꼭 들어보시길 추천합니다.

#### Blizzard - [Building a Near Real-Time Pipeline for All Things Blizzard](https://www.elastic.co/elasticon/conf/2017/sf/agenda?sess=building-a-near-realtime-pipeline-for-all-things-blizzard)
블리자드입니다. 말이 필요 없습니다. Elasticsearch로 World of Warcraft, Overwatch 등을 모니터링 하는 사례를 발표합니다.

#### Verizon Wireless - [The Elasticsearch Journey at Verizon](https://www.elastic.co/elasticon/conf/2017/sf/agenda?sess=the-elasticsearch-journey-at-verizon)
미국 최대 통신사인 버라이즌에서 Elastic Stack을 사용하여 시스템 모니터링을 구축한 사례를 발표합니다. 과거에는 시스템 문제가 발생했을 경우 리커버리 시간이 30분 정도였는데 Elastic Stack을 사용하고 부터는 리커버리 시간이 2~3분으로 줄었다고 합니다. 참고로 시스템이 다운되면 버라이즌이 입는 손실은 1분에 64,000 달러 정도라고 합니다.

#### IBM - [Localizing Kibana for the Global Language Landscape](https://www.elastic.co/elasticon/conf/2017/sf/agenda?sess=localizing-kibana-for-the-global-language-landscape)
IBM은 약간 의외의 주제로 발표를 하는데, 자신의 고객들에게 Kibana를 이용해 서비스를 하고 있습니다. 이것들을 어떻게 로컬라이징 시켰는지에 대해 이야기를 합니다.

#### Terradue - [Advancing Earth Science with Elasticsearch at Terradue](https://www.elastic.co/elasticon/conf/2017/sf/agenda?sess=advancing-earth-science-with-elasticsearch-at-terradue)
테라두드는 나사와 비슷하게 유럽쪽에서 인공위성, 우주탐사 사업을 지원하는 회사입니다. 인공위성에서 수집한 데이터를 가지고 지진, 홍수, 화산활동 등을 분석하는 일을 합니다. Elasticsearch가 자연과학 쪽에 어떻게 쓰이는지 궁금하신 분들은 들어보시면 좋을것 같습니다.

#### Uber - [Powering Uber Marketplace’s Real-Time Data Needs with Elasticsearch](https://www.elastic.co/elasticon/conf/2017/sf/agenda?sess=powering-uber-marketplaces-realtime-data-needs-with-elasticsearch)
개인적으로 정말 존경하는 개발자이고, 과거 넷플릭스에서 [suro](https://github.com/Netflix/suro)를 만드셨던 우버의 [배재현](https://www.facebook.com/jaehb)님께서 우버가 Elasticsearch를 가지고 어떻게 서비스에 적용하고 있는지 직접 발표를 하십니다.

이밖에도 많은 주제들로 재미있는 내용들이 준비되어 있어서 개인적으로 큰 기대가 됩니다. 참석 못 하시는 분들도 시간이 지나면 홈페이지에 영상과 발표 자료들은 공유 될 예정이니 나중에라도 관심 갖고 봐 주시면 좋을것 같습니다.
