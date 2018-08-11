---
title: Elastic Cluster 구성 2
tags:
  - Elastic
  - Elasticsearch
  - Elastic Cluster Settings
categories:
  - Elasticsearch
subtitle: 메모리, 네트워크 설정 및 플러그인 설치
date: 2018-01-02
header-img: "bg-linux-prompt.jpeg"
---

이번 포스트에서는 네트워크 설정 및 플러그인 설치에 대해서 다루도록 하도록 하겠습니다. 이전 또는 이후 내용들은 아래 포스트에서 확인하세요.

> [1. 서버 생성 및 Elasticsearch RPM 설치](/2018/01/2018-01-build-es-cluster-1)
> **2. 메모리, 네트워크 설정 및 플러그인 설치**
> [3. 클러스터 구성 및 마스터, 데이터 노드 설정](/2018/01/2018-01-build-es-cluster-3)
> [4. Kibana 설치 및 X-Pack Monitoring 확인](/2018/01/2018-01-build-es-cluster-4)
> [5. NFS 구성 및 elasticsearch 추가 설정](/2018/01/2018-01-build-es-cluster-5)
> [6. X-Pack Security를 이용한 SSL 및 TLS 설정](/2018/01/2018-01-build-es-cluster-6)
> [7. X-Pack License 적용 및 사용자 생성](/2018/01/2018-01-build-es-cluster-7)
> [8. Logstash 설치 및 Elasticsearch 기본 템플릿 설정](/2018/01/2018-01-build-es-cluster-8)

## Java Heap 메모리 설정.

Java Heap 메모리는 `jvm.options` 파일에서 설정합니다.

```
sudo vim /etc/elasticsearch/jvm.options
```

마스터 노드는 4GB, 데이터 노드는 8GB로 각각 설정을 할 예정입니다. 여기서는 우선 8GB로 설정 합니다.
```
-Xms8g
-Xmx8g
```

Java Heap 외에 시스템 메모리의 절반은 루씬 파일 캐시를 위해 남겨둬야 합니다. 자세한 설명이 아래 블로그들에 나와 있으니 한번은 꼭 읽어보도록 권해 드립니다.

- [Elasticsearch 인덱싱에 대한 성능 고려 사항](https://www.elastic.co/kr/blog/performance-considerations-elasticsearch-indexing)
- [Elasticsearch 2.0 인덱싱 성능 고려사항](https://www.elastic.co/kr/blog/performance-indexing-2-0)
- [A Heap of Trouble: Managing Elasticsearch's Managed Heap](https://www.elastic.co/blog/a-heap-of-trouble)

## 네트워크 설정

네트워크 설정은 `elasticsearch.yml` 설정 파일의 `network.host` 부분을 수정합니다.

```
sudo vim /etc/elasticsearch/elasticsearch.yml
```

보통은 `network.host: 192.168.0.1` 과 같은 형식으로 IP 주소를 직접 입력해도 되지만, 더 간편하게 `_local_`, `_site_`, `_global_` 같은 값 들을 이용할 수도 있습니다. 저희 서버는 아래와 같이 설정하였습니다.

```yml
network.host: _site_
```

network.host 의 값들에 대해서는 아래 페이지를 참고하시기 바랍니다.
https://www.elastic.co/guide/en/elasticsearch/reference/6.1/modules-network.html#network-interface-values

## Bootstrap Check

기본적으로 아래 문서에 나와있는 설정들은 모두 확인 하도록 합니다.
https://www.elastic.co/guide/en/elasticsearch/reference/6.1/important-settings.html


### bootstrap.memory_lock 활성

`elasticsearch.yml` 설정 파일에서 `bootstrap.memory_lock` 을 활성화 합니다.

```yml
bootstrap.memory_lock: true
```

설정 후 elasticsearch를 재시작하면 실행에 실패하는 경우가 있습니다. 시스템 로그를 보면 친절하게 어떻게 설정을 해 줘야 하는지 안내하고 있습니다.

```
[2017-12-29T06:16:43,809][WARN ][o.e.b.JNANatives         ] Unable to lock JVM Memory: error=12, reason=Cannot allocate memory
[2017-12-29T06:16:43,811][WARN ][o.e.b.JNANatives         ] This can result in part of the JVM being swapped out.
[2017-12-29T06:16:43,811][WARN ][o.e.b.JNANatives         ] Increase RLIMIT_MEMLOCK, soft limit: 65536, hard limit: 65536
[2017-12-29T06:16:43,812][WARN ][o.e.b.JNANatives         ] These can be adjusted by modifying /etc/security/limits.conf, for example:
	# allow user 'elasticsearch' mlockall
	elasticsearch soft memlock unlimited
	elasticsearch hard memlock unlimited
[2017-12-29T06:16:43,812][WARN ][o.e.b.JNANatives         ] If you are logged in interactively, you will have to re-login for the new limits to take effect.
...
[1] bootstrap checks failed
[1]: memory locking requested for elasticsearch process but memory is not locked
```

`/etc/security/limits.conf` 파일을 열고
```
sudo vim /etc/security/limits.conf
```

`elasticsearch soft memlock unlimited`, `elasticsearch hard memlock unlimited` 내용을 추가 해 줍니다.
```
...
#ftp             hard    nproc           0
#@student        -       maxlogins       4

elasticsearch soft memlock unlimited
elasticsearch hard memlock unlimited

# End of file
```

### Unicast 설정

다른 노드들이 마스터 노드와 연결될 수 있도록 `discovery.zen.ping.unicast.hosts` 부분을 마스터노드의 ip 주소로 입력 해 줍니다. 네트워크 주소는 `ifconfig` 또는 `ip addr` 명령으로 확인합니다.

```
discovery.zen.ping.unicast.hosts:
  - 192.168.1.10:9300
```

위 예문에는 `192.168.1.10` 이라고 적었지만, 실제로 설치된 서버의 IP 주소를 적으면 됩니다.

### ⚠️ Split Brain 문제

저는 마스터 노드를 1개만 운영 할 것이기 때문에 `discovery.zen.minimum_master_nodes` 설정은 따로 하지 않았습니다.

보통 노드가 10개 내의 클러스터는 마스터 노드를 따로 구분하지 않고 데이터 노드 중 임의의 노드가 마스터 역할을 병행해서 수행하도록 해도 큰 문제는 없습니다. 10개 이상의 노드로 구성된 클러스터인 경우 마스터 전용 노드와 데이터 전용 노드를 분리하는 것이 좋으며, 이 때 마스터 기능의 수행이 가능한 후보(master-eligible) 노드를 3(또는 그 이상의 홀수)개를 두어 실제 마스터 노드가 다운된 경우 다른 노드가 그 역할을 대신 할 수 있도록 합니다. 2개만 두는 경우에는 네트워크 단절로 인한 클러스터 분리 문제 (quorum)로 인해 하나의 클러스터가 서로 다른 마스터를 가진 2개의 클러스터로 나누어 져서 나중에 동기화 문제가 생길 수 있습니다. 이를 [Split Brain](https://www.elastic.co/guide/en/elasticsearch/reference/6.1/modules-node.html#split-brain) 이라고 합니다.

마스터 후보 노드를 3개(또는 그 이상의 홀수)로 두는 경우에는 네트워크 단절로 인해 클러스터가 분리가 되면 마스터 후보가 2개인 클러스터만 실제로 동작하고 1개인 클러스터는 동작을 멈추게 됩니다. 그렇게 해서 다시 네트워크가 복구 되었을 때 활성 상태였던 클러스터 노드들의 업데이트 정보가 비활성 상태였던 클러스터 노드들로 자연스럽게 동기화가 될 수 있습니다.

## X-Pack 설치

X-Pack 설치에 대한 내용은 아래 도큐먼트를 참고해서 진행합니다.
https://www.elastic.co/guide/en/elasticsearch/reference/current/installing-xpack-es.html

### X-Pack 플러그인 설치

elasticsearch 가 설치된 디렉토리에서 `bin/elasticsearch-plugin install x-pack` 명령으로 설치가 가능합니다. X-Pack 그리고 대부분의 플러그인들은 모든 노드들에 동일하게 설치가 되어 있어야 합니다.

```
[ ]$ cd /usr/share/elasticsearch
[ ]$ sudo bin/elasticsearch-plugin install x-pack
-> Downloading x-pack from elastic
[=================================================] 100%
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@     WARNING: plugin requires additional permissions     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
* java.io.FilePermission \\.\pipe\* read,write
* java.lang.RuntimePermission accessClassInPackage.com.sun.activation.registries
* java.lang.RuntimePermission getClassLoader
* java.lang.RuntimePermission setContextClassLoader
* java.lang.RuntimePermission setFactory
* java.net.SocketPermission * connect,accept,resolve
* java.security.SecurityPermission createPolicy.JavaPolicy
* java.security.SecurityPermission getPolicy
* java.security.SecurityPermission putProviderProperty.BC
* java.security.SecurityPermission setPolicy
* java.util.PropertyPermission * read,write
* java.util.PropertyPermission sun.nio.ch.bugLevel write
See http://docs.oracle.com/javase/8/docs/technotes/guides/security/permissions.html
for descriptions of what these permissions allow and the associated risks.

Continue with installation? [y/N]y
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@        WARNING: plugin forks a native controller        @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
This plugin launches a native controller that is not subject to the Java
security manager nor to system call filters.

Continue with installation? [y/N]y
Elasticsearch keystore is required by plugin [x-pack], creating...
-> Installed x-pack
```

### X-Pack Security 패스워드 설정

5.x 에서는 기본적으로 슈퍼유저 계정인 elastic에 패스워드 changeme 가 기본적으로 생성되었으나, 6.0 부터는 기본 패스워드가 생성되지 않습니다. X-Pack 설치 이후에는 바로 `setup-passwords` 프로그램을 이용해서 주요 시스템 계정인 `elastic`, `kibana`, `logstash_system`의 패스워드를 생성 해 줘야 합니다.

```
[ ]$ cd /usr/share/elasticsearch
[ ]$ sudo bin/x-pack/setup-passwords interactive
Initiating the setup of reserved user elastic,kibana,logstash_system passwords.
You will be prompted to enter passwords as the process progresses.
Please confirm that you would like to continue [y/N]y

Enter password for [elastic]:
Reenter password for [elastic]:
Enter password for [kibana]:
Reenter password for [kibana]:
Enter password for [logstash_system]:
Reenter password for [logstash_system]:
Changed password for user [kibana]
Changed password for user [logstash_system]
Changed password for user [elastic]
[ ]$
```

### SSL/TLS

X-Pack Security는 노드간, 그리고 클러스터와 클라이언트 간의 통신을 암호화 하는 SSL/TLS 기능을 가지고 있습니다. 특히 elasticsearch 6.0 부터는 X-Pack Security를 사용하기 위해서는 SSL/TLS 설정을 반드시 활성화 해야 오류나 경고 메시지가 나타나지 않습니다.
SSL/TLS 설정은 다음 포스트에서 클러스터의 모든 노드들의 생성이 끝난 뒤에 설정 하도록 하겠습니다.

## 한글 형태소 분석기 설치

아래 블로그 포스트 또는 각 커뮤니티의 문서를 참고해서 설치하도록 합니다.
아리랑 : https://www.elastic.co/kr/blog/arirang-analyzer-with-elasticsearch

정상적으로 설치가 되면 elasticsearch를 재시작 후 로그에 다음과 같이 플러그인이 나타납니다.
```
[2017-12-29T07:18:31,240][INFO ][o.e.p.PluginsService     ] [kr-demo-master] loaded plugin [analysis-arirang]
```

여기까지 Elasticsearch의 공통적인 설치 및 설정들이 완료되었습니다.
다음 포스트에서는 지금까지 만든 설정들을 복사해서 1개의 마스터 노드와 3개의 데이터 노드 시스템을 생성하고, 각 노드별로 구분되어야 할 환경들을 설정 해 보도록 하겠습니다.

> [1. 서버 생성 및 Elasticsearch RPM 설치](/2018/01/2018-01-build-es-cluster-1)
> **2. 메모리, 네트워크 설정 및 플러그인 설치**
> [3. 클러스터 구성 및 마스터, 데이터 노드 설정](/2018/01/2018-01-build-es-cluster-3)
> [4. Kibana 설치 및 X-Pack Monitoring 확인](/2018/01/2018-01-build-es-cluster-4)
> [5. NFS 구성 및 elasticsearch 추가 설정](/2018/01/2018-01-build-es-cluster-5)
> [6. X-Pack Security를 이용한 SSL 및 TLS 설정](/2018/01/2018-01-build-es-cluster-6)
> [7. X-Pack License 적용 및 사용자 생성](/2018/01/2018-01-build-es-cluster-7)
> [8. Logstash 설치 및 Elasticsearch 기본 템플릿 설정](/2018/01/2018-01-build-es-cluster-8)