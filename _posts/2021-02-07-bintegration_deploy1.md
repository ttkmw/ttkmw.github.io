---
layout: post
title:  bintegration의 배포 절차 1
date:   2021-02-17 21:30:00 +0800
categories: bintegration
tag: project
sitemap :
  changefreq : daily
  priority : 1.0
---

노트북을 3년 넘게 쓰다보니 좀 힘들어한다. 하드도 모자를때가 종종 있고 도커때문인지 메모리가 부족하다고 할 때도 있다. 예전에는 뭐 빌드를 한다거나, 테스트를 한다거나 등등 어떤 작업때매 기다려야할 때 그냥 멍때리면서 기다렸는데, 이제 그냥 기다리다가는 너무 시간 낭비가 심한 정도라 자투리 시간을 활용하려고 한다. 인텔리제이가 메모리 부족하다그래서 정리되는 중 글쓴다. 사실 이건 개발 뿐 아니라 생활 전반적으로 최근 생긴 긍정적 변화다. 뭔가 해야할 것도 많고 하다 보니 병렬적으로 일 처리 하는게 조금씩 익숙해진다. 예전에는 하나에 집중하면 다른거는 아예 신경도 쓰기 싫어했는데, 병렬 작업이 익숙해지는 건 되게 좋은 것 같다.

bintegration의 배포 절차는 특별할 것 없이 빌드 -> 테스트 -> 배포 이다.

CI/CD 도구는 github action을 활용한다.

처음엔 travis ci를 이용하려고 했는데 쓰다보니 너무 오래걸려서 github action으로 바꿨다. 깃허브에서 만든 것이니 깃허브와 더 연동이 잘 될 거라는 기대도 있었다.

![githubaction_button](https://dl.dropbox.com/s/d9ulyxot9su8yvn/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202021-02-17%20%EC%98%A4%ED%9B%84%209.10.58.png)

github action에서 서비스 이름을 써서 run 하면



![github_action](https://dl.dropbox.com/s/23lg3t3a85y0tsg/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202021-02-17%20%EC%98%A4%ED%9B%84%2012.07.21.png)

빌드 및 테스트 -> 도커 허브에 배포 -> ecs에 배포 한다.

마이크로서비스 패턴을 보면 플랫폼 테스트도 해야한다고 하는데 그건 아직 안하고 있다.

그리고 아쉬운게 배포를 하려면 서비스 이름을 정확히 입력해야한다. gradlew 파일을 그 서비스 이름을 토대로 run 하기 때문이다. 만약에 서비스 이름을 잘못 입력하면 Grant execute permission for gradlew 단계에서 끝난다. 뭔가 서비스 이름을 깜빡해도 후보를 볼 수 있으면 더 좋을거같은데 아쉽다. 그래도 잘못 입력했을때 배포가 이상하게 되면 큰 문제인데 빠르게 실패하기 때문에 이정도면 만족한다.

github action을 처음 쓸 때 원하는 액션(태깅하기)을 마켓플레이스에서 찾았는데, 쓰려고 하니까 뭔가 잘 안됐었다. 그래서 깃헙 액션을 한번 만들어봤는데, 당시에 '드디어 나도 스크립트를 공부할 때가 온건가'하고 꽤 재밋게 했었다. 이제까지 스크립트를 꼭 써야 하는 일이 거의 없었고 그래서 스크립트는 자신이 없었는데, 이번 기회에 공부해봐야겠다고 생각했다.

**tag_release.sh**

```shell
#!/bin/sh -l

sh -c "
TARGET_URL=\"$INPUT_GITHUB_API_URL/repos/$INPUT_GITHUB_REPOSITORY/releases?access_token=$INPUT_GITHUB_TOKEN\";

curl -k -X POST -H \"Content-Type: application/json\" \$TARGET_URL -d @- << EOF
{
\"tag_name\": \"$INPUT_BUILD_NAME\",
\"target_commitish\": \"$INPUT_GITHUB_TARGET_BRANCH\",
\"name\": \"$INPUT_BUILD_NAME\",
\"body\": \"Release of $INPUT_BUILD_NAME\",
\"draft\": true,
\"prerelease\": true
}
EOF
"
```

**action.yml**

```yaml
name: "Tag Release"
description: "tag release"
author: "beyondeyesight"

branding:
  icon: 'box'
  color: 'green'

inputs:
  GITHUB_TOKEN:
    description: "access token for github api"
    required: true
  BUILD_NAME:
    description: "build name"
    required: true
  GITHUB_REPOSITORY:
    description: "full name of git repository. ex: beyond-eyesight/chat"
    required: true
  GITHUB_API_URL:
    description: "http api url of github"
    required: true
  GITHUB_TARGET_BRANCH:
    description: "target branch to merge"
    required: true
    default: main
runs:
  using: "docker"
  image: "Dockerfile"
```

**Dockerfile**

```dockerfile
FROM debian:9.5-slim

ADD tag_release.sh /tag_release.sh
RUN chmod +x /tag_release.sh
RUN apt-get update && apt-get install -y curl
ENTRYPOINT ["/tag_release.sh"]
```

이게 그때 만든 깃헙 액션인데, 만들었을 땐 뿌듯했지만 결과적으로 필요가 없어졌다. 이미 만들어져있는거를 좀 다시 만져보니까 쓸 수 있게됐고, 그게 더 잘만든거같아서 그걸 쓰고있다. 헛수고한 느낌이 있지만 그래도 만든게 아쉬워서 지우진 못하고있다.

현재는 ecs에 배포를 하면, 수동으로 컨테이너 인스턴스에 들어가서 컨테이너를 수동으로 켜줘야한다. 맨 처음 배포할때는 안그래도 되는데, 두번째부터는 기존 컨테이너가 꺼지고 새로운 컨테이너가 켜지진 않더라. 이부분이 좀 아쉽다. 스프링 마이크로서비스 코딩공작소에서는 배포자동화에 대해 여기까지 알려주는데, 아직 자동화가 다 안된 느낌이다. ecs를 좀더 공부하면 해결될수도 있을 것 같다. 아니면 무중단배포를 구현하면 이부분이 개선될지도 모르겠다. 그래서 제목을 배포 절차 1편이라고 했다. 개선 후 2편을 쓰거나 나중에 이걸 수정해야지.