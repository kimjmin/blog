---
layout: post
title: Jetty 기본 프로젝트 변경
subtitle: "Jetty 서버의 기본 프로젝트 변경하는 방법."
date: 2013-12-02
header-img: "network.jpg"
tags:
  - Java
  - Web
  - Jetty
categories:
  - Web
---
요즘 개발중인 클라이언트에 jetty 를 포함해서 배포하려고 설정하는 중입니다.

jetty 를 처음 설치하고 실행시켜서 http://localhost:8080 (기본 포트) 에 접속을 하면 아래와 같은 첫 화면이 나타납니다.
![](jetty.png)

이 화면의 소스는 `/webapps/test.war` 파일로 배포되어 있습니다.

프로젝트를 jetty를 통해 배포를 하려면 .war 또는 .zip 파일로 묵은 뒤
`/webapps/project.war`
위 폴더에 업로드 한 뒤
http://localhost:8080/project
이와 같은 경로로 실행 시킬 수 있습니다.

http://localhost:8080
이 기본 주소를 실행시켰을 때 내가 만든 프로젝트를 기본으로 배포하고자 하면 다음과 같이 합니다.

먼저 /context/ 경로 아래 내 프로젝트와 같은 이름의 xml 파일을 만들어줍니다.
> /context/project.xml

xml 파일 내용을 다음과 같이 하고
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE Configure PUBLIC "-//Jetty//Configure//EN" "http://www.eclipse.org/jetty/configure.dtd">
<Configure>
  <Set name="contextPath">/</Set>
  <Set name="war"><SystemProperty name="jetty.home" default="."/>/webapps/project.war</Set>
  <Get name="securityHandler">
    <Set name="loginService">
      <New>
        <Set name="name">Test Realm</Set>
        <Set name="config"><SystemProperty name="jetty.home" default="."/>/etc/realm.properties</Set>
      </New>
    </Set>
    <Set name="authenticator">
      <New>
        <Set name="alwaysSaveUri">true</Set>
      </New>
    </Set>
    <Set name="checkWelcomeFiles">true</Set>
  </Get>
</Configure>
```
이 항목에 처음에 열고자 하는 .war 파일을 셋팅 해 주면 첫 화면이 해당 프로젝트로 나타납니다.
```xml
<Set name=”war”><SystemProperty name=”jetty.home” default=”.”/>/webapps/project.war</Set>
```
