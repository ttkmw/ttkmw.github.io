---
layout: post
title:  예제로 이해하는 IoC
date:   2020-04-06 02:25:00 +0800
categories: 토비의스프링
tag: OOP
sitemap :
  changefreq : daily
  priority : 1.0


---

이 포스트에서는 지난 [토비의스프링 1장](https://p-vibe.github.io/2020/03/25/toby01) 글을 쉽게 이해하기 위해 일상 생활(가구만들기)에 IoC를 적용해봤습니다. 예제코드는 [이 리포지토리](https://github.com/p-vibe/ioc-example)에서 보실 수 있습니다. 글에도 Commit은 확실히 표시해놨으니, 글과 커밋을 매칭시키며 보면 좋을 것 같습니다.

먼저 상황을 가정해보겠습니다. 가구를 만드려면 일단 가구를 만들 수 있는 기술자(이하 FurnitureMaker)가 있어야겠죠. 다음과 같이 정의해보겠습니다. ***Commit - feat: 필요한 객체 정의***

```java
class FurnitureMaker {
    private final String name;

    FurnitureMaker(String name) {
        this.name = name;
    }
}
```

그리고 가구를 만들기 위해 목재(이하 Wood)와 쇠망치(이하 IronHammer)도 필요할 것입니다.

```java
class Wood {}
```

```java
class IronHammer {}
```

둘 다 현재 딱히 기능이 생각나지도 않고, 필드도 생각이 안나서 정의만 해놓았습니다. 잘짜고 못짜고를 떠나, 설명을 위해 필요한 객체를 정의한 것 뿐이니 조금 의아해도 넘어가주세요...!^^;

이제 가구를 만들기 위한 준비가 끝났습니다. 의자(이하 Chair)를 만들어봅시다. Chair를 만들기 위해서는 적어도 Wood 5개는 필요하다고 합니다. ***Commit - feat: 1 - 초난감 FurnitureMaker - makeChair(), makeDesk()***

```java
class Chair {
    static final int WOOD_AMOUNT = 5;

    Chair(List<Wood> woods) {
        if (woods.size() != WOOD_AMOUNT) {
            throw new IllegalArgumentException(String.format("의자를 만들기 위해 %d개의 나무가 필요합니다.", WOOD_AMOUNT));
        }
    }
}
```

다음은 FurnitureMaker가 Chair를 만드는 코드입니다.

```java
// FurnitureMaker.java
Chair makeChair() {
    List<Wood> woods = new ArrayList<>();
    for (int i = 0; i < Chair.WOOD_AMOUNT; i++) {
        woods.add(new Wood());
    }

    IronHammer ironHammer = new IronHammer();
    return ironHammer.makeChair(woods);
}
```

실제로 FurnitureMaker가 Chair를 잘 만드는 지 확인해볼까요?

```java
class FurnitureMakerTest {
    @Test
    void makeChair() {
        FurnitureMaker furnitureMaker = new FurnitureMaker("root");
        Chair chair = furnitureMaker.makeChair();
        assertThat(chair).isNotNull();
    }
}
```

테스트가 통과하네요. FurnitureMaker는 Chair를 만들 수 있습니다. 그런데 이 코드에는 많은 문제가 있습니다. 어떤 문제일까요? 가장 큰 문제는 makeChair() 메소드 하나에 여러가지 관심사항이 있다는 것입니다. makeChair() 함수에는 다음과 같은 관심사가 있습니다.

**makeChair()의 관심사항**

1. 필요한 양의 Wood를 준비(생성)하는 것
2. 필요한 IronMaker를 준비(생성)하는 것
3. 준비물(Wood, IronMaker)를 활용하여 Chair를 만드는 것

많은 관심사가 있다는 게 첫번째 문제입니다. 특히, 1번이나 2번은 중복 가능성이 많다는 것입니다. 예를 들어, FurnitureMaker가 Chair 말고도 Desk도 만든다고 하면 어떻게 될까요?

```java
// FurnitureMaker.java
Desk makeDesk() {
    List<Wood> woods = new ArrayList<>();
    for (int i = 0; i < Desk.WOOD_AMOUNT; i++) {
        woods.add(new Wood());
    }

    IronHammer ironHammer = new IronHammer();
    return ironHammer.makeDesk(woods);
}
```

## 코드 개선 1: 메소드 추출

Desk를 만들 때도 Wood와 IronHammer를 준비해야 합니다. 일단 이 중복부터 없애봅시다. ***Commit - feat: 2 - 메소드 추출(getWoods(), getIronHammer())을 통한 중복 제거***

```java
// FurnitureMaker.java
private List<Wood> getWoods(int woodAmount) {
    List<Wood> woods = new ArrayList<>();
    for (int i = 0; i < woodAmount; i++) {
        woods.add(new Wood());
    }
    return woods;
}
```

```java
// FurnitureMaker.java
private IronHammer getIronHammer() {
    return new IronHammer();
}
```

이제 Chair, Desk 뿐 아니라, 장롱을 만들든 옷장을 만들든 해당 함수에서 getWoods()와 getIronHammer를 호출하면 되기 때문에, 기존의 중복은 없어졌습니다.

이제 맨 처음 코드보다는 좋아졌다고 할 수 있지만, 여전히 문제가 있습니다. 

1. 의자는 어떤 목재를 쓰느냐에 따라 그 품질이 달라진다고 합니다. 목재가 어디 한둘이겠습니까? 지나가다 굴러다니는 목재도 있을 것이고, 천년 묵은 귀한 목재도 있을 것입니다. 지금은 현실 세계와 다르게, Wood라는 객체 하나 뿐입니다.
2. 장인은 도구를 탓하지 않는다는 말도 있지 않습니까? 지금 FurnitureMaker는 IronHammer 밖에 못씁니다. 망치가 꼭 쇠여야 합니까? 돌망치, 나무망치, 심지어 고수는 뿅망치로도 의자를 만들 수 있을 것입니다. 지금은 현실 세계와 다르게, 망치는 IronHammer라는 객체 하나 뿐입니다.

## 추상화

이를 해결하기 위해 먼저 Wood와 Hammer를 interface를 활용하여 추상화해봅시다. 어차피 Wood든 Hammer든 인터페이스로 추상화한다는 건 똑같기 때문에 Hammer 코드만 적어놓겠습니다. ***Commit - feat: 3 - Wood, Hammer 추상화***

```java
interface Hammer {
    Chair makeChair(List<Wood> woods);

    Desk makeDesk(List<Wood> woods);
}
```

```java
class IronHammer implements Hammer {
    @Override
    public Chair makeChair(List<Wood> woods) {
        return new Chair(woods);
    }

    @Override
    public Desk makeDesk(List<Wood> woods) {
        return new Desk(woods);
    }
}
```

여기에서 다음을 알 수 있습니다.

***추상화를 하면 바꿀 수 있다.***

사실 이 문구는 어찌보면 이번 포스트의 가장 핵심이 되는 것이기 때문에, 나중에 적으려고 했는데 의도치 않게 벌써 추상화가 나와버렸네요... 토비의 스프링에는 Connection 객체를 예제로 활용하기 때문에 이 부분이 안나와있는데요. 설명에 필요할 것 같아 벌써부터 추상화를 했습니다... 아쉽지만 계속 진행하겠습니다.

> ***여기서 질문!***
>
> 뭘 바꿀 수 있냐고요? 정답은 "구현체를 바꿀 수 있다" 입니다. 예를 들어, 쇠망치든 뿅망치든 둘 다 망치죠? 그니까 망치라고 추상화하면(뭉뚱그리면), FurnitureMaker가 쇠망치든 뿅망치든 다 쓸 수 있다는 것입니다.

> ***여기서 팁!***
>
> 현재 "망치"로 추상화했죠. 쇠망치, 뿅망치의 공통점을 추려서 말입니다. 그런데 어떤 장인은 망치가 아니더라도, 돌멩이로도 의자를 만들 수 있다고 생각해봅시다. 어쩌면 굴러다니는 커피캔으로도 의자를 만들 수도 있겠죠. 그러면 망치, 돌멩이, 커피캔도 공통점이 있다고 볼 수 있겠죠? 예를 들어, '단단하다'와 같은 공통점이 있다고 생각할 수 있죠. 이 장인은 진짜 도구는 가리지 않고, 뭐만 있기만 하면 의자를 만들 수 있나봅니다. 따라서, 망치든 돌멩이든 커피캔이든 상관없이 도구면 되니, "도구"라고 추상화해봅시다. "망치"는 어찌보면 추상화의 결과물이었는데, "망치"를 또 추상화해서 "도구"가 됐네요? 이럴 때 한 단계 더 추상화했다는 표현을 쓰는 것 같아요. 그리고 망치로 추상화했을 때보다 도구로 추상화했을 때 더 "고수준의 추상화이다"라고 표현하더라고요.

FurnitureMaker의 코드는 다음과 같습니다.

```java
class FurnitureMaker {
    private final String name;

    FurnitureMaker(String name) {
        this.name = name;
    }

    Chair makeChair() {
        List<Wood> woods = getWoods(Chair.WOOD_AMOUNT);

        Hammer hammer = getHammer();
        return hammer.makeChair(woods);
    }

    Desk makeDesk() {
        List<Wood> woods = getWoods(Desk.WOOD_AMOUNT);

        Hammer hammer = getHammer();
        return hammer.makeDesk(woods);
    }

    private List<Wood> getWoods(int woodAmount) {
        List<Wood> woods = new ArrayList<>();
        for (int i = 0; i < woodAmount; i++) {
            woods.add(new CheapWood());
        }
        return woods;
    }

    private Hammer getHammer() {
        return new IronHammer();
    }
}
```

makeChair(), makeDesk() 함수들에서 추상화된 Wood와 Hammer를 사용하고 있다는 걸 보실 수 있습니다.

위에서 "추상화했기 때문에 바꿀 수 있다"라고 했는데요. 이 코드를 보면 실제로 바꿀 수 있습니까? 뭐... 가능은 합니다만, 추상화의 가치를 살리지 못했습니다. 다시 말하면, 바꿀 순 있지만 쉽게 바꿀 순 없습니다. 결론부터 말하자면, new CheapWood(), new IronHammer() 때문입니다. 다른 Wood, Hammer를 쓰고 싶으면 FurnitureMaker의 코드를 직접 바꿔야 하기 때문에 수정을 어렵게 만듭니다. 개발적으로 접근하면 설명하기가 조금 어려워서, 실생활에 빗대어 설명해보겠습니다. 만약에 좋은 Wood를 가져오는 게 FurnitureMaker의 핵심역량이라고 해봅시다. 그러면 FurnitureMaker는 이걸 비밀로 하고 싶겠죠. 하지만 실상은 어떤가요? FurnitureMaker를 아는 사람(FurnitureMaker의 코드를 볼 수 있는 사람)은 이 사람이 어떻게 Wood를 가져오는 지, 어떤 Hammer를 가져오는 지 알 수 있죠. 이런 상태론 FurnitureMaker는 다른 사람이 노하우를 베껴서 쫄딱 망할 것입니다. 또한 너무 투명해서 사기를 치고 싶어도 칠 수도 없어요. 예를 들어, 나쁜 사람한테는 싼 목재를 가져다가 비싼 의자라고 속여서 팔고, 착한 사람한테는 비싼 목재를 쓴다고 합시다. Chair를 사는 사람이 Furniture가 어떤 Wood를 쓰는 지, 어떤 Hammer를 쓰는 지 알기 때문에 속일 수가 없죠. 설령 FurnitureMaker가 공평하고, 워낙 투명해서 사기치지 않는다고 해도 여전히 문제입니다. 부자가 와서, "전 비싼 목재를 써주세요"라고 하면, FurnitureMaker는 "잠시만요! 지금 getWood 코드에서 싼 목재를 가져오고 있어서요. 금방 바꿔줄게요!" 라고 해야 할 것입니다. 다음 손님이 싼 목재를 원한다면? 또 코드를 바꿔줘야 할 것입니다. 즉, 구현체를 바꾸기 위해 FurnitureMaker.java에서 코드를 수정해야 한다는 것 자체로 변화에 대응하기가 어렵게 됩니다. getWood, getHammer와 같은 코드를 쉽게 바꿀 수 있는 방법이 없을까요?

## 코드 개선 2: 상속을 통한 확장

물론 있습니다. FurnitureMaker의 코드를 분리하면 됩니다. 여기서는 FurnitureMaker를 분리한다는 것을 "사장이 혼자 가게를 운영하다가, 친구를 영입하는 것"이라고 생각하면 이해하기 쉬울 것 같습니다. 고객에 따라 바뀌는 함수들을 추상 메소드로 만듭니다. ***Commit - feat: 4 - 상속을 통한 확장***

```java
abstract class FurnitureMaker {
  
		...
      
    public abstract List<Wood> getWoods(int woodAmount);

    public abstract Hammer getHammer();
}
```

이제 가구를 사는 고객은 Furniture.java 코드만 봐서는 어떤 Wood를 쓰는 지 알 수 없게 됐기 때문에, 핵심역량을 감출 수 있을 뿐 아니라 고객에 따라 쉽게 대응할 수도 있게 됩니다. 부자가 왔다? 사장이 상대합니다. 

```java
public class FurnitureMakerForRich extends FurnitureMaker {
    FurnitureMakerForRich(String name) {
        super(name);
    }

    @Override
    public List<Wood> getWoods(int woodAmount) {
        List<Wood> woods = new ArrayList<>();
        for (int i = 0; i < woodAmount; i++) {
            woods.add(new ExpensiveWood());
        }
        return woods;
    }

    @Override
    public Hammer getHammer() {
        return new IronHammer();
    }
}
```

돈 없는 사람이 왔다? 영입한 친구가 상대합니다.

```java
public class FurnitureMakerForPoor extends FurnitureMaker {
    FurnitureMakerForPoor(String name) {
        super(name);
    }

    @Override
    public List<Wood> getWoods(int woodAmount) {
        List<Wood> woods = new ArrayList<>();
        for (int i = 0; i < woodAmount; i++) {
            woods.add(new CheapWood());
        }
        return woods;
    }

    @Override
    public Hammer getHammer() {
        return new IronHammer();
    }
}
```

기존에는 같은 클래스 내에 다른 메소드로 분리됐던 getWood, getHammer라는 관심을 상속을 통해 서브클래스로 분리한 것입니다. 덕분에 FurnitureMaker의 핵심 기능이라고 할 수 있는 makeChair(), makeDesk(), 즉 실제로 가구를 만드는 코드(FurnitureMaker)와 어떤 준비물(Wood, Hammer 등)을 어떻게 가져올 지 등의 관심사를 다루는 클래스(FurnitureMakerForPoor, FurnitureMakerForRich)를 분리했습니다. 이제 FurnitureMaker의 코드는 수정하지 않아도 Wood를 바꿀 수 있게 됐습니다. 새로운 니즈를 발견하여 특별한 고객이 오더라도, FurnitureMaker를 상속하여 그 고객에 맞추면 됩니다(즉 확장할 수 있습니다).

하지만 이러한 상속 방식에는 단점이 있습니다. 첫째로, 만약 하위클래스가 이미 다른 클래스를 상속하고 있다면, 원하는 수퍼클래스를 상속하기 어려울 수 있습니다. 예를 들어, FurnitureMakerForPoor가 가구 뿐 아니라 부업으로 컴퓨터도 만들어준다고 합시다. 하지만 FurnitureMakerForPoor는 FurnitureMaker와 ComputerMaker 둘 다 상속할 수 없습니다. 둘째로 상속을 하면 상하위 클래스의 관계가 너무 밀접해진다는 것입니다. 수퍼클래스가 수정되면 하위클래스는 억지로 따라 변하게 되고, 하위클래스를 사용할 때 수퍼클래스가 내가 원하는대로 기능구현했는 지 확실히 알아야 합니다. 즉, 캡슐화가 깨지게 됩니다. 셋째로 수퍼클래스를 상속하지 않는, 서브클래스와 유사한 성격의 코드에는 잘 만들어놓은 메소드를 재활용할 수 없다는 것도 문제입니다. 예를 들어, ComputerMakerForPoor는 FurnitureMaker를 상속하지 않겠죠. 하지만 ComputerMaker도 Hammer가 필요하다고 합니다. 이 때 기존에 구현해놓은 getHammer를 사용할 수 없고, 새롭게 getHammer를 정의해야 해서 중복이 생기겠죠. 결국 변화에 대응하기 위해 상속을 사용한 게 불편합니다...

## 코드 개선 3: 클래스의 분리 - 인터페이스 활용

지금까지 성격이 다른 관심사를 분리하는 작업을 진행했습니다. 처음에는 메소드를 만들어서 분리했고, 다음에는 상하위 클래스로 분리했습니다. 이번에는 아예 상속관계도 아닌 완전히 독립적힌 클래스로 만들어보겠습니다. 방법은 getWood(), getHammer()을 서브클래스가 아니라, 아예 별도의 클래스에 담는 것입니다. 준비물을 영어로 번역해보니 Material이라 MaterialMaker라고 하고, get보다는 make이 더 적합한 함수명인 것 같아 함수명도 바꿔봤습니다. ***Commit - feat: 5 - 상속을 하지 않는, 독립된 클래스로 기능 분리***

```java
public class MaterialMakerForPoor {
    public List<Wood> makeWoods(int woodAmount) {
        List<Wood> woods = new ArrayList<>();
        for (int i = 0; i < woodAmount; i++) {
            woods.add(new CheapWood());
        }
        return woods;
    }
    
    public Hammer makeHammer() {
        return new IronHammer();
    }
}
```

그러면 FurnitureMaker는 더이상 추상메소드를 가질 필요가 없게 됩니다.

```java
class FurnitureMaker {
    private final String name;

    FurnitureMaker(String name) {
        this.name = name;
    }

    Chair makeChair() {
        MaterialMakerForPoor materialMakerForPoor = new MaterialMakerForPoor();
        List<Wood> woods = materialMakerForPoor.getWoods(Chair.WOOD_AMOUNT);
        Hammer hammer = materialMakerForPoor.getHammer();
        return hammer.makeChair(woods);
    }

    Desk makeDesk() {
        MaterialMakerForPoor materialMakerForPoor = new MaterialMakerForPoor();
        List<Wood> woods = materialMakerForPoor.getWoods(Desk.WOOD_AMOUNT);
        Hammer hammer = materialMakerForPoor.getHammer();
        return hammer.makeDesk(woods);
    }
}
```

이제 FurnitureMaker는 상속 없이도 getWood(), getHammer()의 구현을 몰라도 됩니다! 그런데 여전히 여러 문제가 있습니다. 먼저, new MaterialMakerForPoor() 부분이 중복되는 게 마음에 들지 않습니다. 중복이 마음에 안드는 것 뿐 아니라, 상식적으로도 이상합니다. 사실 MaterialMakerForPoor를 적용하기 전에도 상식적으로 이상했는데요. 맨 처음 코드를 다시 봐봅시다. 

```java
// FurnitureMaker.java
Chair makeChair() {
    List<Wood> woods = new ArrayList<>();
    for (int i = 0; i < Chair.WOOD_AMOUNT; i++) {
        woods.add(new Wood());
    }

    IronHammer ironHammer = new IronHammer();
    return ironHammer.makeChair(woods);
}
```

이 코드를 현실 세계에서 이해해보면, 누군가 FurnitureMaker에게 의자를 만들어달라고 부탁했을 때 FurnitureMaker는 다음과 같이 반응할 것입니다. "잠깐만! 의자를 만들기 전에 목재부터 만들고. 잠깐만! 필요한 목재는 만들었지만 망치도 만들어야 해". 이상하지 않나요? 의자 만들어달라고 했더니 망치부터 만들어야 한답니다. 사실 목재를 준비하고, 망치를 준비하는 게 FurnitureMaker의 최대 관심사라고 하기엔 무리가 있죠. 목재와 망치를 가지고, "의자를 만드는 것"이 FurnitureMaker의 관심사입니다. MaterialMakerForPoor를 적용해도 마찬가지입니다. 의자를 만들어달라고 했더니, "잠깐만! MaterialMakerForPoor부터 만들고" 라고 하는 게 이상합니다. 파라미터로 받도록 수정해보도록 하겠습니다. ***Commit - feat: 6 - 의존 대상(MaterialMakerForPoor)을 파라미터로 받음***

```java
// FurnitureMaker.java
Chair makeChair(MaterialMakerForPoor materialMakerForPoor) {
    List<Wood> woods = materialMakerForPoor.makeWoods(Chair.WOOD_AMOUNT);
    Hammer hammer = materialMakerForPoor.makeHammer();
    return hammer.makeChair(woods);
}
```

아까보다 낫네요. 이제 "잠깐만! MaterialMakerForPoor부터 만들고"의 과정 없이, MaterialMakerForPoor를 받으면, MaterialMakerForPoor가 알아서 준비물을 만들어줍니다. FurnitureMaker는 준비물로 의자를 만들기만 하면 됩니다. 그런데 여전히 이상합니다. Desk를 만들어달라고 했더니 또 MaterialForPoor를 전달해달라고 합니다.

```java
// FurnitureMaker.java
Desk makeDesk(MaterialMakerForPoor materialMakerForPoor) {
    List<Wood> woods = materialMakerForPoor.makeWoods(Desk.WOOD_AMOUNT);
    Hammer hammer = materialMakerForPoor.makeHammer();
    return hammer.makeDesk(woods);
}
```

FurnitureMaker에게 일 시키기가 쉽지 않네요. "아까 줬잖아? 왜 또 필요해?" 라고 물었더니 "그거 버렸는데?"라는 답이 옵니다. 현실 세계라면 어이가 없는 상황이죠. 현실세계에선 FurnitureMaker가 이미 준비물들을 가지고 있겠죠. 그렇게 바꿔보도록 하겠습니다. ***Commit - feat: 7 - 의존 대상(MaterialMakerForPoor)을 생성자에서 받음***

```java
class FurnitureMaker {
    private final String name;
    private final MaterialMakerForPoor materialMakerForPoor;

    FurnitureMaker(String name, MaterialMakerForPoor materialMakerForPoor) {
        this.name = name;
        this.materialMakerForPoor = materialMakerForPoor;
    }

    Chair makeChair() {
        List<Wood> woods = materialMakerForPoor.makeWoods(Chair.WOOD_AMOUNT);
        Hammer hammer = materialMakerForPoor.makeHammer();
        return hammer.makeChair(woods);
    }

    Desk makeDesk() {
        List<Wood> woods = materialMakerForPoor.makeWoods(Desk.WOOD_AMOUNT);
        Hammer hammer = materialMakerForPoor.makeHammer();
        return hammer.makeDesk(woods);
    }
}
```

훨씬 낫네요. 그런데 상속했을 땐 없던 문제가 생겼습니다. 상속으로, "바꿀 수 있게" 하지 않았습니까? 그런데 코드를 보면 FurnitureMaker는 이제 부자들은 상대도 못합니다. 돈 없는 사람들만 상대할 수 있게 코딩돼있고, 만약에 바꾸려면 또 "앗 잠깐만요, 현재 Wood가 싼 거라서요. 금방 코드 바꿔드릴게요" 와 같은 상황이 생깁니다. 이러한 상황을 해결하고, 전처럼 자유롭게 확장하려면 두 가지 문제를 해결해야 합니다. 

첫째는 바꿀 수 있는 것들의 메소드 명이 다르면 수정해야 할 코드가 엄청나게 많아진다는 것입니다. 만약에 부자를 상대하기 위해 MaterialForRich를 다음과 같이 구현하면 어떤 상황이 펼쳐질까요?

```java
class MaterialMakerFoorRich {
    List<Wood> createWoods(int woodAmount) {
        List<Wood> woods = new ArrayList<>();
        for (int i = 0; i < woodAmount; i++) {
            woods.add(new ExpensiveWood());
        }
        return woods;
    }

    Hammer createHammer() {
        return new IronHammer();
    }
}
```

여기에서 주목해야 할 건 함수명입니다. create나 make나 의미는 유사하지만, 어쨌든 다르죠. FurnitureMaker가 부자를 상대하기 위해 꾸역꾸역 MaterialForPoor를 MaterialForRich로 수정했다고 칩시다.

```java
class FurnitureMaker {
    private final String name;
    private final MaterialMakerForRich materialMakerForRich;

    FurnitureMaker(String name) {
        this.name = name;
        this.materialMakerForRich = new MaterialMakerForRich();
    }

    Chair makeChair() {
        List<Wood> woods = materialMakerForRich.makeWoods(Chair.WOOD_AMOUNT);
        Hammer hammer = materialMakerForRich.makeHammer();
        return hammer.makeChair(woods);
    }

    Desk makeDesk() {
        List<Wood> woods = materialMakerForRich.makeWoods(Desk.WOOD_AMOUNT);
        Hammer hammer = materialMakerForRich.makeHammer();
        return hammer.makeDesk(woods);
    }
}
```

바꿨지만, 함수명이 달라 빨간 쭉 쫙 뜹니다. 따라서 비슷한 것들의 함수명을 통일해야 이러한 문제를 해결할 수 있습니다. 

둘째는 FurnitureMaker가 MaterialMakerForPoor를 구체적으로 알아야 한다는 것입니다. 구체적으로 알기 때문에 부자를 상대하기 위해선 FurnitureMaker의 코드를 직접 수정해야만 하죠. 근본적으로, 의존대상의 구현체를 구체적으로 안다는 게 문제입니다.

이러한 두 문제를 해결할 수 있는 방법이 있으니, 바로 인터페이스를 적용하는 것입니다. 인터페이스를 활용하여 두 클래스가 서로 긴밀하게 연결되어 있지 않도록 중간에 추상적인 느슨한 연결고리를 만들어주는 것입니다. 인터페이스는 어떤 일을 하겠다는 기능만 정의해놓고, 자신을 구현한 클래스에 대한 구체적인 정보는 모두 감춰 버립니다. 결국 오브젝트를 만들려면 구체적인 클래스 하나를 선택해야겠지만, 인터페이스로 추상화해놓은 최소한의 통로를 통해 인터페이스를 활용하는 쪽에서는 구현 클래스가 무엇인지 몰라도 됩니다. 이러한 인터페이스의 장점을 생각해보면 두 번째 문제(구현체를 구체적으로 안다는 것)을 해결할 수 있을 것으로 보입니다. 또한, 인터페이스는 동일한 이름의 함수를 구현하는 것을 강제하기 때문에, 첫번째 문제(함수명이 다르면 수정해야 할 코드가 많아진다)도 해결할 수 있습니다. 즉, 실제 구현 클래스를 바꿔도 신경쓰지 않아도 됩니다. 인터페이스를 추출해보겠습니다. ***Commit - feat: 8 - 인터페이스 적용***

```java
public interface MaterialMaker {
    List<Wood> makeWoods(int woodAmount);

    Hammer makeHammer();
}
```

당연히 구현체들은 이 인터페이스를 implements 해야겠죠. 인터페이스를 FunitureMaker에 적용해보겠습니다.

```java
class FurnitureMaker {
    private final String name;
    private final MaterialMaker material;

    FurnitureMaker(String name) {
        this.name = name;
        this.material = new MaterialMakerForPoor();
    }

    Chair makeChair() {
        List<Wood> woods = material.makeWoods(Chair.WOOD_AMOUNT);
        Hammer hammer = material.makeHammer();
        return hammer.makeChair(woods);
    }

    Desk makeDesk() {
        List<Wood> woods = material.makeWoods(Desk.WOOD_AMOUNT);
        Hammer hammer = material.makeHammer();
        return hammer.makeDesk(woods);
    }
}
```

인터페이스를 통해 첫번째 문제(구현체마다 함수명이 다르면 많은 수정이 필요하다는 것)를 바로 해결할 수 있게됐습니다. `this.material = new MaterialMakerForPoor();` 를 `this.material = new MaterialMakerForRich();` 로 바꾸기만 하면 부자를 위한 코드가 됩니다. 이전에는 "잠깐만요! 비싼 목재를 쓰려면 코드를 수정해야 하는데요. 그런데 몇 줄을 수정해야하지..." 였다면, 이제는 "잠깐만요! 비싼 목재를 쓰기 위해 코드 한 줄만 바꾸면 됩니다!" 라고 말할 수 있게 됐습니다. 

## 코드 개선 4: 설계도로서의 팩토리

하지만 인터페이스를 쓴다고 해서 두번째 문제(의존 대상을 구체적으로 안다는 것)를 해결하는 것은 아닙니다. 바로 `this.material = new MaterialMakerForPoor();` 이 코드 때문입니다. makeChair(), makeDesk() 입장에서는 material이 어떤 녀석인지 구체적으로 알지 못하죠. 하지만 FurnitureMaker는 생성자에서 new MaterialMakerForPoor() 를 호출하기 때문에 구현체를 구체적으로 알고 있습니다. 즉 두 개의 관심사, (1) 준비물 만들기 (2) 가구 만들기를 잘 분리했는데도 두 클래스는 직접적인 관련이 있습니다. 이 때문에(다른 관심사항이 공존하기 때문에) FurnitureMaker는 자유롭게 확장하지 못하게 되죠. `new 객체` 하는 것만으로도 독립적인 관심사라는 것입니다. 이 한 줄의 코드가 FunitureMaker와 MaterialMaker의 관계를 설정합니다. 이 코드를 분리하지 않으면 FurnitureMaker는 결코 독립적으로 확장 가능한 클래스가 될 수 없습니다. 이 "관계설정"이라는 관심을 어디에 분리할까요? 바로 "FurnitureMaker의 클라이언트"가 이 관심을 맡기에 적절합니다. 클라이언트라는 게 특별한 게 아니라, FurnitureMaker의 함수가 어디에서든 호출되지 않겠습니까? 그 호출하는 쪽을 클라이언트라고 합니다. 예를 들어, FurnitureMaker.java를 테스트하는 FurnitureMakerTest.java가 FurnitureMaker의 함수를 호출하지 않습니까? FurnitureMakerTest도 FurnitureMaker의 클라이언트라고 할 수 있습니다. 관계설정의 책임을 클라이언트에 넘겨버리면, FurnitureMaker는 그제서야 MaterialMaker로부터 자유로워질 수 있습니다. 다음과 같이 되겠죠. ***Commit - feat: 9 - 관계설정 책임의 분리***

```java
// FurnitureMaker.java
FurnitureMaker(String name, MaterialMaker materialMaker) {
    this.name = name;
    this.materialMaker = materialMaker;
}
```

FurnitureMaker의 클라이언트인 테스트코드를 보겠습니다.

```java
// FurnitureMakerTest.java
@Test
void makeChair() {
    MaterialMaker materialMaker = new MaterialMakerForPoor();
    FurnitureMaker furnitureMaker = new FurnitureMaker("root", materialMaker);
    Chair chair = furnitureMaker.makeChair();
    assertThat(chair).isNotNull();
}
```

`MaterialMaker materialMaker = new MaterialMakerForPoor();`가 여기에 들어있군요! 이제 부자가 와도, 돈 없는 사람이 와도 FurnitureMaker는 전혀 수정하지 않아도 대응할 수 있게 됐습니다. 즉, 기능 확장에는 열려있고, 코드 변경에는 닫혀있게 됩니다.

현재 FurnitureMaker의 클라이언트는 FurnitureMakerTest 입니다. 그런데 FurnitureMakerTest는 테스트 하려고 만든 코드 아닙니까? 테스트라는 관심 말고 관계설정의 관심을 맡게 됐으니, 테스트코드 입장에서는 당황스럽습니다. 관계설정만 담당해주는 클래스를 만들어봅시다. ***Commit - feat: 10 - 팩토리 클래스***

```java
class FurnitureMakerFactory {
    FurnitureMaker createFurnitureMaker() {
        MaterialMaker materialMaker = new MaterialMakerForPoor();
        return new FurnitureMaker("root", materialMaker);
    }
}
```

팩토리는 FurnitureMaker를 어떻게 만들 지, 어떻게 준비시킬 지를 결정합니다. 즉 오브젝트들을 구성하고 그 관계를 정의합니다. 이런 작업이 애플리케이션 전체에 걸쳐 일어난다면 객체들의 의존관계에 대한 설계도와 같은 역할을 할 것입니다. 팩토리 클래스의 가치는 여럿 있겠지만 그 중 으뜸은 애플리케이션의 로직을 담당하는 컴포넌트 역할의 오브젝트(FurnitureMaker, MaterialMaker)와 애플리케이션의 구조를 결정하는 오브젝트(FurnitureMakerFactory)를 분리했다는 것입니다. 어쩌면 "팩토리를 쓰더라도, 돈 없는 사람을 위한 코드에서 부자를 위한 코드로 바꾸려면 팩토리 내의 `MaterialMaker materialMaker = new MaterialMakerForPoor();`를 `MaterialMaker materialMaker = new MaterialMakerForRich();`로 바꿔야 하지 않습니까? 결국 코드 수정을 하긴 해야하네요! 이전에도 한 줄, 지금도 한 줄 수정하는데 뭐가 개선된거죠?"라고 생각하실 수도 있을 것 같습니다. 중요한 건 FurnitureMaker의 입장입니다. FurnitureMaker는 이제 MaterialMakerForPoor든, MaterialMakerForRich든 상관이 없어졌어요. 자기 관심인 "준비물로 가구를 만든다"에 집중할 수 있는 것이죠. FurnitureMakerFactory입장에서도 괜찮습니다. 이제까지 FurnitureMaker가 억울했던 이유는 "내가 별로 관심도 없는 것 때문에 바뀌어야하나?" 였습니다. 근데 FurnitureMakerFactory는 관계설정을 해주는 게 자기 일이예요. 억울할 게 없죠. 그리고 코드 수정량도 다릅니다. 현재는 둘 다 한 줄만 바꿔도 되지만, MaterialMaker를 다른 애들도 쓴다면? 예를 들어, ComputerMaker도 MaterialMaker를 사용하고 있다고 가정하면 얘기가 달라집니다. 팩토리를 쓴다면 한 줄로 될 것을, 쓰지 않는다면 두 줄을 수정해야겠죠.

## 제어의 역전

제어의 역전이란 프로그램의 제어 흐름 구조가 뒤바뀌는 것입니다. 일반적으로 프로그램의 흐름은 main() 메소드와 같이 프로그램이 시작되는 지점에서 다음에 사용할 오브젝트를 결정하고, 결정한 오브젝트를 생성하고, 만들어진 오브젝트에 있는 메소드를 호출하고, 그 오브젝트 메소드 안에서 다음에 사용할 것을 결정하고, 호출히는 식의 작업이 반복됩니다. 제어가 역전되지 않는다면, 모든 오브젝트가 능동적으로 자신이 사용할 클래스를 결정하고, 언제 어떻게 그 오브젝트를 만들 지 스스로 관장합니다. 모든 종류의 작업은 사용하는 쪽에서 제어하는 구조입니다. 반대로 제어의 역전은 이러한 제어 흐름을 거꾸로 뒤집습니다. 제어의 역전에서는 오브젝트가 자신이 사용할 오브젝트를 스스로 선택하지 않습니다. 생성하지도 않습니다. 또 자신이 어떻게 만들어지고 어디서 사용되는지를 알 수 없습니다. 모든 제어 권한을 자신이 아닌 다른 대상에게 위임하기 때문입니다. main() 을 제외하면 모든 오브젝트는 위임받은 제어 권한을 갖는 특별한 오브젝트에 의해 결정되고 만들어집니다. 상속을 활용한 확장도 제어의 역전이 적용된 예입니다. 서브클래스는 필요한 메소드를 구현하지만, 그 메소드가 언제 어떻게 사용될 지 자신은 모릅니다. 서브클래스에서 결정되는 것이 아니라, 기능만 구현해놓으면 수퍼클래스의 메소드가 서브클래스의 함수를 호출하게 됩니다. 프레임워크도 제어의 역전이 적용된 기술입니다. 프레임워크는 단지 미리 만들어둔 반제품이나, 확장해서 사용할 수 있도록 준비된 추상 라이브러리 집합이 아닙니다. 라이브러리를 사용하는 애플리케이션 코드는 애플리케이션 흐름을 직접 제어합니다. 단지 동작하는 중 필요한 기능이 있을 때 능동적으로 라이브러리를 사용할 뿐입니다. 반면 프레임워크는 거꾸로 애플리케이션 코드가 프레임워크에 의해 사용됩니다. 프레임워크가 흐름을 주도하는 중에 개발자가 만든 애플리케이션 코드를 사용하는 방식입니다. 프레임워크는 분명한 제어의 역전 개념이 적용되어 있어야 합니다. 예제 코드도 마찬가지입니다. 원래 FurnitureMaker가 MaterialMaker의 구현체를 결정하고, 생성했죠. 그런데 개선 후에는 FurnitureMakerFactory가 제어권을 가집니다. 즉 FurnitureMaker는 자신이 어떤 클래스를 사용할 지도 모르는, 수동적인 존재가 됐습니다. MaterialMaker의 생성 제어권한 뿐 아니라, FurnitureMaker 자기 자신의 생성 권한도 FurnitureMakerFactory에게 맡깁니다. 제어의 역전에서는 예제 코드에서의 팩토리 클래스와 같이 애플리케이션 컴포넌트의 생성과 관계설정, 사용, 생명주기 관리 등을 관장하는 존재가 필요합니다. 

스프링의 핵심가치 세 가지 중 하나가 IoC입니다. 개발자가 IoC를 쉽게, 잘 적용할 수 있도록 스프링이 도와준다는 뜻입니다. 스프링 내부 구현에도 IoC가 적용되어 있기도 합니다. 조금 더 쉽게 말하자면, 스프링이 FurnitureMakerFactory의 역할, 즉 객체 간 관계 설정의 책임을 도맡아줍니다.

## 의존관계 주입

의존하고 있다는 건 무슨 의미일까요? A가 B에 의존한다는 건, B가 변하면 그것이 A에 영향을 미친다는 뜻입니다. 이제까지 공부한 IoC라는 용어는 매우 느슨하게 정의돼서 폭넓게 사용됩니다. 때문에 스프링을 IoC 컨테이너라고만 해서는 스프링이 제공하는 기능의 특징을 명확하게 설명하지 못합니다. 그래서 새로운 용어를 만드는 데 탁월한 재능이 있는 몇몇 사람의 제안으로 스프링이 제공하는 IoC 방식을 의존관계 주입이라는, 스프링 IoC의 핵심을 짚어주며 좀 더 의도가 명확히 드러나는 이름을 사용하기 시작했습니다. 스프링 IoC 기능의 대표적인 동작원리는 주로 의존관계 주입이라고 불립니다. 물론 스프링이 컨테이너이고 프레임워크이니 기본적인 동작원리가 모두 IoC 방식이라고 할 수 있지만, 스프링이 여타 프레임워크와 차별화돼서 제공해주는 기능은 의존관계 주입이라는 새로운 용어를 사용할 때 분명하게 드러납니다.

Dependency Injection은 여러개의 말로 번역되는데, 그 중 가장 흔히 사용되는 용어가 의존성 주입입니다. 하지만 의존성이라는 말은 DI의 의미가 무엇인지 잘 드러내주지 못합니다. DI는 오브젝트 레퍼런스를 외부로부터 제공(주입)받고 이를 통해 여타 오브젝트와 다이내믹하게 의존관계가 만들어지는 것이 핵심입니다. 이러한 면에서 의존관계 설정이라는 번역도 나쁘지 않습니다.

정리하면 의존관계 주입이란 다음과 같은 세 가지 조건을 충족히는 작업을 말합니다.

1. 클래스 모델이나 코드에는 런타임 시점의 의존관계가 드러나지 않습니다. 그러기 위해서는 인터페이스에만 의존하고 있어야 합니다.
2. 런타임 시점의 의존관계는 컨테이너나 팩토리 같은 제 3의 존재가 결정합니다.
3. 의존관계는 사용할 오브젝트에 대한 레퍼런스를 외부에서 제공(주입)해줌으로써 만들어집니다.

의존관계 주입의 핵심은 설계 시점에는 알지 못했던 두 오브젝트의 관계를 맺도록 도외주는 제 3의 존재가 있다는 것입니다. 제3의 존재는 바로 관계설정 책임을 가진 코드를 분리해서 만들어진 오브젝트라고 볼 수 있습니다.

DI는 자신이 사용할 오브젝트에 대한 선택과 생성 제어권을 외부로 넘기고 자신은 수동적으로 주입받은 오브젝트를 사용한다는 점에서 IoC 의 개념에 잘 들어맞습니다. 스프링 컨테이너의 IoC는 주로 의존관계 주입 또는 DI라는 데 초점이 맞춰져 있습니다. 그래서 스프링을 IoC 컨테이너 외에도 DI 컨테이너 또는 DI 프레임워크라고 부릅니다.

DI의 동작방식은 이름 그대로 외부로부터의 주입입니다. 하지만 단지 외부에서 파라미터로 오브젝트를 넘겨줬다고 해서, 즉 주입해줬다고 해서 다 DI가 아니라는 점을 주의해야 합니다. 주입받는 메소드 파라미터가 이미 특정 클래스 타입으로 고정되어 있다면 DI가 일어날 수 없습니다. DI에서 말하는 주입은 다이내믹하게 구현 클래스를 결정해서 제공받을 수 있도록 인터페이스 타입의 파라미터를 통해 이뤄져야 합니다.

지금까지 기술한 DI 기술의 장점을 요약하면, 코드에는 런타임 클래스에 대한 의존관계가 나타나지 않고, 인터페이스를 통해 결합도가 낮은 코드를 만드므로, 다른 책임을 가진 사용 의존관계에 있는 대상이 바뀌거나 변경되더라도 자신은 영향을 받지 않으며, 변경을 통한 다양한 확장 방법에는 자유롭는 것입니다.