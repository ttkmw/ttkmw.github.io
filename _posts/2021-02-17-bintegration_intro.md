---
layout: post
title:  bintegration 소개
date:   2021-02-17 21:40:00 +0800
categories: bintegration
tag: project
sitemap :
  changefreq : daily
  priority : 1.0
---

코딩 더 하기가 좀 애매해서 글이나 좀 더 쓴다. 집중 안될때는 블로그 글쓰기가 짱인거같다.

bintegration은 big integration을 줄인거다. 대통합이라는 뜻을 쓰고 싶었는데 찾아보니까 union 등이 나왔는데 뭔가 입에 착 안붙어서 bintegration이라고 했다.

맨 처음 프로젝트를 할 때는 뭔가 내가 구현하려는 아이디어에 대해서 나도 잘 이해를 못했다. 두리뭉실한느낌. msa를 구현해보고 싶긴 한데 내 아이디어는 비즈니스적으로 여러개를 나눌만큼 크지 못했다... 근데 공부를 하면서 조금씩 '나중에는 이런 기능 넣으면 좋겠다', '나중에 이런 서비스도 넣으면 좋겠다'고 생각하다가 '내가 하려고 하는거는 결국에 사람과 사람을 연결하려는거네' 라는 생각이 들어서, 모든 것을 연결, 통합하겠다는 포부로 bintegration이라고 지었다.

현재는 access, apigateway, discovery, fellowship, area, config 서비스가 있다. 아키텍처도 잇체인 따라 그려보긴 했는데, 서비스가 자주 바뀔 거 같아서 나중에 좀 정착되면 여기다가도 올려야겠다.

최근 fellowship 서비스를 인간관계와 관련된 로직을 수행하는 human relationship 서비스로 바꿔야겠다는 생각이 들었는데, 이걸 구현할때 좀 재밌을거같아서 기대된다. 그리고 빨리 결제도 해보고싶다. access에 oauth2 인가 도 얼렁 해야겠다.

