---
title: Swiftype 사이트 서치
tags:
  - Elasticsearch
  - Site Search
  - Swiftype
categories:
  - Elasticsearch
subtitle: Swiftype 기능을 이용하여 블로그에 검색 기능을 달아봅시다.
date: 2018-04-04
header-img: "swiftype-head.png"
---

## Swiftype

작년 2017년 11월 경 Elastic은 사이트 서치, 엔터프라이즈 서치 기업인 [Swiftype](https://swiftype.com)을 인수했습니다. (관련 블로그:https://www.elastic.co/blog/swiftype-joins-forces-with-elastic)
저도 처음 소식을 들었을 때는 우리도 스타트업인데 회사가 또 뭘 이렇게 인수하나 싶었는데, 실제로 Swiftype 기능을 한번 보고 나서는 이거 정말 괜찮은 물건이구나 싶었습니다.

swiftype 기능 소개는 아래 영상에서 확인할 수 있습니다.
<iframe width="560" height="315" src="https://www.youtube.com/embed/fmLZzpds0hI" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

이것도 좀 다뤄봐야 하는데 계속 다른 작업에 밀려서 못 하고 있다가 오늘 삘 받아서 한번 사용을 해 봤습니다.
눈썰미 좋은 분들은 제 블로그 첫 페이지에 검색 기능이 생긴 것을 눈치 치셨을텐데요, 이번 포스트에서는 개인 블로그에 Swiftype 적용 한 내용을 다루도록 하겠습니다.
![](001.png)

## 서비스 가입 및 데이터 수집

처음에 서비스를 가입하면 14일간 트라이얼 버전의 사용이 가능합니다. 웹 페이지 내용을 가져오는 방법은

- 크롤러를 사용해서 페이지 내용을 긁어오기
- 웹페이지에 swiftype API 코드 삽입

두가지 방법이 있습니다. 저는 크롤러를 사용해서 내용을 가져오도록 설정했습니다. 

![](002.png)

크롤러 사용을 선택한 후 웹사이트 주소만 적어주면 바로 크롤링을 시작합니다.

![](003.png) ![](004.png)

사이트 하나를 Swiftype 에서는 엔진 (engine) 이라는 단위로 분류하며, 엔진별로 과금을 하게 됩니다. 스탠다드가 월 79달러 입니다.

![](005.png)

## 검색 데이터 관리

크롤러가 데이터를 다 수집하고 나면 다음과 같이 Search Preview 메뉴에서 실제로 수집 된 내용들을 검색 해 볼 수 있습니다.

![](006.png)

각 필드별로 가중치를 조절해서 검색 순위를 조절할 수도 있습니다. 오른쪽에 검색 결과가 달라지는 모습이 실시간으로 나타납니다.

![](007.png)

Synonym 메뉴에서 동의어 지정도 가능합니다. 아래는 `meetup`과 `밋업`을 동의어로 지정하고 검색 해 본 결과입니다.

![](008.png), ![](009.png)

## 웹페이지에 검색 기능 추가

interface -> install search 메뉴로 들어가면 웹페이지에 추가할 수 있는 javascript 코드가 나타납니다.

![](010.png) ![](011.png)

여기서 configurations 메뉴에 들어가면 웹 페이지에 나타나는 모양과 검색 창 입력 형식을 지정할 수 있습니다. 기존에 검색 기능이 없는 경우에는 `No, my site needs an input field` 를 선택하면 검색창을 추가할 수 있는 input 폼 코드, 또는 swiftype search 탭을 사용하도록 선택이 가능합니다.

![](012.png) ![](013.png)

이제 이 코드들을 웹페이지에 넣으면 웹 페이지에서 검색 기능을 사용할 수 있습니다.

![](012.png)

지금까지의 과정을 비디오로 녹화했습니다. 아래 영상을 보시면 Swiftype 을 이용해서 이 웹 페이지에 검색 기능을 추가하는 과정을 처음부터 설명합니다.

<iframe width="560" height="315" src="https://www.youtube.com/embed/_fJ-OCdXGR8" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>


> 다만, 저도 14일간 트라이얼 버전을 사용하고 있기 때문에 4월 17일 이후에는 검색 기능이 동작을 안 할 수도 있습니다. 🤓