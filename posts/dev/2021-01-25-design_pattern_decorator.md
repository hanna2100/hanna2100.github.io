[**Head First Design Patterns**](https://www.hanbit.co.kr/store/books/look.php?p_code=B9860513241) 책을 보고 정리한 내용입니다. 디자인 패턴을 처음 입문하시는 분들께 추천드리고픈 책입니다.


>**[ 목차 ]**
1. 스트래티지 패턴
2. 옵저버 패턴
3. 데코레이터 패턴
4. 팩토리 패턴
5. 싱글턴 패턴
6. 커맨드 패턴
7. 어댑터 패턴 & 퍼사드 패턴
8. 템플릿 메소드 패턴
9. 이터레이터와 컴포지트 패턴
10. 스테이트 패턴
11. 프록시 패턴
12. 컴파운드 패턴

# 3. 데코레이터 패턴
---
## # 커피주문🧉 앱을 생각해봅시다

스타벅스 사이렌 오더처럼 원하는 음료를 커스텀(샷추가, 휘핑변경 등)하여 주문할 수 있는 앱을 만든다고 합시다.

### 1. Bad Case를 살펴봅시다
---
```java
abstract class Beverage {
	description // "녹차 프라푸치노"같은 음료 설명 변수
    
	getDescription()
	cost() // 추상메소드. 모든 음료는 추상클래스 Beverage를 상속받아 구현.
}

class Americano extends Beverage {
	...
	cost()
}

class CafeLatte extends Beverage {
	...
	cost()
}
```

음료를 정의하는 `Beverage`추상클래스를 만들었습니다. 모든 음료는 이 추상클래스를 상속받아 cost, description 등을 구체화합니다.
이 방식은 엄청나게 많은 서브클래스를 만든다는 단점이 있습니다. '휘핑크림'같은 옵션이 추가될 때마다 `AmericanoWithWhippingCream`, `CafeLatteWithWhippingCream` 등 의 클래스들이 음료 종류별로 생겨나겠죠. 또한 휘핑가격이 인상되기라도 한다면 휘핑이 있는 모든 클래스의 `cost()`를 수정해야합니다. 즉, 이런 방법은 매우 비효율적이죠.

**이 방법은 어떤가요?**
```java
abstract class Beverage {
	description
	milk
	soy
	whip
    
	getDescription()
	hasMilk()
	setMilk()
	hasSoy()
	setSoy()
	hasWhip()
	setWhip()
	cost() // 추가된 옵션항목(milk, soy, whip)들을 계산함. 
}

class Americano extends Beverage {
	...
	cost() // 아메리카노 음료 가격에서 수퍼클래스의 cost()를 호출하여 추가요금을 더해줌
}
```

수퍼클래스에서 옵션가격을 계산해주므로 더이상 `~WithWhippingCream`과 같은 서브클래스를 만들지 않아도 됩니다. 하지만 여전히 문제는 남아있습니다.

1. 옵션 종류가 많아지면 새로운 메소드를 추가해야하고 `cost()` 메소드는 수정돼야 합니다.
2. 홍차처럼 휘핑이 필요없는 음료도 `hasWhip()`같은 필요없는 메소드를 상속받습니다.
3. '휘핑 2번추가'와 같은 옵션선택을 할 수 없습니다.

### 2. 디자인 원칙 OCP(Open-Closed Principle)
---
![](https://images.velog.io/images/hanna2100/post/9b266f7f-6c72-416f-9572-cd808ad6de8c/20210125_234137.png)

> **디자인원칙**
클래스는 확장에 대해서는 열려 있어야 하지만 코드 변경에 대해서는 닫혀 있어야 한다.

이 말인 즉슨, 클래스를 상속받아 *동료*가 되는 건 자유지만, 그 동료로 인해 기존 멤버들에게 피해가(?) 가면 안된다는 뜻 입니다.

예를 들어, `Beverage` 클래스를 확장하여 `BubbleTea`클래스를 새로 만드는 건 자유입니다. 하지만 `BubbleTea`에 `펄추가`옵션이 필요하다고 해서, `Beverage` 클래스에 펄과 관련된 메소드들을 맘대로 추가하면, 이 코드로 인해 `Beverage`를 상속받는 다른 클래스에 버그가 생길수 도 있습니다. 즉 기존의 코드는 최대한 변경되어선 안됩니다. 이것을 **OCP 원칙**이라고 합니다.

물론 무조건 OCP를 적용하는 것이 쓸데없는 시간낭비가 될 수 도 있습니다. 결과적으로 불필요하고, 복잡하고, 이해하기 힘든 코드만 남을 수도 있습니다. 그렇기 때문에 바뀌는 부분 중에서 중요한 부분을 선별할 수 있는 객체지향적 안목이 필요합니다.

### 3. 데코레이터 패턴
---

