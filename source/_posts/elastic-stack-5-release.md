---
layout: post
title: Elastic Stack 5.0 출시
subtitle: "Elastic Stack 5.0 출시 및 관련글 번역에 대하여"
date: 2016-11-26
header-img: "elastic-stack-5.png"
tags:
  - Elastic
  - Elasticsearch
categories:
  - Elasticsearch
---

지난 11월 26일에 Elastic Stack 5.0 버전이 출시되었습니다. 한달이나 지난 지금에 와서야 뜬금없이 포스팅을 한 이유는

<p style="font-size:1.6em">**드디어 한글 페이지 번역이 끝났습니다. (/;ㅁ;)/**</p>

5.0 출시를 맞아 Elastic의 홈페이지(https://elastic.co) 역시 새로 단장을 했습니다. 제품 페이지 부터 참 세련되게 바뀌었네요. :)

![](elastic-product-page.png)

지난달 [Elastic{on} Seoul Seminar](/2016/10/elastic-seoul-seminar) 라는 제 입사 이래 가장 큰 행사를 치뤘습니다. 때마침 그 날 Elastic Stack 5.0 이 발표가 되었고요. 본래는 5.0 출시에 맞춰 홈페이지 리뉴얼을 계획하고 있었고, 각 언어별로도 (Elastic 웹사이트는 영어, 프랑스어, 독일어, 일본어, 한국어로 서비스 되고 있습니다.) 미리 페이지들을 번역 해서 출시 하는 것을 목표로 하고 있었습니다. 그러나 저는 행사 준비가 바빠 미리 작업이 힘들었고 (사실 영어 외에 다른 나라 페이지들도 날짜 맞춰 나온 국가는 없었습니다. 😅) 우선 행사가 중요하니 지금은 어려워 행사 이후 최대한 빨리 번역 리뷰 작업을 하겠다고 저희 번역팀과 이야기 한 뒤 우선 행사 부터 치뤘습니다. 참고로 웹페이지나 블로그 번역은 저 혼자 하는건 아니고, 먼저 에이젼시에서 번역 해서 올라온 글을 제가 리뷰 하고 컨펌 하면 저희 홈페이지 웹마스터가 실 서버에 반영하는 방식으로 진행을 합니다.

그런데 Elastic{on} Seoul Seminar 행사 끝나고 나서 행사 참여하신 업체분들로 부터 자사를 방문해서 설명회 요청 및 계약사항에 대한 문의가 끊이질 않아, 행사 이후 한달여 기간 동안을 정말 미팅, 미팅, 미팅을 하며 정신 없이 보냈습니다. 덕분에 성사된 계약도 있고, 상당한 크기의 빅딜 문의도 들어오고 있고 해서 저와 저희 한국 직원들 모두 육체는 피곤하지만 상당히 신나게 정신없이 달렸던 것 같습니다. 제 눈에 혼자 5사람분의 영업은 뛰고 있는 듯 한 우리 정하원 지사장님과 입사 하시자 마자 큰 행사 치르고 지금 혼이 나갈 정도로 구르고 계신 김황곤 부장님... 사랑합니다.

참고로 행사에 참여 해 주신, 특히 개발자이신 분들로 부터 신제품 소개랑 회사 자랑만 있지 별다른 심도있는 기술 이야기가 없어 아쉬웠다 라는 피드백을 받았는데, 이번 행사가 회사를 널리 알리려는 일종의 세일즈 이벤트였기 때문에 이 부분에 대해서는 먼저 다시 한번 양해 부탁드리며, 올해 그리고 내년에 한국에서 비즈니스가 잘 되고 여러 좋은 사용 사례들이 많이 생기게 되면 내년에는 좀 더 알차게 준비 할 수 있을거라 생각됩니다. 이미 국내에서도 아주 탄탄하게 잘 쓰고 계신 분들도 많이 계시지만, 일단 저희 내부적으로 올해는 이정도 규모면 적당 했다 라고 평가 하는 중입니다.

여하튼, 행사 치르기 전 제게 리뷰 요청 온 번역 문서가 프로덕트 페이지 15개와 블로그 페이지 7개... 총 22개 페이지였습니다. 😱

번역된 페이지들은 아래와 같습니다. 대부분 영어로 이미 읽어보셨겠지만, 그래도 5.0 스택의 주요 변경 사항들을 잘 설명하고 있기 때문에 아직 안 보셨다면 한번씩 읽어 보시면 좋을 것 같습니다.

- **프로덕트 페이지**
  - Elastic Stack: https://www.elastic.co/kr/products
    - Elasticsearch: https://www.elastic.co/kr/products/elasticsearch
    - Kibana: https://www.elastic.co/kr/products/kibana
    - Beats: https://www.elastic.co/kr/products/beats
      - Filebeat: https://www.elastic.co/kr/products/beats/filebeat
      - Metricbeat: https://www.elastic.co/kr/products/beats/metricbeat
      - Packetbeat: https://www.elastic.co/kr/products/beats/packetbeat
      - Winlogbeat: https://www.elastic.co/kr/products/beats/winlogbeat
    - Logstash: https://www.elastic.co/kr/products/logstash
    - X-Pack: https://www.elastic.co/kr/products/x-pack
      - Security: https://www.elastic.co/kr/products/x-pack/security
      - Alerting: https://www.elastic.co/kr/products/x-pack/alerting
      - Monitoring: https://www.elastic.co/kr/products/x-pack/monitoring
      - Reporting: https://www.elastic.co/kr/products/x-pack/reporting
      - Graph: https://www.elastic.co/kr/products/x-pack/graph

- **릴리스 블로그**
  - Elastic Stack: https://www.elastic.co/kr/blog/elastic-stack-5-0-0-released
  - Elasticsearch: https://www.elastic.co/kr/blog/elasticsearch-5-0-0-released
  - Kibana: https://www.elastic.co/kr/blog/kibana-5-0-0-released
  - Logstash: https://www.elastic.co/kr/blog/logstash-5-0-0-released
  - Beats: https://www.elastic.co/kr/blog/beats-5-0-0-released
  - X-Pack: https://www.elastic.co/kr/blog/x-pack-5-0-0-released
  - ES-Hadoop: https://www.elastic.co/kr/blog/es-hadoop-5-0-0-released

한국 시간으로 오늘(11월 26일) 새벽에 드디어 실 웹페이지 서버에 반영 되었습니다. 아직 오탈자나 이상한 번역이 있을 수 있습니다. 제 책 출간 할 때 출판사 담당 편집장님과 리뷰 하면서도 느꼈지만 오탈자와의 전쟁은 끝이 없는 것 같습니다. 그 동안 낮에는 미팅 다니느라 번역 리뷰 할 시간이 없어 새벽 4시에 일어나 리뷰하고 그러면서 반영 된 것이라 뿌듯하네요. 😁 그리고 가끔 페이스북에도 올렸지만 저희 블로그 편집하는 직원이 워낙 영화나 아메코믹 덕후라서 일반인은 잘 알지도 못하는 표현을 자주 쓰는지라, 에이젼시 거쳐서 도저히 말이 안되는 이상한 문장이 온 것도 많아 애도 꽤 먹었습니다.

예를 들면, 로그스태시 관련 글을 쓰는 창시자인 Jordan Sissel도 항상 글 마무리를 ***Happy ‘stashing!*** 으로 끝맺음 하는데, 사실 이걸 뉘앙스를 살려서 한국말로 딱 번역이 어렵습니다. Logstash 가 Log + Stash 의 합성어인데, Log 는 흔히들 아시는 **시간 나열에 의한 기록물** 이라는 의미와 **통나무** 라는 의미의 두가지 뜻이 있습니다. (그래서 Logstahs 예전 마스코트가 콧수염이 난 통나무였습니다) Stash는 **은닉하다** 라는 의미로, 한국말로 더 확 와닿는 번역으로는 **짱박다** 라는 뜻인데, 그래서 결국은 **로그 기록을 짱박다** 라는, 정말 Logstash가 어떤 기능을 하는 프로덕트인지를 설명하는 데 있어서는 더할 나위 없이 적절한 이름입니다. 근데 이걸 한국어로 간단하게 설명하긴 쉽지가 않네요. 공식 웹페이지에 ***즐거운 짱박음 되세요*** 라고 번역할 수도 없고. 여튼, 그래서 ***즐거운 stashing 되시길 바랍니다!*** 라는 애매한 문장으로 번역이 되었네요.

![](logstash.png)

그리고 페이스북에서도 한번 주절거렸 던 것인데, ES-Hadoop 블로그에 필자가 ***May your TIMED_WAIT’s be few, and your Spark Streaming Jobs live long and prosper.*** 라는 표현이 있습니다. **May the force be with you** 라는 스타워즈 대사와 **Live long and prosper** 라는 스타트랙의 벌칸족 인사말을 섞어서 써 놓은 것입니다. 😩 대체 이걸 어떻게 번역하라고... 한글도 알아야 하고, 영어도 알아야 하고, 영화 대사도 알고 있어야 하고, IT 지식도 있어야 하는데... 결국에는 이것 역시 ***TIMED_WAIT가 적어짐이 함께하고, Spark Streaming 작업에 장수와 번영을 기원합니다.*** 라는 애매모호한 문장으로 번역을 했습니다. 사실 저 문장에 대해 미국에 계신 저희 프로덕트 매니저인 김보현님께 미국 친구들은 너무 이런 geek한 표현을 좋아한다고 했더니 본인도 저 (영어)문장 이해를 못 하겠다고 하신 기억이 납니다.

![](force.gif)  ![](livelong.gif)

올해 2월에 번역했던 [Elastic Cloud 관련 블로그](https://www.elastic.co/kr/blog/introducing-elastic-cloud-and-elastic-cloud-enterprise) 에서도 뜬금없이 포스트에 ***A Rose by Any Other Name...*** 라는 문장이 있어 번역에 애를 먹었던 것이 기억나네요. 구글링 했더니 저게 셰익스피어의 로미오와 줄리엣에 나오는 대사였습니다. 자세한 스토리는 해당 블로그 하단에 첨언을 해 놓았습니다.

여하튼, 지금 Elastic 에서 제가 하고 있는 일이 대략 고객 기술지원, 컨설팅, 번역, 커뮤니티 및 컨퍼런스 지원, 기타 잡무들인데 (개발 컨트리뷰션은 언제.. ㅠ_ㅠ) 회사 블로그 번역 리뷰 업무도 해 보면 나름 재밌고 흥미로운 일인 것 같습니다.