---
layout: post
title: AWS 설치 - TimeZone, Java
subtitle: "AWS 설치 – TimeZone 및 Java 설치"
date: 2014-01-17
header-img: "aws_icons.png"
tags:
  - AWS
  - Java
categories:
  - AWS
---

AWS 설치를 계속 진행 해 보도록 하겠습니다. [AWS - 생성 및 실행](/2014/01/aws-ec2-install) 은 이전 포스트를 참고 해 주세요.

AWS 를 설치 하고 나면 몇가지 작업을 해 주어야 합니다. 그 중 하나가 시간을 설정 하는 것입니다. EC2 인스턴스를 생성하고 나면 시간이 서울이 아닌 표준 0 시로 셋팅되어 있습니다.
이 시간 설정은 tzselect 라는 툴을 이용해 간단히 설정 가능 합니다. 먼저 EC2 인스턴스에 콘솔 접속을 해서 tzselect 를 실행시키고 다음과 같이 진행시킵니다.
```
[ec2-user@ ~]$ tzselect
Please identify a location so that time zone rules can be set correctly.
Please select a continent or ocean.
1) Africa
2) Americas
3) Antarctica
4) Arctic Ocean
5) Asia
6) Atlantic Ocean
7) Australia
8) Europe
9) Indian Ocean
10) Pacific Ocean
11) none – I want to specify the time zone using the Posix TZ format.
#? 5
```
5 – Asia 를 선택합니다.
```
Please select a country.
1) Afghanistan   18) Israel     35) Palestine
2) Armenia   19) Japan     36) Philippines
3) Azerbaijan   20) Jordan     37) Qatar
4) Bahrain   21) Kazakhstan     38) Russia
5) Bangladesh   22) Korea (North)     39) Saudi Arabia
6) Bhutan   23) Korea (South)     40) Singapore
7) Brunei   24) Kuwait     41) Sri Lanka
8) Cambodia   25) Kyrgyzstan     42) Syria
9) China   26) Laos     43) Taiwan
10) Cyprus   27) Lebanon     44) Tajikistan
11) East Timor   28) Macau     45) Thailand
12) Georgia   29) Malaysia     46) Turkmenistan
13) Hong Kong   30) Mongolia     47) United Arab Emirates
14) India   31) Myanmar (Burma)     48) Uzbekistan
15) Indonesia   32) Nepal     49) Vietnam
16) Iran   33) Oman     50) Yemen
17) Iraq   34) Pakistan
#? 23
```
23 – Korea (South) 를 선택합니다.
```
The following information has been given:
Korea (South)

Therefore TZ=’Asia/Seoul’ will be used.
Local time is now: Mon Jan 27 21:50:45 KST 2014.
Universal Time is now: Mon Jan 27 12:50:45 UTC 2014.
Is the above information OK?
1) Yes
2) No
#? 1
```
정정되기 전 시간과 정정된 후의 시간이 나옵니다. 시간이 맞으면 1 을 선택합니다.
```
You can make this change permanent for yourself by appending the line
TZ=’Asia/Seoul’; export TZ
to the file ‘.profile’ in your home directory; then log out and log in again.
Here is that TZ value again, this time on standard output so that you
can use the /usr/bin/tzselect command in shell scripts:
Asia/Seoul
```
이제 profile 안에 TZ=’Asia/Seoul’; export TZ 를 입력하고 콘솔을 재시작 해야 합니다. /etc/profile 파일을 열어 맨 처음에 앞의 명령어를 입력 해 줍니다.
```
[ec2-user@~]$ sudo vi /etc/profile
# /etc/profile

# System wide environment and startup programs, for login setup
# Functions and aliases go in /etc/bashrc

# It’s NOT a good idea to change this file unless you know what you
# are doing. It’s much better to create a custom.sh shell script in
# /etc/profile.d/ to make custom changes to your environment, as this
# will prevent the need for merging in future updates.

TZ=’Asia/Seoul’; export TZ

pathmunge () {

…
```
이제 exit 명령을 눌렀다가 다시 콘솔을 재 접속을 해도 되고, 아니면 source 명령을 사용해서 바로 profile 을 재실행 시켜줍니다.
```
source /etc/profile
```
이제 date 명령으로 현재 시간을 확인합니다.
```
[ec2-user@ ~]$ date
2014. 01. 27. (월) 21:54:59 KST
```
시간이 잘 적용이 되었습니다.


이번에는 인스턴스에 Java SDK 를 설치 하도록 하겠습니다.

Oracle Java Download 페이지 에 가서 Java 버전을 내려받습니다. 저는 간단히 rpm 으로 설치했습니다. rpm 파일을 다운받고 다음 명령어를 실행해줍니다.
```
[ec2-user@]$ sudo rpm -ivh jdk-7u51-linux-x64.rpm
```
이제 java 명령어와 javac 명령어의 버전을 확인 해 봅니다.
```
[ec2-user@]$ java -version
java version “1.6.0_24”
OpenJDK Runtime Environment (IcedTea6 1.11.14) (amazon-65.1.11.14.57.amzn1-x86_64)
OpenJDK 64-Bit Server VM (build 20.0-b12, mixed mode)
[ec2-user@]$ javac -version
javac 1.7.0_51
```
javac 는 이번에 설치한 1.7 버전인데 java 는 기존에 설치되어 있는 버전이 그대로 남아 있습니다. /usr/bin/ 디렉토리에 있는 java 파일들을 확인합니다.
```
[ec2-user@]$ ls -l /usr/bin/java*
lrwxrwxrwx 1 root root 22 2013-12-10 23:46 /usr/bin/java -> /etc/alternatives/java
lrwxrwxrwx 1 root root 47 2013-12-10 23:46 /usr/bin/java6 -> //usr/lib/jvm/jre-1.6.0-openjdk.x86_64/bin/java
lrwxrwxrwx 1 root root 27 2014-01-24 09:01 /usr/bin/javac -> /usr/java/default/bin/javac
lrwxrwxrwx 1 root root 29 2014-01-24 09:01 /usr/bin/javadoc -> /usr/java/default/bin/javadoc
lrwxrwxrwx 1 root root 28 2014-01-24 09:01 /usr/bin/javaws -> /usr/java/default/bin/javaws
```
javac, javadoc, javaws 는 /usr/java/default 에 있는 디렉토리인데 java 는 /etc/alternatives/java 로 되어있군요. 간단히 저 java 파일을 삭제하고 ln -s 명령으로 새로 링크를 만들어줍니다. 그리고 다시 java 파일들을 확인해봅니다.
```
[ec2-user@]$ sudo rm /usr/bin/java
[ec2-user@]$ sudo ln -s /usr/java/default/bin/java /usr/bin/java
[ec2-user@]$ ls -l /usr/bin/java*
lrwxrwxrwx 1 root root 26 2014-01-24 09:08 /usr/bin/java -> /usr/java/default/bin/java
lrwxrwxrwx 1 root root 47 2013-12-10 23:46 /usr/bin/java6 -> //usr/lib/jvm/jre-1.6.0-openjdk.x86_64/bin/java
lrwxrwxrwx 1 root root 27 2014-01-24 09:01 /usr/bin/javac -> /usr/java/default/bin/javac
lrwxrwxrwx 1 root root 29 2014-01-24 09:01 /usr/bin/javadoc -> /usr/java/default/bin/javadoc
lrwxrwxrwx 1 root root 28 2014-01-24 09:01 /usr/bin/javaws -> /usr/java/default/bin/javaws
[ec2-user@]$ java -version
java version “1.7.0_51”
Java(TM) SE Runtime Environment (build 1.7.0_51-b13)
Java HotSpot(TM) 64-Bit Server VM (build 24.51-b03, mixed mode)
```
이제 java -version 명령어로 확인 해 보면 1.7 버전이 설치된 것이 보입니다.

profile 파일 맨 아래에 JAVA_HOME 과 CLASSPATH 를 추가합니다. 그리고 나서 profile 을 재시작합니다.
```
[ec2-user@ ]$ sudo vi /etc/profile
…
#JAVA
JAVA_HOME=/usr/java/default
CLASSPATH=$CLASSPATH:$JAVA_HOME/lib/*:.
export JAVA_HOME; export CLASSPATH
[ec2-user@ip-172-31-16-58 bin]$ source /etc/profile
```
이제 java 까지 전부 설치가 잘 되었습니다.