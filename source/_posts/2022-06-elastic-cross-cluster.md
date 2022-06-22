---
title: Elastic Cloud 에서 Cross Cluster 설치하기
tags:
  - Elastic
  - Engineering
  - Cross Cluster Search
  - Cross Cluster Replication
  - CCR
  - DR
  - 
subtitle: Elastic Cloud 에서 서로 다른 클러스터들 끼리 검색 및 색인을 할 수 있도록 설정하는 방법에 대해 설명합니다.
header-img: network.jpg
categories:
  - Engineering
date: 2022-06-22
---

Elasticsearch 의 가장 큰 시스템 단위는 클러스터 입니다. 기본적으로 클러스터가 다르면 서로 데이터 간섭이 일어나지 않으며 클러스터 마다 (정확히는 노드 마다) 서로 다른 접근 엔드포인트를 가지고 있습니다.

서비스에서 여러개의 서로 다른 클러스터를 동시에 운영하고 있거나, DR(Disaster Recovery) 구성, 멀티 리전 운영 등을 위해 한 클러스터에서 다른 클러스터의 데이터를 접속해야 하는 경우가 종종 있는데 Elastic 에서는 여러 클러스터의 데이터를 동시에 검색할 수 있는 [Cross Cluster Search](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cross-cluster-search.html) 과 서로 다른 클러스터의 데이터를 동기화 시키는 [Cross Cluster Replication](https://www.elastic.co/guide/en/elasticsearch/reference/8.2/xpack-ccr.html) 기능들을 지원하고 있습니다.

이 포스트에서는 Elastic Cloud 서비스 ([https://cloud.elastic.co](https://cloud.elastic.co)) 에서 Cross Cluster 기능들을 사용하는 방법에 대해 설명 드리려고 합니다.

## 같은 Elastic Cloud 계정

Elastic Cloud 서비스에는 Cross Cluster 기능을 사용하기 위해 Trust 라는 것을 설정해야 합니다. Elastic Cloud 관리 화면의 아무 클러스터 메뉴에 가서 Features 아래에 Trust 메뉴로 가면 **Trust all my deployments** 체크가 있습니다. 여기에 체크가 되어 있으면 같은 Elastic Cloud 계정 (Organization) 끼리는 기본적으로 Cross Cluster 기능을 허용하게 됩니다.

![](trust-all-my-depoly.png)

## 서로 다른 Elastic Cloud 계정

서로 다른 Elastic Cloud 계정 간의 클러스터들 끼리 Cross Cluster 기능을 활성화 하기 위해서는 각 클러스터에 대해 Trust 정보를 추가해야 합니다. **이 작업은 2개 클러스터 모두 진행해야 합니다.**

먼저 Elastic Cloud 관리 페이지에서 Cross Cluster 를 사용할 클러스터의 설정 화면 중 **Security** 메뉴로 이동합니다. 화면의 **Trusted deployments** 섹션에 있는 **Create trus** 버튼 클릭합니다.

![](create-trust.png)

이제 Trust 를 추가 할 Elastic Cloud Orgazniation(계정) 의 ID와 연결 해 줄 클러스터(Deployment) ID 를 입력해야 합니다. 이 정보들은 연결 할 클러스터 관리 화면에서 확인할 수 있습니다.

![](add-org-id.png)

연결할 클러스터의 우측 상단 아이콘의 드롭다운 메뉴에서 Organization 화면으로 이동하면 Organization ID 를 확인할 수 있습니다. 이 ID 를 복사해서 이전 클러스터 관리 화면의 Add organization ID 란에 넣어줍니다.

![](org-id.png)

**All deployments** 를 체크하면 연결한 organization 의 모든 Cluster 와 Cross Cluster 설정이 가능합니다. 특정 Cluster 만 허용하고 싶으면 **Specific deployments** 를 선택하고 Deployment ID 를 넣어줍니다. Deployment ID 는 연결할 클러스터 관리 화면으로 들어간 뒤 **URL 에 나타나는 32자의 문자열** 입니다. 이 문자열을 복사해서 붙여넣어줍니다.

![](dep-id-url.png)

**이 과정을 서로에 대해 각각 연결할 클러스터 모두 진행 해 줍니다**

## Remote Cluster 설정

이제 Organization 간의 Trust 작업을 마쳤으면 Elasticsearch 클러스터애 Remote Cluster 정보를 등록해야 합니다. Kibana 의 Stack Management 메뉴의 Remote Cluster 메뉴로 이동한 뒤 **Add a remote cluster** 버튼을 클릯합니다.

![](remote-cluster.png)

Name 은 원격 클러스터의 닉네임으로 사용할 이름을 적어줍니다.
그리고 **Manually enter proxy...** 를 체크하여 **Proxy address** 와 **Server Name** 을 입력해야 합니다.

![](add-mycrawl.png)

이 정보들은 연결할 Elastic Cloud 의 관리 화면 중 앞에서 보았던 Security 메뉴의 맨 아래 Remote cluster parameters 섹션에 있습니다. 값이 화면에 표시되지는 않으며 Prox address 와 Server name 링크를 클릭하면 각각 값들이 클립보드로 복사되어 위 Kibana 의 Add Remote Cluster 화면에 붙여넣기 하면 됩니다.

![](remote-cluster-param.png)

이제 원격 클러스터에 접속할 준비는 다 끝났습니다. 앞에서 만든 원격 클러스터 이름 뒤에 콜론 `:` 을 붙이고 인덱스를 넣어 원격 클러스터에 있는 데이터를 사용할 수 있습니다. 다음은 mycrawl 이라고 명명한 원격 클러스터의 googleplayreview 인덱스를 검색한 결과입니다.
```
GET mycrawl:googleplayreview/_search
```

![](remote-search.png)

다음과 같이 멀티테넌시로 원격 클러스터와 로컬 클러스터의 인덱스들을 한꺼번에 검색하는것도 가능합니다.
```
GET remote:my-index,my-index/_search
```

기본적으로는 같은 메이저 버전의 클러스터들 끼리 원격 클러스터로 설정이 가능하며, 이전 버전의 경우 가장 마지막 마이너 버전의 클러스터와의 연결이 지원됩니다.

![](ccr-suppert-matrix.png)

따라서 예를 들어 현재 8.2 버전 클러스터에서 7.11 버전 클러스터를 연결하고 싶으면 먼저 7.11 버전을 7.x 의 마지막 마이너 버전인 7.17 로 업데그레이드를 해야 합니다.

## 맺음말

Cross Cluster 기능은 하나의 서비스에서 여러 클러스터를 운영하는데 같이 검색이나 분석을 하거나 DR 구성등을 하는데 유용하게 쓸 수 있습니다. 그리고 지금 또 한가지 유용한 팁은, 현재 운영중인 클러스터를 업그레이드를 해야 할 때 운영중 업그레이드가 부담스럽다면 기존 서비스는 그대로 두고 최신 클러스터를 새로 만들어서 새로운 데이터는 새 클러스터로 색인하고 기존 데이터도 새 클러스터에서 예전 클러스터 데이터를 같이 조회하도록 하면 좀 더 안전하고 쉽게 최신 버전을 사용할 수 있습니다. 

메이저 버전이 다르다면 이전 클러스터는 메이저의 마지막 버전으로는 업그레이드 해야 합니다. Elastic Cloud 에서는 원클릭으로 바로 가능합니다.