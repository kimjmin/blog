---
layout: post
title: 빅데이터
subtitle: "RDB, NoSQL 그리고 빅데이터"
date: 2015-04-24
header-img: "big-data-1086802_1920.jpg"
tags:
  - Data
  - ICT
categories:
  - ICT & Tech
---

며칠 전 오랬만에 대학 모교를 방문해서 은사님을 찾아뵈었습니다.

반갑게 담소를 나누며 요즘 제 근황을 말씀 드리는데 요즘 데이터 다루는데 관심이 있다는 이야기를 드리니 혹시 요즘 트랜드를 학부생 후배들이 쉽게 이해 할 만하게 한번 정리 해 줄 수 있는지 물어보셔서 저도 그 동안 여기저기서 보고 들은 것들 정리 할 겸 포스팅 올립니다.

요즘 “갑질” 이라는 단어를 많이 듣습니다. 예전에는 사회생활 하면서 고객을 응대 하면서 처음 알게 되는 단어였는데 OO유업, OO 항공사 등에서 발생한 사건들이 사회적 이슈가 되고 미생같은 드라마가 유행하면서 요즘은 초등학생들도 아는 단어가 되었습니다.

![](gab_ul.jpg)
> 이런게 갑질

갑과 을을 한마디로 쉽게 설명하면 돈 내는 쪽을 갑, 돈을 받고 물건이나 서비스를 제공하는 쪽을 을 이라고 합니다. 제품이나 서비스가 마음에 안 들면 돈을 쥐고 있는 갑은 을에게 불평 불만을 하고 각종 요구사항을 내어 놓습니다. 속된 말로 쫀다고 하죠.

하지만 을 중에도 간혹 갑이 감히 뭐라 할 수 없는 을 들이 있습니다. 이 회사 제품이 다른 회사 제품보다 너무 뛰어나서 아무리 비싸도 이 회사 제품을 구입할 수 밖에 없거나, 이 회사에서 만들어 놓은 기능들을 바꾸고 싶어도 거기에 내가 맞춰서 해야 하는 그런 을 들이 있습니다.

예를 들면 오라클이라던가
![](oracle-300x60.jpeg)
.
.
.
.
.
.
.
.

또는 오라클이라던가
![](oracle-300x60.jpeg)
.
.
.
.
.
.
.
.

그리고 오라클 같은 회사입니다.
![](oracle-300x60.jpeg)

오라클은 RDBMS-관계형 데이터베이스 관리시스템의 절대 강자로 오랬동안 군림 해 왔습니다. 흔히들 RDB 라고 부르는 관계 데이터베이스 모델은 1970년대에 나왔습니다. 행과 열 (튜플과 어트리뷰트)로 이루어진 테이블 형태의 데이터 구조와 그 데이터들 간에 연결을 정의한 이 모델은 현재 우리 세상에 존재하는 거의 모든 형태의 자료를 표현할 수 있는 정말 완벽에 가까운 모델입니다. 이 이후에도 객체지향 데이터베이스라던지 뭐 여러가지가 나오긴 했는데, 이 관계 데이터베이스가 너무 완벽해서 1970년대에 나온 이 모델이 구조의 큰 변화 없이 아직까지도 거의 대부분의 시스템에 쓰이고 있습니다.

이 완벽한 데이터 모델이 최근 들어 새로운 바람을 맞고 있습니다. 인터넷 사용 인구가 점점 늘어나고, 쌓이는 정보 또한 점점 많아지고 있는 가운데 2000년에 들어 SNS, 소셜 네트워크 서비스가 등장하게 됩니다. 모든 사람들이 웹에 접속해서 자기가 있는곳, 하고있는 일, 만나는 사람, 새로 알게 된 사실 등의 수많은 정보를 서로 공유하기 시작합니다.  어마어마한 양의 자료가 쏟아지며 데이터 빅뱅이 일어납니다. 그 전 까지는 블로그나 홈페이지 운영하는 사람들, 그리고 인터넷 언론사 등과 같이 웹에서 정보를 생산하는 생산자가 특정 되어 있었는데, 이제는 웹에 접속하는 모든 사람들이 정보를 생산합니다.

![](Hilbert_InfoGrowth.png)
> 전 세계에 저장된 데이터 용량 증가 추이 (출처: 위키피디아)

어떤 학자들은 인류의 기록 문화가 생기기 시작한 시점 부터 2000년도 중반까지 기록된 모든 자료의 양 보다 2000년도 중반 이후에 생성된 자료의 양이 더 많을 것이라는 이야기를 했습니다. 믿기 어렵지만 다음 그림을 보면 그 이야기가 수긍이 갑니다.

![](DatainOneMinute.jpg)
> 매 1분마다 생성되는 데이터의 크기 (출처: http://www.domo.com/blog)

위 그림은 2012년도 기준으로 매 1분마다 생성되고 있는 데이터의 양을 표현한 그림입니다. 제 기억에 그 당시 기준으로 페이스북에 업로드 되는 데이터의 양이 1초에 17GB 정도라고 했습니다. 이런 거대한 양의 데이터를 처리하기 위해서는 기존의 RDB와는 다른 방식의 처리 방법이 필요했습니다. 기존의 데이터 모델로 수집, 분석, 처리할수 있는 역량을 넘어선 거대한 규모의 데이터를 뜻하는 **“빅데이터”** 라는 단어가 등장하게 됩니다.

정부, 기업, 연구기관들은 분석을 위해 온갖 소스로부터 자료를 긁어 모읍니다. 대표적으로 SNS, 신문기사, 블로그, 각종 커뮤니티, 댓글, 이메일, 카카오톡 메시지.. 이것들이 전부 데이터 입니다. 그리고 데이터를 가공한 정보가 가치를 얻고 돈이 됩니다. 이 어마어마한 데이터를 처리하기 위해서 뭔가 거대하고 강력 한 녀석이 필요하게 되었습니다.

빅데이터에서 빼놓을 수 없는 시스템이 바로 [하둡(hadoop)](http://hadoop.apache.org/) 입니다. 더그 커팅과 마이크 캐퍼렐라가 2005년에 개발한 이 하둡은 현재 아파치 재단의 최상위 레벨의 프로젝트로 등재되어 있습니다. 하둡은 하둡 분산 파일 시스템(HDFS: Hadoop distributed file system), OS 레벨 추상화 기능(OS level abstractions) 그리고 맵리듀스(MapReduce) 엔진 등이 포함되어 있습니다.
![](hadoop-300x78.png)

요즘에야 TB급 하드디스크도 많지만, 2010년 까지만 해도 보통 하드디스크는 많아야 100~250GB 수준이었습니다. 그러나 앞서 언급 했듯이 데이터가 쌓이는 속도가 무시무시 하다보니 로그 파일 하나가 TB 단위로 쌓이게 됩니다. 그래서 이런 파일들을 저장하고 처리하기 위해서 하둡을 설치하고 클러스터링 해서 분산 시스템을 만들었습니다. HDFS는 100GB 짜리 서버 10개를 연결해서 1TB 서버 하나처럼 동작하게 하는 가상의 공간을 제공합니다. 추가적으로 클러스터에 서버를 계속 연결해서 용량을 점차적으로 확장시켜 나갈 수 있습니다.

보통 빅데이터 하면 하둡이라고 여기저기서 많이 언급하기 때문에 자세한 내용을 모르는 사람들은 하둡이 오라클 같은 데이터 관리 시스템인줄 알고 있지만 하둡은 빅데이터 처리를 위한 일종의 가상 디스크 공간입니다. 하둡에서 실행하기 위해 만든 하이브, 피그 같은 프로그램을 사용하거나 하둡에서 제공하는 라이브러리를 이용해 직접 프로그램 짜서 하둡 위에 올려놓고 빅데이터 분석을 하는겁니다. 하둡은 오픈소스이고 워낙 강력하다 보니 요즘 나오는 왠만한 데이터 분석 소프트웨어들은 거의 다 하둡에다가 쓰고 읽고 하는 연동 기능을 지원하고 있습니다.
![](Hadoop-Ecosystem1-1024x988.jpg)
> Hadoop Ecosystem (출처 : http://opensource.com)

빅데이터를 기존의 RDB 방식을 이용해서 처리하려고 하니 여러가지 문제가 있습니다. 가장 큰 문제는 수집된 데이터들의 출처가 각자 다르다 보니 데이터의 형태가 일정하지가 않습니다. 이 데이터들의 형태를 일일이 비교, 검토해서 스키마를 짜고 입력하기에는 처리해야 할 양이 너무 많습니다. 따라서 이런 형태가 다른 데이터들을 처리하기 위해 RDB방식을 넘어선 NoSQL 방식의 데이터 관리 기법이 주로 사용되게 되었습니다.

NoSQL 프로그램으로는 대표적으로 MongDB, HBase, Cassandra, DynamoDB, Redis 등이 있습니다. 데이터를 저장하는 구조는 프로그램들 마다 각각 다른데 NoSQL에는 크게 다음과 같이 4가지 방식이 있습니다.

### 1. Key:values Stores
해쉬 테이블에 유니크 key값을 저장하고 포인터를 이용해 데이터를 찾습니다. 구현 방법이 간단하지만 key가 아닌 value 값으로 검색하기가 어렵습니다.
***예: Tokyo Cabinet/Tyrant, Redis, Voldemort, Oracle BDB, Amazon SimpleDB, Riak***

### 2. Column Family Stores
key가 하나가 아닌 여러개의 데이터를 가리킵니다. 기존의 RDB 방식과 가장 유사하여 NoSQL이 유행하기 시작하던 초기 시절에 가장 많이 주목 받았으나 지금은 그 때보다는 인기가  많이 떨어진 것 같습니다.
***예: Cassandra, HBase***

### 3. Document Databases
key : value 구조를 기본으로 하되 value 에 또다른 key : value 셋이 저장되는 방식입니다. 주로 JSON 과 같은 형식으로 저장을 합니다. 데이터 구조가 프로그램에서 처리하기에도 용이하고 검색도 수월하여 현재 가장 널리 사용되는 방식입니다.
***예: CouchDB, MongoDb***

### 4. Graph Databases
분산 시스템을 이용해 시스템의 확장성을 중요시 한 모델입니다.
***예: Neo4J, InfoGrid, Infinite Graph***

> (참고 : [http://rebelic.nl/2011/05/28/the-four-categories-of-nosql-databases/](http://rebelic.nl/2011/05/28/the-four-categories-of-nosql-databases/))

제 생각에 요즘 가장 유행하는 NoSQL DB는 아무래도 MongoDB인 것 같습니다. NodeJS와 함께 MEAN Stack에 포함되어 주로 스타트업들이 많이 사용하고 있습니다. Redis도 관심 있게 볼 만한 제품입니다. 메모리 기반의 데이터 관리 시스템으로 데이터를 전부 메모리에 올려서 처리하기 때문에 속도가 정말 빠릅니다. (대신 메모리가 많이 소모되겠죠)

RDB를 사용해서 데이터를 저장 할 때는 먼저 스키마를 정의합니다. 예를 들면 테이블들을 학생, 학과, 과목, 수업 이런 식으로 설계하고 데이터를 서로 join을 해서 실제로 사용할 형태의 데이터를 만들어 냅니다. 각 테이블의 스키마에 맞지 않는 데이터는 넣을 수 없습니다. 필요하다면 스키마를 변경하던지 데이터를 변경해야 합니다.

그런데 빅데이터에서는 프로그램이 수집 해 오는 데이터는 형태가 모두 제각각 입니다. 트위터와 같이 140자 텍스트인 경우도 있고, 신문 기사처럼 제목과 내용, 그리고 댓글이 있는 article 형식 등 가지 각색입니다. 이런것들을 저장하기 위해서는 스키마가 없는 구조, schemeless를 지원하는 시스템이 필요합니다.

그럼 이렇게 서로 맞지도 않은 데이터를 저장해서 여기서 어떻게 정보를 추출을 할까요? 우선 이 데이터들을 모두 다 분리해서 쪼갭니다. 쪼개는 방법은 다양합니다. 공백 같은 화이트 스페이스로 쪼개기도 하고, 사전에 기반한 형태소 분석기로 쪼개기도 하고. 쪼갠 다음 같은 정보끼리 모아서 통계를 만들고 검색 사전을 만듭니다. 보통 이렇게 쪼갠 단위를 텀(term)이라고 합니다. RDB 같은 경우는 얼마나 테이블 접근 속도를 높이고 조인을 최적화 하는 것 등이 중요한 기술력인데, NoSQL 같은 경우는 보통 수집한 데이터를 어떻게 잘 쪼개느냐가 중요한 기술력입니다.

아무 하둡 서적을 구입해서 읽어보면 보통 예제로 wordcount 라는 프로그램을 실습 합니다. 입력된 문장을 단어별로 쪼개서 어떤 단어가 몇개 나왔는지 카운트와 함께 저장하는 프로그램입니다. 이렇게 쪼갠 데이터들을 어디다가 쓰는가 하면, 보통 통계, 트랜드 분석, 검색어 순위, 음원 인기 차트, 쇼핑몰에서 고객 취향 맞춤 서비스 분석 등에 사용합니다. 미국의 오바마 대통령이 빅데이터 분석을 선거 운동에 활용 한 것이 유명하며, 우리나라도 지난 대선, 총선 등에서 지역별, 연령별로 트랜드 분석등을 해서 언론에서 많이 발표를 했습니다.

온라인 쇼핑몰 같은 곳은 사용자들의 구매 패턴을 분석해서 맞춤 광고를 보여주는 식의 전략을 합니다. 예를 들어 A 라는 사용자와 B 라는 사용자가 지난 6개월 동안 구입 한 물건을 분석 해 본 결과 90%가 같은 제품군을 구매를 했다는 결과를 얻게 되면 A와 B를 같은 유형의 사용자로 분류를 합니다. 그리고 나서 어떤 새 제품이 나왔을 때 A 사용자가 그 제품을 구매했다면 B 사용자도 그 제품에 관심을 있을 것이라고 예측을 하고 B 사용자에게 그 제품의 광고를 합니다.

이처럼 빅데이터의 등장은 기업들의 경영과 영업 전략에 새로운 지평을 열었습니다. 2000년대 말 무렵에는 모든 IT 기업이 빅데이터가 우리의 미래다를 외치며 너도 나도 빅데이터를 도입하겠다고 나섰습니다.

그런데 여기서 문제가 발생하게 됩니다. 빅데이터라는 것이 좋다고 어디선가 듣고 오신 높으신 분들이 빅데이터, 빅. 뭔가 크고 강한 느낌이 나잖아요? 이거 쓰면 우리 시스템도 빨라지나보다 해서 기존에 RDB로 있던 시스템을 빅데이터로 바꾸라고 합니다. 정부 같은곳에서 사업 공고 내면 참여하는 회사들이 제안서에다가 우리 빅데이터로 구축하겠다고 합니다. 어딜 가나 사업 보고서에 빅데이터 라는 단어가 빠지질 않습니다.

사실 기존에 RDB로 잘 사용하고 있는 데이터들을 반드시 NoSQL 방식으로 바꿀 필요는 전혀 없습니다. 앞서 설명했듯이 NoSQL 기반의 빅데이터 분석은 사용자들의 패턴 같은 로그 분석, 또는 SNS의 트렌드 분석에 사용하는 것이 적합합니다. 데이터가 빈번하게 수정이 되거나 데이터의 무결성이 중요한 자료들. 인사시스템 자료라던지 계약정보 같은 자료들은 RDB로 보관 하는 것이 적합합니다.

RDB에서는 값을 수정 하는 경우 변경 사항에 해당 칼럼값만 손쉽게 변경이 됩니다. 하지만 보통 NoSQL 시스템에서는 문서의 일부 값을 수정하면 해당 부분만 업데이트 되는것이 아니라 그 문서 전체를 읽어 해당 부분을 수정하고 다시 문서 전체를 입력시킨 뒤 기존 문서를 삭제하는 방식으로 처리가 이루어지기 때문에 문서 수정이 빈번한 시스템에서는 NoSQL이 RDB보다 성능이 떨어집니다.

제가 생각하는 것이 정답은 아니지만 저는 보통 RDB는 영구적인 자료의 무결성이 중요하고 수정이 자주 일어나는 것. NoSQL은 영구적이 아닌 일시적으로 매일 쌓이는 로그 데이터 등을 특정한 목적을 위해 분석하는 용도에 사용하는 것이 적합하다고 생각합니다.

빅데이터가 전지전능 한 것은 아닙니다. RDB를 써야 할 곳과 빅데이터를 써야 할 곳을 제대로 구분할 줄 아는 것이 중요합니다. 그리고, 페이스북 정도 규모의 시스템을 만들게 아니라면 보통 학교에서 졸업 프로젝트를 한다거나, 스타트업 창업을 한다거나 하는 정도의 시스템은, 제가 장담하건데 왠만하면 MySQL이나 MariaDB 정도의 RDB로 충분 할 것입니다. 다시 한번 말씀드리지만 변화가 빠른 IT 세상에서도 RDB는 1970년에 나온 모델이 아직까지도 사용되고 있는 기술입니다. 앞으로도 계속 쓰일것이라 확신하고 있습니다.