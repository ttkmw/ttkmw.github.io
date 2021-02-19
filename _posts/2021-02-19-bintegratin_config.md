---
layout: post
title:  bintegration의 구성 관리
date:   2021-02-19 17:50:00 +0800
categories: bintegration
tag: project
sitemap :
  changefreq : daily
  priority : 1.0
---

## 스프링 마이크로서비스 코딩공작소에 나온 내용

msa에서 사람이 수동으로 구성하거나 배포하면 구성편차와 예상하지 못한 장애, 애플리케이션 확장 요구에 대한 지체 시간이 발생할 수 있다고 한다.

구성 정보를 실제 코드 외부로 분리하면 관리하고 버전 제어를 해야 할 외부 의존성이 생긴다. 애플리케이션 구성을 제대로 관리하지 못하면 탐지하기 어려운 버그와 예상치 못한 장애를 만드므로 애플리케이션 구성 데이터를 추적하고 버전 제어하는 것은 아무리 강조해도 지나치지 않다.

존 카넬이 실제로 겪은 일화를 소개해주는데, 한 프롷젝트에서 발견한 중요한 문제 중 하나가 애플리케이션 팀이 데이터비에스와 서비스의 엔드포인트에 대한 모든 구성을 프로퍼티 파일로 관리한다는 것이었다고 한다. 이 프로퍼티 파일들은 수동으로 관리되며 버전 관리도 되지 않았다. 구성 파일이 엉망이어서 수백 개의 서버와 서버에서 실행 중인 애플리케이션에서 분산된 1만 2천개의 구성 파일을 마이그레이션 해야 했다고 한다. 이 파일들은 애플리케이션 서버 구서용 파일도 아니고 애플리케이션 구성용 파일이었다고 한다. 문제의 팀은 처음엔 소수의 애플리케이션으로 구성 전략을 수립했지만 5년간 구축하고 배포하는 웹 앱이 폭발적으로 증가했다고 한다. 구성 관리 방식의 재작업은 우선순위에서 밀리기 일쑤였다.

이 사례에서 구성 관리를 중앙화하는게 중요하다는 걸 느낄 수 있었다.

## bintegration에 적용

bintegraion은 config server를 두고 config server에 구성 관리를 집중화한다. 그리고 비밀 정보를 보호하기 위해 대칭키 암호화를 사용하고, 대칭키는 AWS Secrets Manger로 관리한다.

![configserver_package](https://dl.dropbox.com/s/lqybw0cpio78sjd/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202021-02-19%20%EC%98%A4%ED%9B%84%2012.51.58.png)

위와 같이 config 서버는 src/main/resources 밑에 각 서비스 구성 정보를 폴더별로 갖고, 각 폴더는 환경에 따른 구성 파일이 있다.

예를 들어, 

http://[config-server host]:[config-server port]/access/dev에 Get 요청을 하면, 다음과 같은 응답을 받는다.

```json
{
    "name": "access",
    "profiles": [
        "dev"
    ],
    "label": null,
    "version": "6aa3017e9625c6ecb621eb755b6f53425d9b61e2",
    "state": null,
    "propertySources": [
        {
            "name": "https://github.com/beyond-eyesight/bintegration/config/src/main/resources/config/access/access-dev.yml",
            "source": {
                "spring.datasource.url": "jdbc:mysql://test-access-1.cuwocpon5oew.ap-northeast-2.rds.amazonaws.com:3306/access",
                "spring.datasource.username": "beyondeyesight",
                "spring.datasource.password": "{cipher}4517326d65dc7a7424d981f3f0c42b90da31317ef28dec52246af9a5a98ea480",
                "spring.jpa.show-sql": true,
                "spring.jpa.hibernate.ddl-auto": "update",
                "spring.jpa.properties.hibernate.show-sql": true,
                "spring.jpa.properties.hibernate.format_sql": true,
                "spring.jpa.properties.hibernate.dialect": "org.hibernate.dialect.MySQL8Dialect"
            }
        }
    ]
}
```

다음과 같이 중요한 정보는 암호화되어 전달받는다.

"spring.datasource.password": "{cipher}4517326d65dc7a7424d981f3f0c42b90da31317ef28dec52246af9a5a98ea480"

그리고 task-definition에

```json
"secrets": [{
  "name": "ENCRYPT_KEY",
  "valueFrom": "arn:aws:secretsmanager:ap-northeast-2:174579883227:secret:ENCRYPT_KEY-I43Lmf"
}]
```

이렇게 SecretsManager arn을 설정해주면 서비스를 배포할 때 인스턴스 환경 변수에 대칭키를 할당하고, 그 대칭키로 중요 정보를 복호화한다.

비대칭키를 활용하면 좀 더 보안적으로 좋다고 하는데, 현재 단계에서는 대칭키도 충분한 것 같다.

ecs-cli compose로 수동 배포할 때는

```yaml
version: "3"
services:
  access:
    image: beyondeyesight/access:latest
    ports:
      - "8081:8081"
    environment:
      ENCRYPT_KEY: ${ENCRYPT_KEY}
```

이렇게 로컬 환경의 환경 변수를 활용할 수 있다.

