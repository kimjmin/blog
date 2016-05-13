---
layout: post
title: AWS - 생성 및 실행
subtitle: "AWS 설치 – EC2 인스턴스 생성 및 실행"
date: 2014-01-16
header-img: "aws_icons.png"
tags:
  - AWS
categories:
  - AWS
---
Amazon Web Service 에 클라우드 서버 구축 중입니다. 설치 내용 기록 겸 포스팅으로 남깁니다.

- AWS 콘솔 화면으로 이동 > EC2 선택.
- 지역을 Tokyo 로 변경 -> Lunch Instance 버튼 클릭.

![](aws_1.png)

운영체제 선택
![](aws_2.png)

인스턴스 타입 선택 (처음에는 무료 티어만 나와있으므로 All instance type 클릭해서 전체 티어 확인.) > `Review and Launch` 버튼 클릭.
![](aws_3.png)

(왜인지 모르겠으나) 처음에 는 EBS 스토리지만 잡혀 있기 때문에 시스템 스토리지를 추가해줘야 합니다. 하단에 Storage 항목을 열고 Edit Storage 를 클릭해서 스토리지 설정 화면으로 이동합니다.
![](aws_4.png)

`Add New Volume` 버튼을 클릭한 후 Instance Storage 를 추가합니다. 이렇게 하면 리눅스 /dev/ 디렉토리 아래 선택한 이름의 디바이스가 잡히게 됩니다. 저는 “sdb” 로 했습니다.
![](aws_5.png)

이제 다시 Review and Launch 화면으로 돌아가서 `Launch` 버튼을 누르면 키페어를 물어봅니다. 이미 생성한 키페어를 사용할 수도 있고 새로 생성도 가능합니다.
![](aws_6.png)

키페어를 다운받고 나면 인스턴스가 생성이 됩니다. 인스턴스에 따라 생성 되는데는 약 1~5분 정도 걸립니다.

인스턴스가 생성되고 나면 외부IP 를 연결 해 줍니다. 1개를 초과하는 IP를 할당하려면 추가 비용이 소모됩니다. Elastic IPs 메뉴로 가셔서 `Allocate New Address` 버튼을 클릭해서 새로운 IP를 생성한 후 `Associate Address` 버튼을 눌러 생성한 인스턴스에 연결 해 줍니다.
![](aws_7.png)

인스턴스를 생성하면 기본적으로 외부에서 접근 가능한 IP와 도메인 주소는 당연히 자동으로 생성이 됩니다. 하지만 이 때 생성된 IP는 인스턴스를 재시작할 때 마다 바뀌게 됨으로 이렇게 Elastic IP 를 연결 해 주는 것이 좋습니다.

이제 생성한 인스턴스에 접근 해 보겠습니다. 접근 방법은 Instances 화면에서 인스턴스를 선택한 후 상단의 `Connect` 버튼을 누르면 자세한 설명이 나타납니다.
![](aws_8.png)

먼저 인증서를 400 모드로 변경을 해 줍니다. (이렇게 하지 않으면 키체인 사용이 불가능합니다.) 그리고 나서 ssh 접속으로 인증서를 사용해서 접속을 합니다. 기본적으로 생성되는 user 의 이름은 “ec2-user” 입니다. sudo 로 루트 명령들을 실행시킬 수 있는 권한이 있습니다.

이제 서버에 접속해서 용량을 확인 해 보면 EBS 용량인 8GB 정도 밖에 잡혀 있지 않습니다. 아까 인스턴스를 생성할 때 만들어 준 볼륨을 마운트 해 줘야 합니다.
![](aws_9.png)
```
sudo mkdir /data
sudo mount /dev/xvdb /data
```
이렇게 디렉토리를 생성하고 해당 디렉토리를 마운트해 줍니다.
```
df -h
```
명령어로 디스크 용량을 확인 해 보면 마운트 한 디렉토리에 시스템 용량이 잡힌 것을 확인할 수 있습니다.

![](aws_10.png)

<span style="color:red">**P.S. 중요정보!!
instance store 는 ephemeral 입니다. 인스턴스를 stop 하고 재시작 하면 데이터가 전부 날아갑니다. AWS 는 EBS Volume 이 아닌 저장소는 전부 ephemeral 으로 설정이 됩니다. 주말 내내 입력한 제 데이터 2억건도 지금 다 날아갔네요 ㅠ_ㅠ**</span>