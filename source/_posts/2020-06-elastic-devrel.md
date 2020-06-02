---
title: Elastic 라이센스, 그리고 오픈 소스 에반젤리스트의 딜레마
tags:
  - Elastic
  - Engineering
  - SW Engineer
  - 오픈소스
  - 커뮤니티
  - DevRel
subtitle: Elastic 의 라이센스 종류에 대한 설명, 그리고 오픈 소스를 회사에서 에반젤리스트로 산다는 것에 대해 이야기 합니다.
header-img: community.jpg
categories:
  - Engineering
date: 2020-06-02
---

정말 오랬만에 글을 씁니다. 반년 넘게 글을 쓰지 않았었네요. 그 동안 새로운 집으로 이사도 했고 여러가지 개인적인 행사에 신경 쓰느라 블로그 관리에 많이 소홀했던 것 같습니다. 오랬만에 그 동안 해 보고 싶었던 오픈소스 라이센스와 비즈니스에 대한 이야기를 해 보려고 합니다.

Elastic 한국 커뮤니티를 시작 한 것이 인연이 되어 Elastic 의 직원으로 합류한지 어느덧 햇수로 5년이 되었습니다. 제 포지션을 Elastic 에서는 **Community Engineer** 라고 부르고 있고 어떤 곳에서는 **에반젤리스트(Evangelist)**, **디벨로퍼 에드버킷(Developer Advocate)** 또는 **데브릴(DevRel)** 등으로 불리우는 일을 하고 있습니다. 이 역할에 대해서는 마이크로소프트의 디벨로퍼 애드버킷이신 유정협(Justin Yoo) 님께서 아주 자세하게 소개를 해 주셨으니 궁금하신 분은 유정협 님의 글 - [디벨로퍼 아드보캇은 뭘 하는 사람인가요?](https://justinchronicles.net/ko/2020/03/14/what-do-developer-advocates-do) 를 한번 읽어 보시기 바랍니다.

Elastic 이 만든 프로덕트인 Elastic Stack 은 (ELK Stack 으로 더 잘 알려져 있는) 오픈소스로 잘 알려져 있습니다. 오픈 소스로 제품을 개발하는 회사들이 비즈니스를 하는 방법은 다양하지만, 대개는 다음의 과정을 많이 거칩니다.

1. 기술지원
2. 상용(유료) 기능 추가
3. Cloud 서비스 제공

Elastic 도 위의 과정을 거치면서 비즈니스를 진행 해 왔습니다. 물론 이 과정에서 다양한 제품과 새로운 서비스를 개발하였고, 다양한 회사들을 인수 하기도 하면서 제품과 서비스를 확장 해 왔습니다.

### Elastic Stack 과 솔루션

Elastic 의 근간이 되는 ELKB 의 핵심 기능은 모두 오픈소스입니다. 처음 Elastic 의 제품 포트폴리오는 아래와 같았습니다.

![Elastic Stack 포트폴리오](elkbx.png)

기본 스택인 Elasticsearch, Logstash, Kibana, Beats 는 전부 오픈소스로 제공이 되었고, 그 외의 Security, Alerting, Monitoring, Graph 같은 상용 기능은 X-Pack 이라는 이름의 플러그인 설치를 통해 제공이 되었습니다.

그 이후에 Elastic 은 ELKB 스택 전반에 걸쳐 APM, 머신러닝, SIEM 등의 기능과 서비스를 계속 해서 개발을 해 왔습니다. 이 각각의 기능, 제품들을 솔루션 이라고 명명을 하였고 이 모든것을 클라우드에 서비스를 하면서 포트폴리오를 다음과 같이 재 편성하였습니다.

![Elastic 솔루션](solutions.png)

또 솔루션이 종류가 점점 많아짐에 따라 최근에는 솔루션들을 

- [Enterprise Search](https://www.elastic.co/kr/enterprise-search)
- [Observability](https://www.elastic.co/kr/observability)
- [Security](https://www.elastic.co/kr/security) 

세 개의 카테고리로 구분을 하고 각 솔루션을 이 카테고리 안에 배치하였습니다.

![](3products.png)

> 스포일러 : 나중에는 Kibana 좌측 메뉴도 위 3개의 카테고리에서 펼침 기능으로 바뀔 예정입니다.

특히 Observability 는 APM, 모니터링, 메트릭, 로그분석 등을 아우르는 개념으로 저도 Elastic 에서 제품화 한 이름으로 처음 접하게 된 단어라 어떻게 한국어로 번역해야 할지 고민이 많았습니다. Elastic 한국 팀에서는 **Observability** 를 공식적으로 **통합 가시성** 이라는 단어로 번역해서 부르고 있습니다.

### Elastic 라이센스 - 특히 Basic 라이센스에 대하여

Elastic 제품들은 오픈소스 외에도 다음과 같은 다양한 라이센스들을 가지고 있습니다. 전체 내용은 Elastic의 [기술지원 구독](https://www.elastic.co/kr/subscriptions) 홈페이지를 방문하시면 볼 수 있습니다.

![Elastic 라이센스 - 오픈소스, 기본, 골드, 플래티넘, 엔터프라이즈](licenses.png)

Elastic 에서 제공하는 제품들 중 가장 이야기거리가 많은 것이 **Basic(기본)** 입니다. Basic 에 속한 기능들은 오픈소스 라이센스는 아니지만 무료로 사용할 수 있도록 제공되고 있는 것 들입니다. 대표적으로 [Canvas](https://www.elastic.co/kr/what-is/kibana-canvas) 같은 확장 기능과 [Search Profiler](https://www.elastic.co/kr/blog/a-profile-a-day-keeps-the-doctor-away-the-elasticsearch-search-profiler) 같은 개발자 도구, [Kibana 쿼리 자동 완성](https://www.elastic.co/kr/blog/improving-kibanas-query-language) 같은 편의 기능등이 대부분 Basic 라이센스에 속해 있습니다.

과거 X-Pack 플러그인을 제공하던 시기에는 플러그인을 Elasticsearch 그리고 Kibana 에 설치하고 라이센스 키를 적용하면 상용 기능들을 이용할 수 있었습니다. 이 시기에도 Basic 라이센스에 속한 기능들을 무료로 사용할 수 있었으나, X-Pack 을 먼저 설치 한 후 `elasticsearch.yml`, `kibana.yml` 에서 x-pack 을 Basic 으로 다시 명시해야 했기에 사용하는 분들이 많이 없었습니다. 그리고 플러그인을 한번 설치해야 한다는 것이 꽤 큰 장벽이었기 때문에 [Elastic 6.3 버전 부터는 Basic 라이센스의 기능들이 포함된 설치 파일을 기본 배포판에 포함시키고 Gold, Platinum 라이센스의 기능들 까지 모두 코드를 오픈하였습니다](https://www.elastic.co/kr/blog/doubling-down-on-open).

아파치 라이센스의 기능만을 포함하는 배포판은 다운로드 페이지에서 별도의 링크로 제공됩니다. 각 스택의 다운로드 페이지에 다음과 같은 링크를 찾을 수 있습니다.

![](oss-download.png)

Elastic의 Basic 배포판에는 상당히 많은 기능들이 포함되어 있습니다. 사실 Basic 을 넘어 Gold, Platinum 에만 속해 있는 기능은 실제로 얼마 되지 않을 정도입니다. 아래는 OSS, Basic, Platinum 의 Kibana 메뉴들을 각각 비교 한 것입니다.

![](kibana-compare.png)

물론 메뉴 개수 말고도 각 기능 안에서의 차이들이 있습니다. Machine Learning 의 경우 Basic 에서는 Data Visualizer 만 활성화 되고 Anomaly Detection 같은 기능은 Platinum 이상에서 활성화가 됩니다.

![](data-visualizer.png)

Basic 에서 제공하는 Data Visualizer 도 실제 운영을 할 때에는 상당히 편리한 기능입니다. Beats 나 Logstash 등을 이용하지 않고 Kibana 에서 파일을 업로드 하면 알아서 소스 파일을 분석해서 인덱스 매핑과 ingest pipeline 등을 자동으로 만들어 주는 기능입니다.

이 Basic 에 있는 기능은 모두 무료로 사용이 가능합니다. 무료로 배포 되고 깃헙에 코드도 공개되어 있지만 라이센스는 Elastic 사에 귀속되어 있기 때문에 몇 가지는 주의 하셔야 합니다.

Elastic 제품을 내재 한 제품으로 비즈니스를 하려면 Elastic사와 협약을 체결하지 않은 경우 OSS 버전으로만 비즈니스를 해야 합니다. SI, 클라우드 서비스, 컨설팅, 기술지원, 교육 등이 모두 포함됩니다. 그래서 클라우드 서비스 사업자들 중에 Elastic 사와 파트너 협약을 하지 않고 Elastic Stack 서비스를 제공하는 사업자들은 OSS 라이센스에 속한 기능만 서비스를 해야 합니다.

> 사실 Elastic 사에서 서비스 하는 공식 클라우드 (https://cloud.elastic.co) 가 아닌 다른 서비스들을 통해 Elastic 을 시작 해 보신 분들은 OSS 기능만 경험 해 보시고 Basic 이상의 기능이 있는 줄 모르는 경우가 많아 Elastic 직원의 입장에서는 상당히 안타까운 경우가 많습니다.

그리고 당연한 이야기지만 Basic 이상 기능의 소스 코드를 베껴서 자사 제품을 만드는 것도 하시면 안됩니다. 라이센스 전문은 [Elastic 깃헙](https://github.com/elastic/elasticsearch/tree/master/licenses)에 있습니다.

![](github-license-file.png)

그럼 Basic 라이센스는 쓰라고 만들어 놓은게 맞기는 한거냐? 라고 반문 하시는 분들이 계실것 같습니다. 네. Basic 라이센스는 Elastic 사가 사용자 분들을 위해 만들고 무료로 쓰시라고 제공 해 드리는 기능이 맞습니다. Basic 에는 심지어 APM, SIEM 같은 대부분 상용으로 구매해야 하는 도구들 까지 속해 있습니다. 다만 이 기능들의 **소스코드, 배포에 대한 권리, 비즈니스를 할 권리 등은 Elastic 사에 귀속**이 되어 있다는 점을 명심하시면 될 것 같습니다.

Elastic을 내재해서 APM, SIEM 같은 도구들을 만들어 서비스 하는 사업을 하시는 분들도 실제로 많이 계실텐데, Elastic 사도 그런 분들과 동등한 입장으로 (오픈소스) Elastic Stack 을 이용해서 APM, SIEM 등을 만들어 비즈니스를 하는 회사라고, Elastic 오픈소스 커뮤니티와 잠시 분리해서 생각 해 보시면 조금 이해가 되실 것 같습니다. 다만 Elastic 사는 자신들이 개발한 상용 기능 중 머신러닝을 이용한 이상징후 탐지 등은 유료로, APM, SIEM 등은 무료로 제공 하고 있는 것입니다.

저한테 Basic 을 사용해서 회사 시스템에서 쓰는 것은 문제가 없느냐는 질문을 주시는 분들도 자주 계십니다. 저 라이센스 전문을 읽어봐도 사실 Elastic 은 제품에 대한 소유권만 주로 명시하고 있지 어디까지 사용해도 된다고 속 시원하게 설명을 하고 있지 않습니다. 적절할지 모르겠으나, 얼마 전에 이 내용에 대해 이해를 도울 수 있을 것 같은 비유가 생각이 나서 한번 설명을 드려보겠습니다.

> COVID-19 때문에 동네 주민센터에서 주민들에게 무료로 손 소독제를 나누어 주기로 했습니다. 주민센터 앞에 손 소독제를 용기에 담아 쌓아놓고 필요하신 분들은 자유롭게 가져다 쓰세요 라고 팻말도 적어 놓았습니다. 
> 그런데 어떤 사람이 주민센터에서 가져온 손 소독제를 다른 용기에 담아 돈을 받고 팔기 시작했습니다.
> 주민센터에서는 필요하신 분들은 사용을 하라고 배부를 한 것입니다. 다른 곳에 가져다 팔지 말라고 따로 명시는 하지 않았지만, 만약에 위 판매자의 행동이 적절하지 않다고 판단이 되는 경우 법적인 조치를 취할 수도 있을 것입니다.

회사에서 APM, 로깅, 메트릭 등이 필요하면, Elastic 의 Basic 기능에서 제공되니 가져다 쓰시면 됩니다. 하지만 Elastic 에서 어떤 회사, 조직, 개인의 Basic 라이센스 기능의 사용이 Elastic 비즈니스에 부정적인 영향을 미친다고 판단이 되는 경우에는 어떤 조치를 취할 수도 있지 않을까 생각 해 봅니다.

### Elastic 커뮤니티 - 오픈 소스 에반젤리스트의 딜레마

[Elastic 한국 커뮤니티](https://www.facebook.com/groups/elasticsearch.kr)는 제가 Elastic 에 입사하기 전에 시작한 곳입니다. 저는 이 커뮤니티가 Elastic 사의 소유물이라고 생각하지 않으며, 다른 Elastic 직원들에게도 이 점을 항상 명확히 전하고 있습니다.

물론 저는 Elastic 사로 부터 월급을 받는 사람이라, 커뮤니티에는 가능하면 Elastic 사의 비즈니스에 도움이 되는 이야기를 해야 할 의무가 있습니다. 하지만 공지글에도 명시 했듯이 (Elastic 기술과 관련된) 모든 질문, 답변, 홍보 등을 환영합니다. Elastic 의 상용 기능과 경쟁을 하는 제품의 홍보나 추천도 제가 막을 권리는 없습니다. (저희께 더 좋다 라고 슬쩍 어필을 할 수는 있겠지요 ㅎ)

제가 Elastic 에서 일을 처음 시작할 때만 해도 에반젤리스트, 디벨로퍼 애드버킷으로 일하시는 분들이 그리 많지 않았던 걸로 기억하지만 지금은 어느정도 이름 있는 테크, 서비스 기업들은 디벨로퍼 애드버킷에 준하는 일을 하시는 분들이 많이 계신 것 같습니다. 만약에 Elastic 이 오픈소스가 아닌 상용 제품이나 서비스 만을 취급하는 회사였다면 저는 좀 더 마음 편하게 일을 할 수 있었을 것 같습니다. 하지만 저에게는 Elastic 이 오픈 소스를 하기 때문에 갖는 딜레마가 있습니다.

제게는 Elastic 직원으로서 Elastic의 비즈니스 (Basic 이상)을 어필해야 하는 의무가 있다고 스스로 생각을 하고 있습니다. 하지만 또 한편으로는 이 역할은 영업, 솔루션 아키텍트 분들께 맡기고 저는 광범위한 Elastic 커뮤니티 - *이는 Elastic 오픈소스와 Elastic 의 상용 서비스의 경쟁 제품들을 사용하는 사용자 분들, 그리고 이런 제품들을 개발하고 서비스하는 회사들 까지 모두 포함한다고 생각합니다* - 를 위해 일해야 하는 의무도 가지고 있습니다. 그래서 제가 아는 지식과 정보를 최대한 제공 해 드리고자 하면서도 Elastic 비즈니스(상용 제품, 기술지원, 컨설팅, ...)는 carnivalize 하지 않기 위해서 항상 신경이 쓰입니다.

제가 오픈 소스를 하는 회사의 에반젤리스트로서 가지는 딜레마는 이것입니다.

<font color=red>**Elastic의 가장 큰 경쟁 제품은 A사도, S사도, D사도 아닌 Elastic 오픈 소스 입니다.**</font>

저희 영업 분들이 Elastic 유료 기능, 기술 지원 등을 판매하려고 해도 오픈소스(그리고 Basic)이 워낙 잘 되어 있어서 팔기 힘들다는 것이 참 아이러니 합니다. 열심히 저희와 미팅 하면서 구매 검토 하시다가 오픈소스로 (기술 지원은 알아서) 하기로 결정했다는 통보를 받을 때면 마음이 착찹할 때도 많습니다.

대한민국은 Elastic 글로벌 사 입장에서 보면 매출이 1%도 되지 않는 작은 시장입니다. 하지만 [Elastic 홈페이지](https://www.elastic.co/kr)가 전부는 아니지만 한글을 서비스하고 있고, (현재는 8개국어를 제공하지만 한국어가 5번째로 제공되기 시작한 언어였습니다. 제 개인적으로 가장 뿌듯하게 생각하는 부분입니다) [한글 형태소 분석기](https://www.elastic.co/kr/blog/nori-the-official-elasticsearch-plugin-for-korean-language-analysis) 까지 본사에서 직접 개발 해서 관리 해 주고 있습니다. 그리고 대한민국에 6명의 기술지원 엔지니어분들이 계셔서 한국어로 기술지원 응대가 가능합니다. 앞으로도 한국에서 계속 Elastic의 비즈니스가 잘 되었으면 하는 바램입니다.

> 한국 시장을 위해서만 한국 기술지원 엔지니어분들을 채용 한 것이 아니라 APEC 리전 기준으로 채용을 했는데, 한국에 워낙 뛰어난 엔지니어 분들이 많으셔서 많이 뽑혔습니다. 🤓

두서 없이 적다 보니 꽤 긴 글이 되었네요. Elastic 제품과 커뮤니티를 사랑하는 한 사람으로서 Elastic 기술, 그리고 우리 Elastic 한국 커뮤니티가 계속해서 발전하기를 바라며 글을 마치겠습니다. 😘