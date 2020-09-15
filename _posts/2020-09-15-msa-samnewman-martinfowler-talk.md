---
layout: post
title:  msa - SAM NEWMAN과 MARTIN FOWLER의 토론 정리
date:   2020-09-15 11:50:00 +0800
categories: msa
tag: msa
sitemap :
  changefreq : daily
  priority : 1.0
---

이 포스트는 [SAM NEWMAN과 MARTIN FOWLER의  msa에 관한 토론](https://www.youtube.com/watch?v=GBTdnfD6s5Q)을 정리합니다.

## 1. MSA를 언제 써야 하는가

SAM NEWMAN은 사람들이 보통 '뭘 했는지'에 집중하고 '결과'에 집중하지 않는다고 한다. 마이크로서비스를 적용할 땐 '결과'가 더 좋다고 생각할 때여야 한다.

MARTIN FOWLER는 msa 를 쓰지 말아야 하는 이유 말고, msa를 쓰면 좋은 점을 얘기해달라고 한다.

SAM NEWMAN은 1.more options to scale up applications, 2. independent deployablity, 3. isolation of processing around data를 이야기한다. 

참고로 James Lewis는 마이크로서비스의 장점으로 "msa buy you options"라고 했다. msa를 선택함으로써 다양한 선택을 할 수 있게된다는 것이다. 여기서 buy는 비용이 든다는 것을 의미한다.

SAM NEWMAN은 msa를 흑백으로 구분하면 안된다고 한다. 예를 들어, msa를 분산 시스템이고 monolith를 분산이 아니라고 하지만, monolith도 실제로는 분산 시스템이라고 할 수 있다. 여러 컴퓨터에서 데이터를 가져오기 때문이다. 기본적으로 모노리스로 선택하고, 특별한 경우에 msa 선택을 추천한다. msa는 default 선택지가 될 수 없고, msa가 도움이 된다고 생각될 때 하나의 모노리스 모듈로부터 가볍게 시작하고 발전시켜라.

**msa의 좋은 점 정리**

1. More options to scale up applications
2. (zero downtime) Independent deployability (ex: SaaS business where you can't afford downtime)
3. Limit the "blast radius" of failure
4. isolaction of processing around data(ex: health care industry, gdpr)
5. Enable a higher degree of organizational autonomy - distribute responsibilities into teams to reduce the amount of coordination with the rest of the organization, reduce the amount of coordination those teams need with other parts of the organizations

## 2. 분산 모노리스를 피하기 위해서는?

1. create a deployment mechamism
2. look for patterns(ex: a collection of microservices often being changed together)

어떻게 위 작업을 할 수 있는가?

- 지라를 사용해서 커밋을 하나의 작업으로 묶어라(?)
- 그리고 커밋들을 돌아봐라
- 커밋들을 서비스와 연결시켜, 그 커밋들이 영향도를 안다. 그러면 뭐가 같이 바뀌는지 알 수 있다.

서비스들을 캡슐화하면(공개 api를 줄이면?) 영향도를 가둘 수 있다.

## 3. 왜 독립적인 배포를 해야하나?

엄청나게 배포를 자주해야하는 시스템에서, 마이크로서비스는 릴리즈의 영향도를 줄일 수 있다. 모노리스로 못한다는게 아니라, 모노리스로 하면 훨씬 힘들다.

마틴 파울러는 msa로 배포하기 쉽다고는 하지만, 분산 시스템의 특성상 너무 어려워서 릴리즈의 영향도를 줄일 수 있다는 것만으로는 부족하다고 한다. 그냥 커다란 모노리스로 만들면 운영에 배포하기 전 전체를 테스트하기 쉽다는 장점도 있다. 릴리즈 영향도 줄이는 것도 있겠지만, 배포 용이성을 추구해야 하는 이유는 팀 사이즈 때문이기도 하다. 

**독립 배포를 추구해야하는 이유 정리**

1. 마이크로서비스를 활용하면 배포 시 영향도를 줄일 수 있다.
2. 팀 사이즈가 커질수록 배포를 조정하기 어려워진다.

### 4. 모듈성

모놀리스에서도 모듈화가 될 순 있겠지만, 우리는 항상 그 경계를 부시곤 한다. msa에서는 프로세스로 이루어진 경계를 부시는게 어렵기 때문에 잘 안부시게 된다.

### 5. 데이터 관련

서비스별로 데이터를 나눠 갖게 하는게 엄청 어려운데, 마이크로서비스를 추출하고 나면 이 서비스가 사용하던 예전 데이터베이스를 잘 이해해야 한다. msa에서는 데이터 쪽이 엄청 큰 이슈다.

### 6. 의사 결정 관련

마이크로서비스를 결정하기까지 의사결정하는게 어렵다는 얘기를 나눈다.







