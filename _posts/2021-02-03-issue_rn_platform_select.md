---
layout: post
title:  삽질하며 배우는 즐거운 모바일 앱 만들기
date:   2021-02-02 11:35:00 +0800
categories: issue
tag: react-native
sitemap :
  changefreq : daily
  priority : 1.0
---

최근 React-Native로 모바일 앱을 만들고 있다.

그 와중에 별거 아니지만 굉장히 삽질한 경험을 기록해놓는다.

이번이 처음으로 모바일 앱을 만들어보는건데, Mac에서 안드로이드 애뮬레이터를 켜볼 수 있는지 몰랐다.

그래서 UI 개발은 IOS 시뮬레이터로만 확인하고 있었다.

다른 부분을 개발하다가 그저께 이제 안드로이드도 어떻게 하면 켤 수 있는지 알아봐야겠다고 생각했다.

하루동안 이런 저런 삽질 하는데... 계속 안드로이드에서 앱이 제대로 안켜지고 이런 오류가 났다.

![](http://dl.dropbox.com/s/j68r67o1fxlq1ax/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202021-02-03%20%EC%98%A4%EC%A0%84%2011.04.35.png?)

이게 IOS가 아니라서 안된다니...

뭐때매 이런 에러가 나는지 모르니 이것저것 다 건드려보면서 시간이 지나갔다. 계속 내 코드가 문제가 아니고 애뮬레이터 설정 등이 문제일거라고 생각했다.

그러다 아예 새로운 프로젝트를 만들어서 안드로이드를 켜보니 되는 것이다...

그때부터 내 코드에서 좀 짐작이 가는걸 다 건드려봐도 계속 같은 오류가 났다.

기존 코드 중 어디가 문제인지 찾는거보다

새로운 프로젝트에다가 하나씩 기존 코드를 복사해가면서 찾아봤다.

코드를 붙여가는데도 계속 안드로이드가 잘 되길래 "그냥 무슨 이윤지는 모르겠지만 일단 되게하자"는 생각으로 내가 만든 UI를 띄워봤다(이전까진 그냥 예제 UI만 띄웠다).

그러니까 또 저 에러가 난거다.

이제 거의 다 찾았다.

방금 전에 커밋한것까진 문제 없고 추가한 부분 중에 백프로 문제가 있는거다.

근데 이것저것 다바꿔도 여전히 안됐다.

그러다 결국 이 코드때문이라는 걸 알았다.

```typescript
const getIosStatusBarHeight = (isSafe: boolean) => {
  if (isNotIOS) {
    throw new Error('This is not IOS');
  }
  if (isIphoneX) {
    return getIphoneXStatusBarHeight(isSafe);
  }

  return 20;
};
```

This is not IOS는 내가 throw 하고 있던 에러였다^^;;;;;

저 코드는 OS에 따라 Status Bar Height를 다르게 만들어주려고 한 코드다. 사실 이 코드는 다른 사람의 코드를 베껴오면서 내가 조금 수정한 것인데, 내가 조금 수정한 것때매 버그가 생겼다.

```typescript
export const iosStatusBarHeight = (safe: boolean) => {
  if (isIOS) {
    return ifIphoneX(safe ? 44 : 30, 20);
  }
  return 0;
};

export const ifIphoneX = (iphoneXStyle: any, regularStyle: any) => {
  if (isIphoneX) {
    return iphoneXStyle;
  }
  return regularStyle;
};
```

그냥 ctrl C + V 했으면 버그는 없었을텐데...ㅎㅎ

iosStatusBarHeight를 구하는데 IOS인지 아닌지 체크하는 부분이 아쉬웠고(클라-서버가 계약이 제대로 안된느낌), 아예 IOS가 아니면 에러 쓰로우를 하게 만들었었다.

근데 왜인지 안드로이드일 때도 이 코드를 탄다.

```typescript
export const getStatusBarHeight = (isSafe: boolean) => {
  return Platform.select({
    android: getAndroidStatusBarHeight,
    ios: getIosStatusBarHeight(isSafe)
  });
};
```

여기에서 아예 android 환경에서는 getAndroidStatusBarHeight만 타게 돼있는건줄알았는데, 엉뚱하게 getIosStatusBarHeight가 에러를 뿜는다. 테스트로 Platform.select를 할 때 안드로이드 환경이면 android: getAndroidStatusBarHeight 이게 선택이 된다는걸 확인했는데 아직도 왜 getIosStatusBarHeight(isSafe) 이쪽에서 에러를 뿜는지 모르겠다. 더 딥하게 알아보고 싶기도 하지만 이정도 수준에서 넘어가려고 한다. 서버쪽에 더 중요하고 재밌는 이슈들이 많다. 나중에 생각나면 더 딥하게 공부해봐야겠다.

이번 이슈를 삽질하며 해결하며 디버거 사용의 중요성을 느꼈다. 자바 쓸때야 디버거 쓰는게 익숙해져서 어지간하면 다 디버거 돌리는데 프론트 작업할때는 콘솔로 찍어볼 때가 많았다. 근데 만약에 내가 이거 디버거 썼으면 더 빠르게 해결할 수 있었을것같다.

이틀동안 삽질하긴 했지만 그래도 후련하다~