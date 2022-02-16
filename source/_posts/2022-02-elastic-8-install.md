---
title: Elastic 8.0 설치하기
tags:
  - Elastic
  - Elasticsearch
  - Kibana
  - Release
  - Engineering
subtitle: 새로 출시된 Elasticsearch 8.0 과 Kibana 8.0 설치 방법이 어떻게 달라졌는지 살펴봅니다.
header-img: elastic-8-release.png
categories:
  - Engineering
date: 2022-02-16
---

정말 오랬만에 블로그 포스팅을 하네요. 얼마 전에 드디어 Elastic 8.0 이 출시되었습니다. 6.x 릴리스 까지는 보통 1년 ~ 18개월 정도의 텀을 두고 비교적 빠르게 버전 업데이트를 했는데 7.x 릴리스는 2019년에 출시되고 나서 거의 3년 가까이 유지를 해 왔네요.

사실 7.0 에서 7.17 버전으로 오기 까지 마이너 업데이트를 하면서 변경된 기능들이 정말 많았습니다. [7.0 버전 출시 때 작성했던 포스트](/2019/04/2019-04-elastic-stack-7-release)에서도 이야기했지만, 보통 Elastic 의 기능 추가는 <font color='blue'>**마이너 릴리스**에서 기능들이 추가</font> 되고 <font color='red'>**메이져 릴리스**는 데이터 저장방식 등의 구조 변경에 초점을 맞추기 때문에 **오히려 기능들이 Deprecated 되거나 Expire**</font> 되는 경우가 많습니다.

이번 8.0 릴리스의 가장 큰 테마는 **NLP (자연어 처리)** 와 **Vector Search** 입니다. 저도 이 분야에는 알못이라서 차근 차근 공부 해 가면서 계속 공유 해 보도록 하겠습니다. 자세한 내용은 [Elastic 8.0 출시 블로그](https://www.elastic.co/blog/whats-new-elastic-8-0-0)를 참고하세요. 아직은 영문 버전만 올라와 있는데 곧 한국어로 번역되어 올라올 예정입니다.

Elastic 처음 설치 과정에도 몇가지 변경된 내용이 있습니다. 오늘 포스트에서는 Elasticsearch 와 Kibana 의 처음 설치 과정이 어떻게 변경되었는지에 대해 간단하게 한번 살펴보겠습니다.

현재는 (그리고 아마 앞으로도 계속) Elastic에서 하는 거의 모든 홍보, 영업이 대부분 Elastic Cloud 를 중심으로 이루어지고 있습니다. Elastic 공식 홈페이지(https://www.elastic.co) 에서도 대부분의 링크들이 클라우드 서비스 페이지(https://cloud.elastic.co) 로 연결되도록 유도하고 있고, 가입된 Elastic Cloud 계정을 로그인 정보로 이용해서 공식 교육 페이지라던가 내부 행사 등에 사용합니다.

하지만 여전히 온프렘으로 설치, 사용하시는 분들이 많이 계신걸로 알고, 오늘은 간단히 로컬에서 설치하는 방법 까지만 살펴보고, 클러스터 구성이나 추가적인 설정들은 이후 다른 포스트에서 계속 다루도록 하겠습니다. 설치 파일 다운로드를 위해서는 Elastic 공식 홈페이지(https://www.elastic.co) 에서 상단에 product 메뉴로 이동해서 Elasticsearch + Kibana 부분을 클릭합니다.

![](elastic-co-prod.png)

그리고 그 아래 Or Download ... 링크를 클릭합니다.

![](elastic-co-download.png)

그러면 시작 튜토리얼 페이지 (https://www.elastic.co/start) 로 이동하게 됩니다. 이제 여기서 안내하는대로 쭉 따라서 하시면 되는데, 먼저 elasticsearch 를 설치할 OS의 버전을 선택하고 아래 버튼을 눌러 elasticsearch 설치 파일을 다운로드 합니다.

![](elastic-co-start.png)

그 다음은 `tar` 또는 `zip` 명령을 이용해서 압축을 풀고 시작 튜토리얼 페이지에 나온 대로 명령들을 실행하시면서 설치하면 됩니다. 각 명령 옆에 있는 스니펫 카피 버튼을 누르면 클립보드에 명령이 복사되어 바로 붙여넣기를 하실 수 있습니다.

![](elastic-co-start-sniffet.png)

`bin/elasticsearch` 명령어로 실행을 하고 난 뒤, 터미널을 잘 보고 계시면 이전 버전까지는 보이지 않았던 다음과 같은 메시지가 하나 나타탑니다.

![](es-cli-1.png)

OSS 버전(7.11 부터 OSS 는 제공하지 않고 현재 최소 기능 버전은 Basic 입니다) 에서는 제공하지 않았고 Basic 에서도 선택적으로 활성화 가능했던 Security 기능이 이제 필수로 활성화가 됩니다. 슈퍼 관리자(superuser) 계정인 `elastic` 의 패스워드가 자동으로 생성 되고 그 외에 다른 노드, 클라이언트 및 에이젼트 연동에 필요한 암호화 키 정보들이 표시가 됩니다.

이제 Kibana를 실행시켜 봅니다. Kibana도 처음 실행하면 보안 설정을 마저 마무리 하라는 메시지가 터미널에 나타납니다.

![](kb-cli-1.png)

웹브라우저를 열고 kibana 화면에 들어가면 (디폴트 포트 :5601) `enrollment token` 을 입력하라는 화면이 보입니다.

![](kb-enroll-1.png)

여기 아까 elasticsearch 실행 터미널에 있던 enrollment 토큰을 복사해서

![](es-cli-2.png)

Kibana 화면의 enrollment 입력란에 붙여넣기 해 줍니다. 그리고 `Configure Elastic` 버튼을 클릭합니다.

![](kb-enroll-2.png)

그럼 또다시 Kibana verification code 를 입력하라는 화면이 나옵니다. 요즘은 6자리 숫자 인증이 공통 갬성인것 같습니다.

![](kb-num-verify-1.png)

이 숫자는 kibana 실행 터미널에서 볼 수 있습니다.

![](kb-cli-verify-num.png)

터미널에서 확인된 숫자를 kibana 화면에 입력 해 줍니다.

![](kb-num-verify-2.png)

그럼 이제 kibana 가 나머지 설정들을 진행합니다.

![](kb-processing.png)

Kibana 홈 디렉토리 아래애 `config/kibana.yml` 파일을 살펴보면 elasticsearch 와 연동에 필요한 암화 정보 들이 자동으로 추가되어 있는 것도 확인할 수 있습니다.

![](kibana-yml.png)

잠시 기다리면 이제 kibana 화면에 로그인 화면이 나타납니다. elasticsearch 터미널에 표시되었던 슈퍼 관리자 elastic 의 패스워드를 복사해서

![](es-cli-pw.png)

사용자 elastic, 패스워드는 붙여넣고 로그인 합니다.

![](kb-login-pw.png)

이제 예전처럼 elasticsearch 와 kibana 를 사용할 수 있습니다. 화면 디자인은 7.17 버전에서 크게 변한 것은 없습니다.

![](kb-dashboard.png)

메이져 릴리즈가 나왔는데 이전 버전과 화면이 동일한 것은 살짝 아쉽기는 합니다. 새 버전 느낌이 나도록 배경 색상 정도라도 바꿔주면 좋지 않았을까 하는 생각이 있지만, 7.x 에서도 마이너 버전이 업데이트 될 때 마다 바뀐게 많았기 때문에 8.x 도 앞으로 계속 마이너 릴리스가 나올 때마다 발전될것이라 기대 해 봅니다.

오늘 포스팅은 이것으로 마치고 다음에는 또 다른 8.x 내용에 대해 다루어 보도록 하겠습니다. 🤓