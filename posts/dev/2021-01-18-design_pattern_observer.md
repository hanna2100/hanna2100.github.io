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

# 2. 옵저버 패턴
---
## # 날씨 앱🌞을 생각해 봅시다
개발자를 꿈꾸는 당신은 간단한 날씨 앱을 만들고자 한다. 편리하게도, 기상청에서 오픈소스로 WeatherData 객체를 바로 제공한다고 생각해봅시다. 이 객체는 현재 기상 조건(기온, 습도, 기압)을 바로 가져오는 객체입니다.

![](https://images.velog.io/images/hanna2100/post/4f906070-4233-40d2-aeac-9e711e253283/20210118_223745.png)

**1. 현재 날씨
2. 날씨 예보
3. 기상 통계** 

당신은 이 앱에서 3가지 화면을 기획했습니다. 이 화면들은 모두 WeatherData 객체에서 최신 데이터로 업데이트 될 때마다 실시간으로 갱신됩니다. 자, 이 어플리케이션을 어떤식으로 구현해야 할까요?

---
### 1. Bad Case를 살펴봅시다

```java
class WeatherData {
	getTemperature()
	getHumidity()
	getPressure()
	measurementsChanged() // 기상 관측값이 갱신되면 해당 메소드가 호출됨
}
```
기상청에서 제공하는 WatherData 클래스를 살펴보니 `getTemperature()`, `getHumidity()`, `getPressure()`와 같이 온도, 습도, 기압을 가져오는 메소드들을 제공합니다. 이 클래스가 이 데이터를 어떻게 가져오는지는 중요하지 않습니다. 중요한 것은 WatherData클래스가 해당 데이터들을 제공한다는 것, 그리고 `measurementsChanged()` 메소드가 기상 관측값이 갱신되면 호출된다는 것이죠.

**당신은 `measurementsChanged()` 메소드를 통해 3가지 화면을 구현하면 됩니다.**


```java
class WeatherData {
// 인스턴스 변수 선언
	public void measurementsChanged() {
		float temp = getTemperature();
		float humidity = getHumidity();
		float pressure = getPressure();
    
		currentWeatherDisplay.update(temp, humidity, pressure);
		forecastWeatherDisplay.update(temp, humidity, pressure);
		staticsticDisplay.update(temp, humidity, pressure);
	}
}
```

자, 이렇게 구현해보는 건 어떨까요? `getTemperature()` 메소드들을 이용하여 기온, 습도, 기압을 가져오고 각 화면단에 해당 변수들을 전달하여 `update()` 하고 있네요!


**이 코드는 다음과 같은 문제가 있습니다!**

1. `currentWeatherDisplay.update(..`
`forecastWeatherDisplay.update(..`
`staticsticDisplay.update(..` 와 같이 구체적인 구현에 맞춰서 코딩했기 때문에, 다른 화면을 추가하고자 한다면 `measurementsChanged()` 메소드를 고치지 않고서는 새로운 화면을 추가할 수 없습니다.

2. 따라서 바뀔 수 있는 부분(각 화면 단)을 캡슐화 해야합니다. 즉, 이 코드는 캡슐화 되어있지 않다고 볼 수 있겠네요.

---
### 2. 옵저버 패턴을 만나봅시다
혹시 좋아하는 유튜버를 구독해보신 적 있으신가요?
만약 그렇다면, 옵저버패턴을 쉽게 이해할 수 있습니다. 유튜버는 주제(subject), 구독자는(Observer)로 볼 수 있습니다.

![](https://images.velog.io/images/hanna2100/post/6a8c2db1-1f2a-49f5-919d-d046f4643aa8/20210118_232543.png)

> 옵저버 패턴(Observer Pattern)에서는 한 객체의 상태가 바뀌면 그 객체에 의존하는 다른 객체들한테 연락이 가고, 자동으로 내용이 갱신되는 방식으로 일대다(one-to-many) 의존성을 정의한다.

옵저버 패턴을 구현하는 방법에는 여러가지가 있지만, 대부분 주제(Subject)인터페이스와 옵저버(Observer) 인터페이스가 들어있는 클래스 디자인으로 합니다.

**주제 인터페이스와 구현체**
```java
interface Subject {
	registerObserver() // 옵저버 등록
	removeObserver() // 옵저버 삭제
	notifyObserver() // 옵저버에게 업데이트 알림
}

class SubjectImpl implements Subject {
	registerObserver() { ... }
	removeObserver() { ... }
	notifyObserver() { ... }

	getState() // 주제 객체는 상태를 설정하고 알기위한 겟터,셋터가 있을 수 있다.
	setState()
}
```

**옵저버 인터페이스와 구현체**
```java
interface Observer{ // 옵저버가 될 객체에서는 반드시 Observer 인터페이스를 구현해야함.
	update() // 주제의 상태가 바뀌었을때 호출됨
}

class ObserverImpl implements Observer {
	update() { 
		// 주제가 업데이트 될 때 해야하는 일
	}
}
```

옵저버에서 `update()`메소드를 통해 주제의 `state`를 전달받긴 하지만, 실제 `state`데이터의 주인은 주제(Subject)에 있습니다. 옵저버는 데이터가 변경되었을 때 주제에서 데이터를 전달해주기를 기다리는 입장이기 때문에 의존성을 가진다고 할 수 있죠. 이런 방법을 사용하면 여러 객체에서 동일한 데이터를 제어하도록 하는 것에 비해 더 깔끔한 객체지향 디자인을 만들 수 있습니다.

---
### 3. 느슨한 결합(Loose Coupling)의 위력
두 객체가 느슨하게 결합되어 있다는 것은, 그 둘이 상호작용을 하긴 하지만 서로에 대해 잘 모르는 것을 의미합니다. 옵저버 패턴에서는 주제와 옵저버가 느슨하게 결합되어 있는 객체 디자인을 제공하지요.

느슨한 결합의 장점은 다음과 같습니다.

1. 옵저버를 언제든 새로 추가, 제거할 수 있다.
2. 새로운 형식의 옵저버라 할 지라도 주제를 전혀 변경할 필요가 없다.
3. 주제와 옵저버는 서로 독립적으로 재사용 할 수 있다.
4. 주제나 옵저버가 바뀌더라도 서로에게 영향을 미치지 않는다.


> 디자인원칙
서로 상호작용을 하는 객체 사이에서는 가능하면 느슨하게 결합하는 디자인을 사용해야 한다.


Loose Coupling 디자인을 활용하면 변경사항이 생겨도 무난히 처리할 수 있는 유연한 객체지향 시스템을 구축할 수 있습니다. 객체 사이의 상호의존성을 최소화 할 수 있기 때문이죠.

