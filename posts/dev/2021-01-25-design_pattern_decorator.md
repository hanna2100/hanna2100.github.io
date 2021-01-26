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

`Decorator` 란 `장식가`를 뜻합니다. 무언갈 꾸며주는 역할을 하지요. 데코레이터 패턴은 객체를 래핑하는 방식으로 다른 객체를 꾸밀 수 있습니다.

![](https://images.velog.io/images/hanna2100/post/33a47902-3970-4761-ab23-b3e243c79d25/20210126_225448.png)

>
1. `프라푸치노` 인스턴스를 생성한다.
2. `녹차` 객체로 장식한다
3. `휘핑` 객체로 장식한다
4. `cost()` 메소드를 호출한다. 이때 첨가물 가격을 계산하는 일은 객체들에게 위임된다.

데코레이터 패턴으로 만든 `휘핑추가한 녹차 프라푸치노` 입니다. 여기서 `녹차`와 `휘핑`의 객체형식은 `프라푸치노`와 마찬가지로 `Beverage` 타입입니다. 따라서 모두가 `cost()`메소드를 가지고 있습니다.

위 그림에서 처럼, 래핑된 객체에게 `cost()`메소드를 호출하면 가장 안쪽의 수퍼객체까지 일이 위임됩니다. 가장 안쪽의 수퍼객체는 자신의 `cost()`메소드를 리턴하고, 그 값을 받은 래핑객체(`녹차`, `휘핑`)는 위에서 받은 값에 자신의 가격을 추가하여 `cost()`값을 리턴합니다. 


![](https://images.velog.io/images/hanna2100/post/5639e751-f86d-4221-9c90-14371d591b17/20210126_233131.png)

**데코레이터패턴**에서는 객체에 추가적인 요건을 동적으로 추가할 수 있습니다. 또한 서브클래스를 통해 기능을 확장할 수 도 있습니다. 
위 클래스 다이어그램은 데코레이터 패턴을 적용시킨 큰 그림입니다. 이 패턴에 맞춰 커피주문앱의 클래스 다이어그램을 생각해봅시다.

1. 구성
커피주문앱에서 `Beverage` 클래스가 구성에 해당됩니다. 이 구성클래스는 추상클래스일 수도 , 인터페이스일 수도 있습니다.
2. 행동구성(구체화된 구성)
커피주문앱에서 `Frappuccino`, `Americano`등이 해당됩니다. 구성요소를 나타내는 구체적인 구성클래스입니다.
3. 데코레이터
`Beverage` 클래스를 확장하여 `BeverageDecorator` 클래스를 만듭니다. 같은 상속이지만 이전의 `Bad Case`와 다른 점은 상속을 통해서 형식만 맞추는 것이지, 구체적인 행동을 물려받진 않습니다. 구체적인 행동은 `행동구성`클래스들에 정의될 테니까요. 이 `행동구성`클래스들에게서 필요한 행동만 가져와서 데코레이터패턴을 꾸미게 됩니다. 스트래지패턴에서 배웠던 **상속보다는 구성**이 여기서도 활용됩니다.
4. 행동데코레이션
커피주문앱에서 `Whip`, `Milk`등이 해당됩니다. 이 클래스들에겐 Beverage타입의 멤버변수가 있습니다.

### 4. 커피주문앱 코드 구현
---
다음 코드는 데코레이터 패턴을 적용하여 설계된 커피주문앱 코드입니다.
```java
public class Espresso extends Beverage {
  
	public Espresso() {
		description = "Espresso"; // description은 Beverage로부터 상속받음
	}
  
	public double cost() { // 가격은 커스텀을 생각치 않고 에스프레소 가격만 입력
		return 1.99;
	}
}


public class Milk extends BeverageDecorator {
	Beverage beverage; // 데코레이션 할 음료를 저장하기 위한 인스턴스 변수
    
	public Milk(Beverage beverage) {
		this.beverage = beverage; // 생성자를 통해 데코레이션 할 음료객체를 전달
	}
 
	public String getDescription() {
		return beverage.getDescription() + ", Milk"; // 음료 설명에 Milk 추가
	}
 
	public double cost() {
		return .20 + beverage.cost(); // 음료 가격에 우유요금 추가
	}
}


public class StarbuzzCoffee {
 
	public static void main(String args[]) { // 실제 주문을 합니다
    
		Beverage beverage = new Espresso(); // 아무것도 넣지 않은 에스프레소 주문
		System.out.println(beverage.getDescription() 
				+ " $" + beverage.cost());
 
		Beverage beverage2 = new DarkRoast(); // 우유2, 휘핑1 추가한 다크로스트
		beverage2 = new Milk(beverage2);
		beverage2 = new Milk(beverage2);
		beverage2 = new Whip(beverage2);
		System.out.println(beverage2.getDescription() 
				+ " $" + beverage2.cost());
 
		Beverage beverage3 = new HouseBlend(); // 두유1, 우유1, 휘핑1 추가한 하우스블랜드
		beverage3 = new Soy(beverage3);
		beverage3 = new Milk(beverage3);
		beverage3 = new Whip(beverage3);
		System.out.println(beverage3.getDescription() 
				+ " $" + beverage3.cost());
	}
}
```


### 5. 데코레이터 패턴의 단점
---
1. 특정 구성요소인지 확인한 다음 어떤 작업을 처리(ex. 하우스 블렌드 커피는 특별할인됨)하는 경우에 데코레이터 패턴을 사용하기 힘들수 있다.
2. 클래스가 많이 추가되는 경우 남들이 봤을 때 이해하기 힘든 디자인이 만들어 질 수 있다.~~(자바 IO 클래스가 그렇다)~~
3. 구성요소를 초기화하는 코드가 훨씬 복잡해진다.



