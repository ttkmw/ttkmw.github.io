---
layout: post
title:  삽질하며 알게된 java 버전과 gradle 버전의 깊은(?) 의존도
date:   2021-04-29 19:02:00 +0800
categories: issue
tag: build
sitemap :
  changefreq : daily
  priority : 1.0
---

## 서론

디버깅 중 도저히 뭐가 원인인지 모르겠어서 이것저것 다 바꾼 적이 있다. 그 때 자바 버전이 문제인가 싶어서 java 16버전으로 프로젝트를 만든 적이 있다. 그 이후부터 갑자기 `gradle build`는 되지만 `./gradlew build`는 안되는 이슈가 생겼다. 당시에는 이건 내가 집중하고 있는 부분이 아니었기에 그냥 `gradle build`로 빌드 후 디버깅을 했다. 하지만 도커 컨테이너 안에서 keycloak이 빌드하다보니, `./gradlew build` 이 안되는게 진짜 문제가 됐다. 만약에 내가 실력이 좋아서 java 16버전으로 트라이를 안해봤다면 불필요한 삽질은 안해도 됐을 것 같은데, 삽질을 하며 그래도 알게 된 게 있어 적는다.

## 결론

1. gradle 버전에 따라 지원하는 java 버전 범위는 다르다.  [참고](https://docs.oracle.com/javase/specs/jvms/se15/html/jvms-4.html)

2. ./gradlew는 JAVA_HOME이 설정됐는지 그렇지 않은지에 따라 java 버전을 인식한다.

   ```groovy
   # Determine the Java command to use to start the JVM.
   if [ -n "$JAVA_HOME" ]; then
     if [ -x "$JAVA_HOME/jre/sh/java" ]; then
       # IBM's JDK on AIX uses strange locations for the executables
       JAVACMD="$JAVA_HOME/jre/sh/java"
     else
       JAVACMD="$JAVA_HOME/bin/java"
     fi
     if [ ! -x "$JAVACMD" ]; then
       die "ERROR: JAVA_HOME is set to an invalid directory: $JAVA_HOME
   ```

## 본론

자바 11 버전, gradle 6.8.3 버전을 쓰고 있다. gradle 6.8.3이 지원하는 main 자바 버전은 11이 아니다. 그래서 프로세스에 $JAVA_HOME이 설정되어 있지 않으면, 다른 버전(아마 메이저일듯)을 쓰려고한다. 그러면 내 자바 11버전과 gradle이 빌드하려고 하는 버전이 달라서 빌드가 안되는 듯하다.

## 아직도 모르겠는거

1. jdk 와 jre이가 정확히 뭐가 다른진 아직 잘 모르겠다. 에러 메시지를 봤을 때 jre 버전이 다르다고 했는데, 그래서 내가 jdk와 jre 개념의 차이를 몰라서 생긴 버그인줄알았다. 근데 그렇지 않았고, 그닥 궁금하진 않아서 더 찾아보진 않았다.
2. 왜 인텔리제이로 16 버전 프로젝트를 시작한 후 이 문제가 생긴진 모르겠다. 기존에도 JAVA_HOME은 설정되지 않아있었고, java와 gradle의 버전은 같은데... 아몰랑





