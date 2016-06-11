---
title: 나만의 Beats 만들기
tags:
  - Elasticsearch
  - Beats
subtitle: 나만의 Elasticsearch Beats 를 만들어 보자
header-img: metricbeats.png
categories:
  - Elasticsearch
date: 2016-06-02 11:24:07
---

Beats는 Elasticsearch로 데이터를 전송하는 **경량 데이터 전송 프로그램 (Lightweight Data Shipper)** 입니다. 현재 Elastic 에서는 자체적으로 Packetbeat, Filebeat, Topbeat 등을 배포하고 있으며 커뮤니티 개발자들이 직접 만든 다양한 [Community Beats](https://www.elastic.co/guide/en/beats/libbeat/current/community-beats.html) 도 사용할 수 있습니다. 

Beats는 개발자들이 자신이 필요로 하는 Beat을 만들 수 있도록 libbeat 이라고 하는 라이브러리와 Beat Generator 라는 패키지를 제공하고 있습니다. 이번 포스트에서는 Beat Generator를 사용해서 직접 나만의 Beat 를 만드는 법을 알아보도록 하겠습니다. 오늘 제가 만들 Beat은 유닉스 계열에서 `ls` 명령을 실행하면 나오는 파일, 디렉토리 리스트 정보를 elasticsearch 에 색인하는 lsbeat 입니다. 참고로 저는 MacOS 에서 했기 때문에 Windows나 다른 Unix 계열 OS 에서는 추가적으로 필요한 정보들을 확인 하시고 그에 맞게 설치 및 진행을 하시기 바랍니다.

Beats는 Go 언어로 개발이 되어 있습니다. Beats 를 시작하기 위해서는 Go 언어 컴파일러 및 glide가 설치되어 있어야 합니다. glide 는 Golang 에서 사용하는 Package Management 도구 입니다. NodeJS의 npm 같은 도구이라고 보시면 될 것 같습니다. 앞의 두 패키지들을 설치 해 줍니다.
```sh
$ brew install go
$ brew install glide
```

Beat Generator는 cookiecutter를 이용해서 내려받을 수 있습니다.참고로 저는 파이썬을 써 보질 않아서 pip, cookiecutter 가 설치되어 있지 않았습니다. 그래서 cookiecutter도 설치했습니다.
```sh
$ sudo easy_install pip
$ sudo pip install cookiecutter
$ sudo pip install virtualenv
```

여기서 잠깐 우리가 개발 할 lsbeat 에서 사용할 파일과 디렉토리를 목록을 조회하는 프로그램을 먼저 Go 언어로 짜 보겠습니다. 프로그램은 다음과 같습니다.
```go
package main

import (
	"fmt"
	"io/ioutil"
	"os"
)

func main() {
	//파라메터를 안 받았다면 현재 경로 "." 이하 파일, 디렉토리들 출력
	if len(os.Args) == 1 {
		listDir(".")
	} else {
		listDir(os.Args[1])
	}
}

func listDir(dirFile string) {
	files, _ := ioutil.ReadDir(dirFile)
	for _, f := range files {
		t := f.ModTime()
		fmt.Println(f.Name(), dirFile+"/"+f.Name(), f.IsDir(), t, f.Size()) //파일(디렉토리)명, 전체path, 디렉토리여부, 생성시간, 파일크기 출력
    
    //디렉토리인 경우 하위 파일들 재귀 호출.
		if f.IsDir() {
			listDir(dirFile + "/" + f.Name())
		}
	}
}
```
저는 위 파일을 `lsscan.go` 라는 이름으로 저장했습니다. `go run lsscan` 을 실행하면 현재 위치 이하의 디렉토리와 파일들을 출력합니다.
```sh
$ go run lsscan.go
lsscan.go ./lsscan.go false 2016-06-02 15:11:47 +0200 CEST 575
```

출력이 잘 됩니다. 이제 lsbeats 를 만들기 위한 사전 작업들을 실행 해 보겠습니다. cookiecutter 명령어를 이용해서 [Beat Generator Project](https://github.com/elastic/beat-generator) 스켈레톤을 설치합니다.
```sh
$ cookiecutter https://github.com/elastic/beat-generator.git
```

설치 중에 몇가지 질문이 나오는데 대답 해 줍니다. 프로젝트 이름은 우리가 만들 `lsbeat`로 합니다.
```sh
project_name [Examplebeat]: lsbeat
github_name [your-github-name]: kimjmin
beat [lsbeat]:
beat_path [github.com/kimjmin]:
full_name [Firstname Lastname]: Jongmin Kim
```

위 과정을 실행하고 나면 `lsbeat` 디렉토리가 생성됩니다. `lsbeat`에 들어가서 설치된 파일들을 확인 해 보면 다음과 같습니다.
```sh
$ cd lsbeat
$ tree
.
├── CONTRIBUTING.md
├── LICENSE
├── Makefile
├── README.md
├── beater
│   └── lsbeat.go
├── config
│   ├── config.go
│   └── config_test.go
├── dev-tools
│   └── packer
│       ├── Makefile
│       ├── beats
│       │   └── lsbeat.yml
│       └── version.yml
├── docs
│   └── index.asciidoc
├── etc
│   ├── beat.yml
│   └── fields.yml
├── glide.yaml
├── lsbeat.template.json
├── main.go
├── main_test.go
└── tests
    └── system
        ├── config
        │   └── lsbeat.yml.j2
        ├── lsbeat.py
        ├── requirements.txt
        └── test_base.py
```

이 프로젝트는 [libbeat](https://github.com/elastic/beats/tree/master/libbeat) 파일들을 포함하고 있으며 수집한 데이터를 elasticsearch로 전송하기 위한 모든 기본 기능들을 담고 있습니다. 이제 프로젝트에서 연관 라이브러리들을 내려받아 설치하고 컴파일 해서 기본 설정 파일들을 생성해야 합니다. 다음 명령들을 차례대로 실행합니다.
```sh
$ make init
$ make commit
```
위 명령을 실행하고 나면 `lsbeat.yml`, `lsbeat.template.json` 파일들이 생성되고 `vendor` 디렉토리 아래에 `elstic/beats` 라이브러리 들이 내려받아집니다. `lsbeat.yml` 파일을 살펴보면 beats 실행에 필요한 기본적인 환경변수 값들이 미리 설정이 되어 있습니다.
```yml
lsbeat:
  # Defines how often an event is sent to the output
  period: 1s
```
현재 `lsbeat.yml`에는 beats 의 실행 주기인 period 값이 1초로 설정되어 있습니다. 이는 beats가 매 1초마다 재실행되면서 elasticsearch 로 데이터를 전송하도록 하는 것을 뜻합니다. 지금 1초로 되어 있는 기준을 10초로 늘리고, 여기에 lsbeat 가 실행될 때 파일과 디렉토리 목록을 가져올 기준 경로로 지정할 `path` 매개변수를 추가 해 보도록 하겠습니다. 매개변수 추가는 `lsbeat.yml` 이 아니라 `etc` 디렉토리 아래에 있는 `beat.yml` 에 추가 한 후 `make update` 명령어로 다시 컴파일을 합니다. `etc/beat.yml`의 내용을 다음과 같이 수정합니다.
```yml
lsbeat:
  # Defines how often an event is sent to the output
  period: 10s
  path: "."
```

다시 lsbeat 홈 경로로 돌아와 `make update` 를 실행하고 `lsbeat.yml` 파일을 다시 열어보면 앞의 `etc/beat.yml` 에서 설정 해 준 내용들이 반영된 것을 확인할 수 있습니다.
```sh
$ make update
$ head lsbeat.yml
################### Lsbeat Configuration Example #########################

############################# Lsbeat ######################################

lsbeat:
  # Defines how often an event is sent to the output
  period: 10s
  path: "."
###############################################################################
```
추가된 path 값을 사용하려면 `config` 디렉토리 아래 있는 `config.go` 파일에 내용을 추가해야합니다. `config.go` 파일에 다음과 같이 path 환경변수 값을 가져오는 로직을 추가합니다.
```go
type LsbeatConfig struct {
	Period string `config:"period"`
	Path   string `config:"path"` //path 환경변수 값 추가.
}
```
참고로 Go 언어에서는 다른 패키지에서 사용할 수 있도록 public으로 선언된 변수는 변수명이 대문자로 시작해야 합니다. 변수명을 반드시 Path 로 하되 path 로 하지 않도록 합니다.

이제 컴파일 할 때 추가된 libbeat 패키지의 `/vendor/elastic/beats/libbeat/beat/beat.go` 파일 안에 있는 Beater의 interface 를 확인 해 보면 다음과 같은 interface 들을 포함하고 있습니다 .
```go
type Beater interface {
	Config(*Beat) error  // Read and validate configuration.
	Setup(*Beat) error   // Initialize the Beat.
	Run(*Beat) error     // The main event loop. This method should block until signalled to stop by an invocation of the Stop() method.
	Cleanup(*Beat) error // Cleanup is invoked to perform any final clean-up prior to exiting.
	Stop()               // Stop is invoked to signal that the Run method should finish its execution. It will be invoked at most once.
}
```
위의 `Config`, `Setup`, `Run`, `Cleanup`, `Stop` 5개 인터페이스들이 실제 모든 종류의 Beat이 실행되는 루틴입니다. 실제 우리가 만들 lsbeat 가 실행하는 로직은 `beater` 디렉토리 아래에 있는 `lsbeat.go` 파일에 구현을 하게 됩니다. `beater/lsbeat.go` 파일을 열어 확인 해 보면 `Config`, `Setup`, `Run`, `Cleanup`, `Stop` 함수들이 실제로 구현되어 있습니다. `beater/lsbeat.go` 내용을 하나씩 살펴보겠습니다.

먼저 상단에 `Lsbeat` 이름의 구조체가 선언되어 있습니다.  `lsbeat` 전체에서 사용할 전역 변수들을 담고 있습니다. 우리가 사용 할 파라메터 값들도 여기에 추가하게 됩니다. lsbeat 가 실행될 root 디렉토리값인 `path` 와 가장 마지막에 검색 한 시간을 가져올 `lastIndexTime` 값을 각각 string 그리고 time.Time 형식으로 추가 선언 해 주도록 합니다.
```go
type Lsbeat struct {
	beatConfig   *config.Config
	done         chan struct{}
	period       time.Duration
	client       publisher.Client
	path         string    //root 디렉토리
	lasIndexTime time.Time //가장 마지막에 검색한 시간
}
```

`Setup` 함수에서는 config 에서 설정된 파라메터 값들을 Lsbeat 구조체 값들에 대입해줍니다. 현재 `Period` 라는 파라메터값이 디폴트로 생성되어 있는데 `Period` 값이 없다면 1s 로 셋팅 해 주도록 하고 있습니다. 여기에 우리가 추가한 path 파라메터의 기본값을 할당하는 코드를 추가합니다.
```go
func (bt *Lsbeat) Setup(b *beat.Beat) error {

	// Setting default period if not set
	if bt.beatConfig.Lsbeat.Period == "" {
		bt.beatConfig.Lsbeat.Period = "1s"
	}
  
  // path 값을 lsbeat.yml 에 설정된 값으로 설정.
	if bt.beatConfig.Lsbeat.Path == "" {
		bt.beatConfig.Lsbeat.Path = "."
	}
	bt.path = bt.beatConfig.Lsbeat.Path
  
	bt.client = b.Publisher.Connect()

	var err error
	bt.period, err = time.ParseDuration(bt.beatConfig.Lsbeat.Period)
	if err != nil {
		return err
	}

	return nil
}
```

이제 가장 중요한 `Run` 함수를 살펴봅니다.
```go
func (bt *Lsbeat) Run(b *beat.Beat) error {
	logp.Info("lsbeat is running! Hit CTRL-C to stop it.")

	ticker := time.NewTicker(bt.period)
	counter := 1
	for {
		select {
		case <-bt.done:
			return nil
		case <-ticker.C:
		}

		event := common.MapStr{
			"@timestamp": common.Time(time.Now()),
			"type":       b.Name,
			"counter":    counter,
		}
		bt.client.PublishEvent(event)
		logp.Info("Event sent")
		counter++
	}
}
```
여기 `Run` 함수에 있는 `event := common.MapStr` 부분이 elasticsearch 에 전송될 json 도큐먼트를 MapStr 형식으로 저장하는 부분이며, `bt.client.PublishEvent(event)` 라인이 도큐먼트를 elasticsearch 로 실제 전송하는 부분입니다. 현재 `@timestamp`, `type`, `counter` 필드가 디폴트로 구현되어 있는 것을 확인할 수 있습니다. 이제 여기에 우리가 수집한 파일(디렉토리)명, 전체 path, 디렉토리여부, 생성시간, 파일크기를 추가 할 것입니다.

먼저 `lsbeat.go` 파일 맨 아래에 우리가 앞서 만들었던 파일과 디렉토리 list 를 가져오는 listDir() 함수를 추가 합니다. 상단의 `import` 에도 `"io/ioutil"` 패키지를 추가 해 줘야 합니다.
```go
import (
	"fmt"
	"io/ioutil" //listDir 에서 사용하기 위해 추가
	"time"
  
... 중략 ...

// 파일, 디렉토리 조회해서 출력하는 함수
func listDir(dirFile string) {
	files, _ := ioutil.ReadDir(dirFile)
	for _, f := range files {
		t := f.ModTime()
    fmt.Println(f.Name(), dirFile+"/"+f.Name(), f.IsDir(), t, f.Size())
    
		if f.IsDir() {
			listDir(dirFile+"/"+f.Name(), bt, b, counter)
		}
	}
}
```

이제 이 파일에서 `fmt.Println`로 콘솔 출력을 하는 대신 `event` MapStr 데이터를 만들어서 전송 해 주는 로직을 추가합니다. 이를 위해서 listDir 함수가 입력 받는 파라메터로  `bt *Lsbeat`, `b *beat.Beat`, `counter int` 를 추가합니다. 그리고 event 객체에에 다음 5개의 필드를 추가합니다.
- `modTime`: f.ModTime() - 마지막 파일 수정 시간 
- `filename`: f.Name() - 파일명
- `fullname`: dirFile + "/" + f.Name() - 전체 path 경로
- `isDirectory`: f.IsDir() - 디렉토리 여부
- `fileSize`: f.Size() - 파일 크기

또한 계속해서 같은 파일들을 중복해서 색인하지 않도록 첫번째 실행에만 파일들을 색인 하고 이후에는 이전 색인 프로세스 이후에 생성/변경된 파일들만 색인하도록 시간을 체크하는 로직도 추가합니다. 최근 데이터 송신 시간은 `Lsbeat` 구조체의 `lasIndexTime` 변수에 저장 할 것입니다.
```go
// bt *Lsbeat, b *beat.Beat, counter int 추가
func listDir(dirFile string, bt *Lsbeat, b *beat.Beat, counter int) {
	files, _ := ioutil.ReadDir(dirFile)
	for _, f := range files {
		t := f.ModTime()
		//fmt.Println(f.Name(), dirFile+"/"+f.Name(), f.IsDir(), t, f.Size())

		event := common.MapStr{
			"@timestamp":  common.Time(time.Now()),
			"type":        b.Name,
			"counter":     counter,
			"modTime":     t,
			"filename":    f.Name(),
			"fullname":    dirFile + "/" + f.Name(),
			"isDirectory": f.IsDir(),
			"fileSize":    f.Size(),
		}
		//첫번째 실행인 경우 전체 파일 목록 색인.
		if counter == 1 {
			bt.client.PublishEvent(event) //elasticsearch 로 데이터 전송.
		} else {
			//2번째 이후 인 경우 지난번 실행 이후에 추가된 파일만 색인
			if t.After(bt.lasIndexTime) {
				bt.client.PublishEvent(event) //elasticsearch 로 데이터 전송.
			}
		}

		if f.IsDir() {
			listDir(dirFile+"/"+f.Name(), bt, b, counter) // bt *Lsbeat, b *beat.Beat, counter int 추가
		}
	}
}
```

이제 `Run` 함수에서 elasticsearch 로 데이터를 전송하던 부분을 주석 처리하고 우리가 만든 `listDir` 함수를 호출하는 로직과 송신 후 `lasIndexTime`에 타임스탬프를 기록하는 로직을 추가합니다.
```go
func (bt *Lsbeat) Run(b *beat.Beat) error {
	logp.Info("lsbeat is running! Hit CTRL-C to stop it.")

	ticker := time.NewTicker(bt.period)
	counter := 1
	for {
		listDir(bt.path, bt, b, counter) // lsbeat 데이터 송신
		bt.lasIndexTime = time.Now()     // 송신 후 타임스탬프 기록.

		select {
		case <-bt.done:
			return nil
		case <-ticker.C:
		}

		// 기존 elasticsearch 전송 로직 삭제
		// event := common.MapStr{
		// 	"@timestamp": common.Time(time.Now()),
		// 	"type":       b.Name,
		// 	"counter":    counter,
		// }
		// bt.client.PublishEvent(event)

		logp.Info("Event sent")
		counter++
	}
}
```

프로그램이 거의 완성되었습니다. 새로 추가한 필드들의 매핑 정보를 설정해야 합니다. 매핑 정보는 `etc` 디렉토리 아래의 `fields.yml`에서 추가합니다.
```yml
lsbeat:
  fields:
    - name: counter
      type: integer
      required: true
      description: >
        PLEASE UPDATE DOCUMENTATION
    #lsbeat 에서 추가된 필드들.
    - name: modTime
      type: date
    - name: filename
      type: text
    - name: fullname
    - name: isDirectory
      type: boolean
    - name: fileSize
      type: long
```

이제 새로 업데이트 한 설정들을 반영하기 위해 다시 `make update` 를 실행합니다.
```sh
$ make update
```

`make update`를 실행하고 나면 `lsbeat.template.json` 파일에 앞에 수정한 매핑 설정이 반영됩니다. `filename` 필드는 nGram 방식으로 색인을 진행 할 것이기 때문에 `lsbeat.template.json` 파일을 열고 "settings" 부분에 nGram analyzer 를 추가하고 filename 필드에도 적용 해 줍니다.
```js
//filename 에 "analyzer": "ls_ngram_analyzer" 추가
"filename": {
  "norms": false,
  "type": "text",
  "analyzer": "ls_ngram_analyzer"
  
//settings 에 ls_ngram_analyzer 추가.
"settings": {
  "index.refresh_interval": "5s",
  "analysis": {
    "analyzer": {
      "ls_ngram_analyzer": {
        "tokenizer": "ls_ngram_tokenizer"
      }
    },
    "tokenizer": {
      "ls_ngram_tokenizer": {
        "type": "ngram",
        "min_gram": "2",
        "token_chars": [
          "letter",
          "digit"
        ]
      }
    }
  }
},
```

이제 이 템플릿을 elasticsearch 에 적용하고 `make` 명령어로 `lsbeat` 를 컴파일 합니다.
```sh
$ curl -XPUT localhost:9200/_template/lsbeat -d@lsbeat.template.json
{"acknowledged":true}
$ make
go build
```

컴파일 후 `lsbeat` 실행 바이너리 파일이 생성됩니다. 이제 모든 준비가 완료되었습니다. 윈도우 계열이라면 `lsbeat.exe` 파일이 생성 될 것입니다. 이제 생성된 `lsbeat` 파일을 실해 해 볼 것입니다. 실행 전에 `lsbeat.yml`에서 파일을 색인 할 root 경로를 지정 해 줘야 합니다. 저는 제 컴퓨터의 $GOPATH 인 `/Users/kimjmin/golang` 을 설정하겠습니다.
```yml
lsbeat:
  # Defines how often an event is sent to the output
  period: 10s
  path: "/Users/kimjmin/golang"
```

이제 `lsbeat`를 실행합니다. 당연히 elasticsearch 는 사전에 실행중이어야 합니다.
```sh
$ ./lsbeat
```

파일이 색인되고 있는지 확인 해 보겠습니다. 먼저 `_cat` API를 이용해서 인덱스가 제대로 생성되었는지 확인합니다.
![](cat.png)

**lsbeat-2016.06.03** 인덱스가 생성되었고 데이터가 정상적으로 들어오고 있습니다. 이제 query 를 사용해서 lsbeat 를 검색 해 보겠습니다. nGram 으로 제대로 색인이 되었는지 `lsbe` 까지만 검색 해 보겠습니다.
![](query.png)

검색이 정상적으로 실행됩니다.

이처럼 libbeat 프레임워크를 사용해서 내가 필요로 하는 Beat를 만들 수 있습니다.

