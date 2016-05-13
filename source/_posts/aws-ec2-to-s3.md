---
layout: post
title: EC2 에서 S3 로 파일 전송
subtitle: "AWS – EC2 에서 S3 로 파일 전송하는 방법"
date: 2014-02-06
header-img: "aws_icons.png"
tags:
  - AWS
  - ICT
categories:
  - AWS
---

지난 몇번에 걸쳐 AWS 설치와 EC2 셋팅 방법을 알아 보았습니다. 지난 내용들은 아래 포스트를 참고 해 주세요.

1. AWS 설치 – 인스턴스 생성 및 실행
1. AWS 설치 – TimeZone 및 Java 설치
1. AWS 설치 – Apache, MySQL, PHP, phpMyAdmin

오늘은 AWS 의 컴퓨팅 서비스인 EC2 에서 데이터 저장 서비스인 S3 로 파일을 전송하는 방법을 알아보도록 하겠습니다.

S3 는 매우 저렴한 비용으로 (1GB당 한달에 0.12$ 정도) 사용 가능한 클라우드 스토리지 서비스입니다. EC2 가 CPU, RAM, HDD 등을 선택해서 설치,임대하는 서비스이고, S3 는 무제한의 용량을 가진 외장 하드를 임대하는 서비스라고 생각하시면 됩니다. 하지만 아쉽게도 EC3 에 HDD 처럼 S3 를 붙여서 사용할 수는 없습니다. EC2 에서 사용하는 스토리지는 [EBS(Elastic Block Store)](https://aws.amazon.com/ko/ebs/?tag=vig-20) 라는 녀석을 사용합니다.

EBS 와 S3 의 차이점은 EBS는 처음에 몇GB 를 설정하고 EC2에 연결 한 다음 EC2 콘솔에서 HDD 처럼 사용을 합니다. EBS는 처음에 설정한 용량 대로 비용 청구가 됩니다. S3 는 무제한 용량의 스토리지가 있는데 거기에 때려 넣은 데이터 용량 만큼 알아서 과금이 되는 방식입니다. S3는 EC2 에 붙일 수 없으며, 드랍박스 같은 웹하드 형식으로 서비스가 됩니다. 자세한 내용은 AWS 홈페이지를 방문하셔서 살펴 보시거나 정보가 많이 나와 있으므로 구글링을 해 보시기 바라며, 이번 포스트에서는 EC2 에서 S3 로 데이터를 전송하는 방법을 위주로 설명을 드리겠습니다.

먼저 AWS 콘솔에서 S3 콘솔을 선택합니다. 다음과 같은 S3 처음 화면이 나타납니다.
![](aws_1.png)

S3 의 데이터 저장소는 “버킷” 이라는 단위로 관리가 됩니다. 이제 `Create Bucket` 버튼을 눌러 버킷을 생성 해 보도록 하겠습니다.
![](aws_2.png)

버킷을 생성하고 나면 보통의 웹 하드 같은 형식의 콘솔 메뉴가 나타납니다. 여기서 디렉토리를 만들고 로컬에 있는 파일을 업로드 하는 등의 동작을 수행 할 수 있습니다.
![](aws_3.png)

이제 EC2 에서 S3로 직접 파일을 전송하기 위한 준비를 하겠습니다. 사실 이 포스트는 이 프로그램을 소개하기 위해 작성했습니다. 바로 **Amazon S3 Tool** 이라는 녀석입니다. 어느 훌륭한 외쿡 개발자님께서 저같은 사람을 위해 만들어 (무려 공짜로) 주셨습니다. 감사감사~

[http://s3tools.org/download](http://s3tools.org/download) 에 가서 프로그램을 다운받습니다. 다운로드를 클릭하는 소스포지로 연결이 되네요. 페이지 뷰도 올려드릴 겸 여긴 직접 한번 찾아가 보시기 바랍니다. :) 현재 보이는 최종 버전은 1.5.0-beta-1 버전이네요.

다운받은 s3cmd-1.5.0-beta1.tar.gz 파일을 EC2 인스턴스에 업로드 하고 EC2 콘솔로 들어갑니다. tar xvfz 명령으로 압축을 풀어줍니다.
![](aws_4.png)

.py 파일들이 보이네요. 파이썬으로 만들었나봅니다. 파이썬 공부해야 하는데 ㅠ_ㅠ. 이제 저 명령어를 어디서나 사용할 수 있도록 PATH 가 잡힌 디렉토리로 옮기도록 하겠습니다. /usr/local/s3cmd 경로로 옮기고 /usr/bin 에 링크 파일을 만들겠습니다.

```
[ec2-user@ ~]$ sudo mv s3cmd-1.5.0-beta1 /usr/local/s3cmd
[ec2-user@ ~]$ sudo ln -s /usr/local/s3cmd/s3cmd /usr/bin/s3cmd
[ec2-user@ ~]$ ls -l /usr/bin/s3cmd
lrwxrwxrwx 1 root root 22 2014-02-06 16:18 /usr/bin/s3cmd -> /usr/local/s3cmd/s3cmd
```

이제 처음 설정을 해 주어야 하는데 그 전에 AWS 의 Access Key 와 Secret Access Key 를 기록 해 두어야 합니다. Access Key 를 확인 하러 갑니다. AWS 콘솔 상단 우측 메뉴에서 Security Credentials 를 클릭합니다.
![](aws_5.png)

여기서 Access Keys 항목 왼쪽의 `[+]` 를 클릭 해 펼친 뒤 `Create New Access Key` 버튼을 눌러 인증키를 생성합니다. ***<span style="color:red">Secret Access Key 는 한번 확인 후 두번 확인이 불가능하니 반드시 다운로드 받아 잘 보관하시고 본인만이 아는 곳에 적어 두시기 바랍니다.!!!</span>***
![](aws_6.png)

자, 이제 다시 EC2 콘솔로 돌아와서, s3cmd –configure 명령어를 실행시킵니다. Access Key 와 Secret Key 를 물어보는데, 아까 적어둔 코드 들을 넣어줍니다.
![](aws_7.png)

다음에 3자가 접근시 확인을 위한 암호를 물어봅니다. 다음 Path 는 그냥 엔터 쳐서 넘어갑니다.
![](aws_8.png)

HTTPS 프로토콜을 사용하는지 물어봅니다. Yes 를 입력합니다.
![](aws_9.png)

접속 테스트를 할지 물어봅니다.엔터를 치면 접속 테스트를 합니다. 잘 접속이 되네요. :) 마지막으로 Save Setting 에서 Y 를 눌러 저장을 하면 설정이 끝납니다.
![](aws_10.png)

이제 s3cmd 명령을 통해 버킷을 확인 해 보겠습니다. s3cmd 뒤에 파라메터로 명령어를 주면 됩니다. s3cmd ls 명령어를 치면 버킷 목록을 보여줍니다.
![](aws_11.png)

아까 S3 콘솔에서 생성한 kimjmin-bucket-1 버킷이 보입니다. 이제 mb 명령어로 새로운 버킷을 생성 해 보겠습니다. s3cmd mb s3://kimjmin-bucket-2 라고 명령을 내린 뒤 다시 ls 명령어로 확인합니다.
![](aws_12.png)

버킷이 생성되었네요. S3 콘솔에서도 확인 할 수 있습니다.
![](aws_13.png)

자, 이제 하이라이트, 파일 전송을 해 보도록 하겠습니다. 현재 디렉토리에 있는 s3cmd-1.5.0-beta1.tar.tar.gz 파일을 방금 생성한 버킷으로 전송 해 보겠습니다. 전송을 위해서는 put 명령을 사용합니다.
```
s3cmd put s3cmd-1.5.0-beta1.tar.tar.gz s3://kimjmin-bucket-2/
```
위 명령어로 s3cmd-1.5.0-beta1.tar.tar.gz 파일을 kimjmin-bucket-2 버킷으로 전송합니다. 전송 후에는 ls s3://버킷명/경로 명령어로 파일 내용을 확인할 수 있습니다.
![](aws_14.png)

모두 전송이 잘 되었습니다. :)

다른 s3cmd 명령어들은 http://s3tools.org/s3cmd 페이지에서 확인 가능합니다.