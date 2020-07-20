---
title: Elastic Cloud 활용 2 - Elastic Cloud 마이그레이션
tags:
  - Elasticsearch
  - Elastic
  - AWS
  - Elastic Cloud
  - S3
  - Snapshot
subtitle: Elastic Cloud의 클러스터를 스냅샷과 리스토어 기능을 이용해서 다른 클러스터로 마이그레이션 하는 방법에 대해 설명합니다.
header-img: datacenters.jpg
categories:
  - Engineering
date: 2020-07-20
---

오늘은 Elastic Cloud 활용 시리즈의 두번째로 Elastic Cloud 에서 Snapshot & Restore 기능을 이용한 클러스터 마이그레이션에 대해서 알아보도록 하겠습니다.

> [1 - Amazon S3 에서 Elastic Cloud 로 데이터 수집하기](/2020/07/2020-07-aws-s3-to-elastic-cloud)
> 2 - Elastic Cloud 마이그레이션

### Elastic 클러스터 마이그레이션

시스템을 운영하다 보면 버전이나 장비 업그레이드, 클라우드 서비스 사업자 교체 등의 이유로 데이터를 통째로 옮겨야 하는 경우가 종종 발생합니다. Elasticsearch 에는 데이터 마이그레이션을 할 수 있는 다양한 방법들이 있는데 잘 알려진 방법으로는

  - [소스 데이터를 다시 색인](#소스-데이터를-다시-색인)
  - [data 디렉토리를 통째로 복사](#data-디렉토리를-통째로-복사)
  - [Logstash 또는 _reindex API 를 이용해서 다른 클러스터로 복사](#Logstash-또는-reindex-API-를-이용해서-다른-클러스터로-복사)
  - [Cross Cluster Search 를 이용해서 다수의 클러스터 운용](#Cross-Cluster-Search-를-이용해서-다수의-클러스터-운용)
  - [_snapshot, _restore API를 이용한 복원](#snapshot-restore-API를-이용한-복원)

등이 있습니다.

##### 소스 데이터를 다시 색인
저는 개인적으로 **소스 데이터를 다시 색인** 하는 것을 선호하는 편입니다. 시간이 가장 오래 걸리긴 하지만 Elasticsearch 새로운 장비나 버전에 대한 호환성 등에 신경 쓸 필요가 없고 마이그레이션 중 발생할 수 있는 문제를 체크 해 가며 가장 안전하게 할 수 있는 방법이기 때문입니다.

##### data 디렉토리를 통째로 복사
**data 디렉토리를 통째로 복사** 하는 방법은 보통 데이터 마이그레이션 보다는 플러그인 패치나 업그레이드를 할 때 주로 사용하는 방법입니다. `path.data` (디폴트는 홈 디렉토리의 `\data`) 내의 파일들은 그대로 두고 Elasticsearch 홈 디렉토리의 `\bin`, `\lib`, `\plugins` 등만 새 버전으로 덮어 씌운 뒤 노드를 다시 시작하면 데이터를 새로 색인 할 필요가 없이 바로 새 버전의 Elasticsearch 가 실행이 됩니다.

5.x 이상 부터는 메이저 버전으로의 롤링 업그레이드도 지원하기 때문에 클러스터 다운 없이 무중단으로 업그레이드도 가능합니다. 롤링 업그레이드 할 때는
샤드 리로케이션을 중단 -> 노드 재시작 -> 샤드 리로케이션 활성 -> 클러스터 상태 Green 으로 변경 확인 -> 다시 처음부터
를 반복해야 하기 때문에 중간에 순서가 꼬이지 않도록 조심해야 합니다. 자세한 순서와 명령들은 공식 도큐먼트에 잘 나와 있습니다.
https://www.elastic.co/guide/en/elasticsearch/reference/7.8/setup-upgrade.html

다만 이 때도 플러그인이라던가 depricated 된 API 등을 체크하지 않으면 샤드들이 살아나지 않을 수도 있습니다. 이것은 과거 6.4 버전때 제가 한번 실패했고

<iframe width="560" height="315" src="https://www.youtube.com/embed/P6ezu7FJthg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

다시 했던 경험이 있는데

<iframe width="560" height="315" src="https://www.youtube.com/embed/1NPnSJ5i0-M" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

위의 두 영상들을 보면서 확인하실 수 있습니다.

##### Logstash 또는 _reindex API 를 이용해서 다른 클러스터로 복사

**Logstash 또는 _reindex API 를 이용해서 다른 클러스터로 복사** 하는 것도 시간은 걸리지만 비교적 안전하게 데이터를 옮길 수 있는 방법입니다. 기존 클러스터의 _source 데이터를 이용해서 다른 클러스터로 다시 색인을 하는 것이기 때문에 기능적으로는 **소스 데이터를 다시 색인** 하는 것과 차이가 없습니다. 이 방법을 사용하기 위해서는 기존 클러스터의 인덱스들은 모두 _source 를 저장하고 있어야 합니다. (인덱스 mapping 에서 _source 를 저장하지 않도록 하는 설정도 있습니다.) 참고로 _reindex 로 재색인을 하는 경우 도큐먼트id 까지 동일하게 복사를 하며 Logstash 로 하는 경우 `docinfo` 라는 설정값으로 지정을 할 수 있습니다.

사용 방법은 Logstash 도큐먼트 페이지
https://www.elastic.co/guide/en/logstash/current/plugins-inputs-elasticsearch.html
에서 확인하실 수 있습니다.

##### Cross Cluster Search 를 이용해서 다수의 클러스터 운용
Elastic 클러스터 운영 경험과 노하우가 많이 쌓인 분들은 Cross Cluster Search 를 이용해서 여러 클러스터를 운영 해 가며 마이그레이션 하는 방법도 있습니다. 새로운 데이터는 새 버전으로만 색인을 하고 과거 클러스터와 새 클러스터를 한 클라이언트(Kibana) 에서 같이 검색하면서 사용하다가 과거 클러스터의 데이터들의 보관 주기가 끝나면 이제 새 클러스터로만 운영하는 방식입니다. 장기간의 계획으로 운영 되어야 하겠지만 잘만 활용 되면 매우 유용하게 사용할 수 있는 방법입니다.

Elasticsearch 는 5.3, Kibana 는 5.7 버전 부터 Cross Cluster Search 기능을 지원합니다. 아래는 제가 오래 전에 Cross Cluster Search 관련해서 발표했던 장표를 캡쳐 한 것입니다.

![Cross Cluster](cross-cluster-kibana.png)

### _snapshot, _restore API를 이용한 복원

Elasticsearch 에는 스냅샷을 찍어 인덱스를 저장하는 기능이 있습니다. 이 스냅샷을 다른 클러스터에서 복원(restore) 시키는 것으로 클러스터의 데이터를 옮길 수 있습니다. Snapshot, Restore 방법은 아래 도큐먼트에 있습니다.
https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshot-restore.html

스냅샷은 로컬 디스크나 공유 디스크, HDFS, Amazon S3, Azure, Google Cloud Storage 등에도 저장이 가능합니다.
https://www.elastic.co/guide/en/elasticsearch/plugins/5.5/repository.html

Elastic Cloud 에서 클러스터를 생성 할 때는 GCP, Azure, AWS 중에 선택이 가능합니다.

![Elastic Cloud 서비스 제공자](es-cloud-vendors.png)

Elastic Cloud 에서는 자동적으로 매 30분 마다 클러스터의 스냅샷을 해당 서비스 사업자의 스토리지(AWS의 경우 S3)에 자동으로 저장하고 있습니다. 클러스터에 문제가 생기면 인덱스를 삭제하고 과거 스냅샷의 복원이 가능합니다.

![Elastic Cloud 스냅샷](es-cloud-snapshot.png)

또한 Elastic Cloud 에서 새 클러스터를 생성할 때 다른 클러스터가 저장 해 놓은 스냅샷으로부터 클러스터를 생성하거나 가져와서 복원 하는 것이. 이는 운영 서비스 데이터를 복사해서 개발 클러스터를 만들거나 할 때 매우 유용합니다. 다만 <font color='orange'>같은 서비스 사업자와 같은 리전에 있는 클러스터의 스냅샷</font>만 사용 가능한 제약이 있습니다.

![Elastic Cloud 스냅샷 선택](es-cloud-sn-select.png)

아마도 이번에 Elastic Cloud 의 서울 리전 발표로 Elastic Cloud 를 사용하시는 분들이 서울 리전으로의 데이터 마이그레이션을 계획하고 계신 분들이 많으실겁니다. <font color='blue'>Elastic Cloud 관리 화면과 Kibana 에서 스냅샷 환경을 직접 설정하면 스냅샷을 다른 리전에서도 복원이 가능합니다<font>.

##### s3.client 설정

Amazon S3 를 스냅샷 리파지토리로 사용하기 위해서는 S3 클라이언트 정보를 설정해야 합니다. 기본적인 설정법은 아래 도큐먼트 페이지에 있습니다.
https://www.elastic.co/guide/en/elasticsearch/plugins/7.8/repository-s3-client.html

먼저 AWS IAM 에서 `AmazonS3FullAccess` 권한을 가진 사용자의 `Access Key` 와 `Secret Key` 가 필요합니다. 위의 도큐먼트에서는 이 값들을 elasticsearch-keystore 에서 설정하도록 안내를 하고 있습니다.

![](keystore-doc.png)

keystore 는 터미널에서 하는 작업이기 때문에 위와 같은 명령은 온프렘 환경에서만 가능합니다. 다행이 Elastic Cloud 에서는 Security 메뉴에서 keystore 값을 설정할 수 있도록 기능을 제공하고 있습니다. Security 화면의 Elasticsearch Keystore 섹션의 Add settings 버튼을 클릭하면

![](security-keystore.png)

오른쪽에서 아래와 같은 화면이 팝업되어 설정을 추가할 수 있습니다.

![](aws-access-key.png)

클라이언트 정보는 `s3.client.CLIENT_NAME.SETTING_NAME` 같은 형태로 설정합니다. 위 스크린샷에서 설정한 클라이언트의 이름은 `default` 이고 `access_key` 값을 지정 한 것입니다. 같은 방법으로 `s3.client.default.secret_key`도 저장을 합니다.

이제 Kibana 의 Stack Management 메뉴에 있는 Snapshot and Restore 메뉴에서 새로운 S3 리파지토리를 등록할 수 있습니다.

![](kibana-snapshot-1.png) ![](kibana-snapshot-2.png) ![](kibana-snapshot-3.png)

또는 Dev Tools 에서 _snapshot API 를 이용해서 바로 등록하는 것도 가능합니다.
```
PUT _snapshot/s3-repository
{
  "type": "s3",
  "settings": {
    "bucket": "my_bucket",
    "client": "default"
  }
}
```

이제 스냅샷을 실행하면 앞서 지정한 S3 버킷에 저장이 됩니다. 다른 클러스터에도 같은 S3 리파지토리를 등록하고 S3 버킷에 저장된 스냅샷을 복원할 수 있습니다.

더 자세한 사용법은 아래 영상을 참고하시기 바라며 안전하게 마이그레이션 성공하시길 바랍니다. 🤓

<iframe width="560" height="315" src="https://www.youtube.com/embed/YDDvdFqaIWU" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>