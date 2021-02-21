---
layout: post
title:  bintegration의 식별자 생성 전략
date:   2021-02-21 11:35:00 +0800
categories: bintegration
tag: Architecture
sitemap :
  changefreq : daily
  priority : 1.0

---

맥북 메모리가 꽉찼다그래서 정리를 좀 해줬더니 인텔리제이 인덱싱이 끝날 기미가 보이지 않는다... 캐시를 지운건지 왜때문인지 모르겠다. 혹시 킬때마다 이래야하면  다시 깔아야 할거같다...

기다리는 동안 블로그나 좀 쓴다. 이 글은 지금은 되게 대략적으로 쓰고 나중에 좀 더 자세히 쓸 예정이다.

기본적으로 bintegration의 생성자 타입은 uuid이다. uuid로 한 이유는 도메인 스페시픽 하지도 않고, 마이크로서비스 패턴에서 uuid를 식별자 타입으로 쓰면 서비스 간 통신할 때 식별하기 편리하다고 추천했기 때문이다.

IDDD - 엔티티에 식별자 생성 전략이 다음과 같이 소개되었다.

1. 사용자가 하나 이상의 초기  고유값을 입력, 애플리케이션은 이것이 고유한지 확인
2. 애플리케이션이 고유성이 보장되는 알고리즘을 사용해 식별자 생성
3. 데이터베이스와 같은 영속성 저장소가 고유 식별자 생성
4. 다른 바운디드 컨텍스트에서 먼저 고유 식별자 결정

현재는 2번과 3번을 같이 사용하고 있다.

3번은 mysql을 쓰는 access서비스에서 쓰고 있고, 나머지 서비스들은 2번을 쓰고 있다.

```java
public abstract class BaseEntity {

    @Id
    @GeneratedValue(generator = "uuid2")
    @GenericGenerator(name = "uuid2", strategy = "uuid2")
    @Column(columnDefinition = "BINARY(16)")
    private UUID id;
```

uuid 생성 전략은 uuid2 전략을 썼는데, 자세히는 잘 모르고 위키에서 읽어보니 데이트랑 맥어드레스 등의 정보를 활용해서 만드는것같다. 이러면 내가 uuid를 생성해서 주입해주지 않아도, mysql이 uuid를 생성해준다.

area에서는 현재 이런식으로 하드코딩된 uuid 값을 넣어주고 있다. 개선할 부분이 많은 것 같다. 일단 저 uuid는 랜덤값인데, nameUUIDFromByte를 쓰면 그래도 의미를 담을 수 있을 것 같다.

```java
private final static UUID WANGSIMNI_ID = UUID
        .fromString("110841e3-e6fb-4191-8fd8-5674a5107c33");

Zone wangsimni = new Zone(WANGSIMNI_ID, "Wangsimni",
    GeoJsonPolygon.of(
        Arrays.asList(
            new Point(37.56186460715209, 127.03878873296458),
            new Point(37.55827838080759, 127.03892370195969),
            new Point(37.55753238253602, 127.04159778997553),
            new Point(37.561966294018255, 127.04168241338516),
            new Point(37.56186460715209, 127.03878873296458))
    )
);
```

chat 서비스에서는 다음과 같이 cql로 하드코딩된 값을 넣고 있다. 

```CQL
INSERT INTO chat_room ("name", "room_id") VALUES ('Wangsimni', 110841e3-e6fb-4191-8fd8-5674a5107c33);
```

이 부분은 고민할 게 많은 것 같다. 원래는 db 초기화를 flyway로 하고 싶었지만 카산드라같은 nosql 에서는 flyway를 못쓴대서 일단 cql로 입력하게 해놨다. 제대로된 것 같진 않지만 분류하자면 2번 식별자 생성 전략인 것 같다(어플리케이션이 정해주고 있으니...). 

이 cql을 만들 당시 db 초기화를 할 때 저수준에서 해결하는 게 더 좋다고 생각했다. 구체적인 근거는 없지만 뭔가 느낌적 느낌...? 디비와 관련된 관심사이니 디비가 해결하는 게 좋다고 생각했다. 그래서 mysql 뿐 아니라 다 db가 식별자를 생성하게끔 하고 싶었다(3번 전략). 근데 ddd 엔터티에 어플리케이션 레이어에서 주입해주는 것도 좋은 방법이라고 하여, 그냥 ContextRefreshed 이벤트를 컨슘하여 디비 데이터를 초기화해줄 수도 있을 것 같다.

그리고 이건 도메인 관련된 영역이지만, chat 서비스의 chat room은 현재 area 서비스의 zone과 같은 의미이기에, 4번 전략으로 수정할 예정이다. 이건 오래 전부터 생각하고 있던건데, iddd에서 이 전략은 보수적으로 사용하라고 해서 좀 더 검토해봐야할 것 같다.

아직도 인텔리제이 인덱싱이 끝나지 않았다...ㅜ_ㅜ