---
layout: post
title: AWS 설치 - APM
subtitle: "AWS 설치 – Apache, MySQL, PHP, phpMyAdmin"
date: 2014-01-18
header-img: "aws_icons.png"
tags:
  - AWS
  - ICT
  - APM
categories:
  - AWS
---
생성한 AWS 인스턴스에 APM 모듈을 설치 해 보겠습니다.

AWS 인스턴스 생성은 이전 포스트를 참고하세요

사실 APM 설치는 인스턴스 생성 전에 AWS Marketplace 에 가면 원클릭에 설치가 가능합니다. 하지만 마켓에서 설치를 해 보니 잘 안되는 것도 있고 해서 직접 설치를 했습니다.

설치 방법은 AWS Document 페이지에도 잘 나와 있습니다. 다만 몇가지 추가로 설정 해야 할 것들이 있어서 기록 할 겸 포스트 남깁니다.

APM 설치 전에 먼저 인스턴스의 Security Group 을 설정합니다. 웹 서버를 사용하려면 기본적으로 http 포트인 80번 포트와 https 포트인 443 포트, 그리고 ssh 포트인 22번 포트를 열어 주어야 합니다.

Security Groups 메뉴에 들어갑니다. 기존의 그룹에 추가를 해도 되고 새로 Create Security Group 버튼을 눌러 새로운 그룹을 추가해도 됩니다. Security Group 을 선택하고 하단에 Inbound 탭을 선택한 뒤 port range 에 22, 80, 443 포트를 추가합니다. 더 오픈할 포트가 있으면 1000-2000 이런 식으로 1000부터 2000번 까지의 전체 포트도 오픈이 가능합니다.
![](apm_1.png)

이제 서버에 접속해서  yum 을 최신버전으로 업데이트를 합니다.
```
[ec2-user ~]$ sudo yum update -y
```

다음은 yum groupinstall 을 사용해서 Apache, MySQL, PHP 를 동시에 설치합니다.
```
[ec2-user ~]$ sudo yum groupinstall -y “Web Server” “MySQL Database” “PHP Support”
```

다음은 php-mysql 패키지를 설치합니다.
```
[ec2-user ~]$ sudo yum install -y php-mysql
```

이제 Apache 웹서버를 실행시킵니다.
```
[ec2-user ~]$ sudo service httpd start
```

시스템이 시작 될 때 마다 웹 서버가 실행되도록 등록 합니다.
```
[ec2-user ~]$ sudo chkconfig httpd on
```

정상적으로 서버가 실행되었는지 확인합니다.
```
[ec2-user ~]$ chkconfig --list httpd
httpd           0:해제 1:해제 2:활성 3:활성 4:활성 5:활성 6:해제
```

시스템 메시지가 한글로 나타나도록 하려면 i18n 파일을 수정합니다.
```
[ec2-user ~]$ sudo vi /etc/sysconfi/i18n
LANG=”ko_KR.UTF-8"
SUPPPORTED=”ko_KR.UTF-8:ko_KR.eucKR:ko_KR:ko”
SYSFONT=”lat0-sun16"
SYSFONTACM=”8859-15"
```

이제 인스턴스의 공개 주소에 접속해서 Apache 화면이 정상적으로 나타나는지 확인합니다.
![](apm_2.png)

기본 웹 페이지 경로는 /var/www/html 입니다.
```
[ec2-user ~]$ ls -l /var/www
total 16
drwxr-xr-x 2 root root 4096 Jul 12 01:00 cgi-bin
drwxr-xr-x 3 root root 4096 Aug 7 00:02 error
drwxr-xr-x 2 root root 4096 Jan 6 2012 html
drwxr-xr-x 3 root root 4096 Aug 7 00:02 icons
```

ec2-user 가 경로의 파일들을 변경할 수 있도록 www 그룹을 추가합니다.
```
[ec2-user ~]$ sudo groupadd www
[ec2-user ~]$ sudo usermod -a -G www ec2-user
```

이제 서버에 다시 재접속 해서 그룹을 확인합니다.
```
[ec2-user ~]$ exit
[ec2-user ~]$ groups
ec2-user wheel www
```

/var/www 경로의 하위 디렉토리와 파일들의 권한을 변경합니다.
```
[ec2-user ~]$ sudo chown -R root:www /var/www
[ec2-user ~]$ sudo chmod 2775 /var/www
[ec2-user ~]$ find /var/www -type d -exec sudo chmod 2775 {} +
[ec2-user ~]$ find /var/www -type f -exec sudo chmod 0664 {} +
```

이제 phpinfo 페이지를 생성해서 php 정보를 확인합니다.
```
[ec2-user ~]$ echo “<?php phpinfo(); ?>” > /var/www/html/phpinfo.php
```

http://서버/phpinfo.php 페이지에 접속해서 PHP 정보를 확인합니다.
![](apm_3.png)

확인 후에는 phpinfo 파일은 다시 삭제하도록 합니다.
```
[ec2-user ~]$ rm /var/www/html/phpinfo.php
```

이제 MySQL 을 시작합니다. 저는 MySQL 을 시작하기 전에 DB를 현재 설정된 8GB의 EBS 볼륨이 아니라 예전에 마운트한 400GB의 system 경로에 저장하기 위해 파일 설정을 변경하였습니다. 변경할 디렉토리에 mysql 이 저장 권한을 갇도록 지정합니다.
```
[ec2-user ~]$ chown mysql.mysql /mysqldata
```

그리고 /etc/my.cnf  및 /etc/init.d/mysqld 파일에서 datadir 부분을 새로 생성한 디렉토리경로로 수정합니다.
***ex) datadir=/usr/local/mysql/data ==>  datadir=/mysqldata***

이제 mysql 데몬을 시작합니다.
```
[ec2-user ~]$ sudo service mysqld start
Initializing MySQL database: Installing MySQL system tables…
OK
Filling help tables…
OK
To start mysqld at boot time you have to copy
support-files/mysql.server to the right place for your system
PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
…
Starting mysqld: [ OK ]
```

MySQL 을 처음 실행시켰기 때문에 보안 설정을 해 줍니다.
```
[ec2-user ~]$ sudo mysql_secure_installation
```

처음에 root 계정의 비밀번호를 물어보는데 처음 실행했기 때문에 그냥 엔터를 치면 넘어갑니다. 그 이후에 새로운 root의 비밀번호를 저장하기 위해 물어봅니다. root 비밀번호를 저장 한 이후 나타나는 질문은 모두 엔터로 넘어갑니다.

이제 MySQL 도 설치가 완료되었습니다.

별도의 Admin 관리 툴이 없기 때문에 MySQL 을 편리하게 사용할 수 있는 phpMyAdmin 을 설치합니다. 설치 파일은 [phpMyAdmin 페이지](https://www.phpmyadmin.net/)에서 다운로드 할 수 있습니다. 파일을 다운로드 받아 웹 디렉토리에 압축을 푼 뒤 브라우저로 접속하면 간단하게 바로 실행이 됩니다.

… 가 되어야 하는데 실행이 되지 않습니다. (두둥)

apache 웹 로그를 살펴보면
```
PHP Fatal error:  Call to undefined function mb_detect_encoding()
```

라는 오류 메시지가 찍혀 있습니다. phpMyAdmin 을 실행하기 위해서는 php에 mbstring 이라는 모듈이 설치가 되어 있어야 합니다.

먼저 /etc/php.ini 파일을 열어 맨 아래에 다음 내용을 추가해줍니다.
```
[ec2-user ~]$ sudo vi /etc/php.ini
extension=mysqli.so
extension=mbstring.so
```

그리고 위 패키지들을 설치한 뒤 웹서버를 재시작합니다.
```
[ec2-user ~]$ sudo yum install php-mysqli
sudo yum install php-mbstring
sudo /etc/init.d/httpd restart
```

이제 다시 접속해보시면 정상적으로 phpMyAdmin 페이지가 나타납니다. root 와 아까 설정한 패스워드로 들어가 사용하면 됩니다.

![](apm_4.png)
