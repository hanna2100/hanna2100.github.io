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
두 객체가 느슨하게 결합되어 있다는 것은, 그 둘이 상호작용을 하긴 하지만 서로에 대해 잘 모르는 것을 의미합니다. 옵저버 패턴에서는 주제와 옵저버가 느슨하게 결합되어 있는 객체 디자인을 제공합니다.

느슨한 결합의 장점은 다음과 같습니다.

1. 옵저버를 언제든 새로 추가, 제거할 수 있다.
2. 새로운 형식의 옵저버라 할 지라도 주제를 전혀 변경할 필요가 없다.
3. 주제와 옵저버는 서로 독립적으로 재사용 할 수 있다.
4. 주제나 옵저버가 바뀌더라도 서로에게 영향을 미치지 않는다.


> 디자인원칙
서로 상호작용을 하는 객체 사이에서는 가능하면 느슨하게 결합하는 디자인을 사용해야 한다.


Loose Coupling 디자인을 활용하면 변경사항이 생겨도 무난히 처리할 수 있는 유연한 객체지향 시스템을 구축할 수 있습니다. 객체 사이의 상호의존성을 최소화 할 수 있기 때문이죠.

---
### 4. 날씨 앱 설계

**1. Subject 인터페이스를 구현하여 WeatherData를 주제객체로 만들었습니다.**
```java
interface Subject {
	registerObserver()
	removeObserver()
	notifyObserver()
}

class WeatherData implements Subject {
	registerObserver()
	removeObserver()
	notifyObserver()
    
	getTemperature()
	getHumidity()
	getPressure()
	measurementsChanged()
}
```

**2. `현재날씨, 날씨예보, 기상통계` 화면에서 구현해야 할 인터페이스를 정의합시다.**
```java
interface Observer {
	update() // 새로 갱신된 주제 데이터를 전달하는 인터페이스
}

interface DisplayElement {
	display() // 화면에 표현시키는 인터페이스
}
```

**3. 위 인터페이스를 구현한 `현재날씨, 날씨예보, 기상통계` 클래스를 만듭니다.**
```java
class CurrentWeather implements Observer, DisplayElement{
	update()
	display() { // 현재 측정 값을 화면에 표시 }
}

class ForcastWeather implements Observer, DisplayElement{
	update()
	display() { // 날씨 예보 표시 }
}

class StatisticsDisplay implements Observer, DisplayElement{
	update()
	display() { // 평균 기온, 평균 습도 등 표시 }
}
```

**4. 만약 새로운 화면 `불쾌지수` 화면을 만든다고 한다면 쉽게 추가할 수 있겠죠?**
```java
class DiscomfortDisplay implements Observer, DisplayElement{
	update()
	display() { // 불쾌지수를 화면에 표시 }
}
```

---
 ### 5. 실제 구현
 실제 구현된 자바코드는 다음과 같습니다. 별도의 설명을 추가하진 않겠습니다. 
 
 ```java
 public interface Subject {
	public void registerObserver(Observer o);
	public void removeObserver(Observer o);
	public void notifyObservers();
}

public interface Observer {
	public void update(float temp, float humidity, float pressure);
}

public interface DisplayElement {
	public void display();
}
 ```
 
 ```java
 public class WeatherData implements Subject {
	private List<Observer> observers;
	private float temperature;
	private float humidity;
	private float pressure;
	
	public WeatherData() {
		observers = new ArrayList<Observer>();
	}
	
	public void registerObserver(Observer o) {
		observers.add(o);
	}
	
	public void removeObserver(Observer o) {
		observers.remove(o);
	}
	
	public void notifyObservers() {
		for (Observer observer : observers) {
			observer.update(temperature, humidity, pressure);
		}
	}
	
	public void measurementsChanged() {
		notifyObservers();
	}
	
	public void setMeasurements(float temperature, float humidity, float pressure) {
		this.temperature = temperature;
		this.humidity = humidity;
		this.pressure = pressure;
		measurementsChanged();
	}

	public float getTemperature() {
		return temperature;
	}
	
	public float getHumidity() {
		return humidity;
	}
	
	public float getPressure() {
		return pressure;
	}

}
 ```

```java
public class CurrentConditionsDisplay implements Observer, DisplayElement {
	private float temperature;
	private float humidity;
	private WeatherData weatherData;
	
	public CurrentConditionsDisplay(WeatherData weatherData) {
		this.weatherData = weatherData;
		weatherData.registerObserver(this);
	}
	
	public void update(float temperature, float humidity, float pressure) {
		this.temperature = temperature;
		this.humidity = humidity;
		display();
	}
	
	public void display() {
		System.out.println("Current conditions: " + temperature 
			+ "F degrees and " + humidity + "% humidity");
	}
}

```

```java
public class ForecastDisplay implements Observer, DisplayElement {
	private float currentPressure = 29.92f;  
	private float lastPressure;
	private WeatherData weatherData;

	public ForecastDisplay(WeatherData weatherData) {
		this.weatherData = weatherData;
		weatherData.registerObserver(this);
	}

	public void update(float temp, float humidity, float pressure) {
        lastPressure = currentPressure;
		currentPressure = pressure;

		display();
	}

	public void display() {
		System.out.print("Forecast: ");
		if (currentPressure > lastPressure) {
			System.out.println("Improving weather on the way!");
		} else if (currentPressure == lastPressure) {
			System.out.println("More of the same");
		} else if (currentPressure < lastPressure) {
			System.out.println("Watch out for cooler, rainy weather");
		}
	}
}
```

```java
public class StatisticsDisplay implements Observer, DisplayElement {
	private float maxTemp = 0.0f;
	private float minTemp = 200;
	private float tempSum= 0.0f;
	private int numReadings;
	private WeatherData weatherData;

	public StatisticsDisplay(WeatherData weatherData) {
		this.weatherData = weatherData;
		weatherData.registerObserver(this);
	}

	public void update(float temp, float humidity, float pressure) {
		tempSum += temp;
		numReadings++;

		if (temp > maxTemp) {
			maxTemp = temp;
		}
 
		if (temp < minTemp) {
			minTemp = temp;
		}

		display();
	}

	public void display() {
		System.out.println("Avg/Max/Min temperature = " + (tempSum / numReadings)
			+ "/" + maxTemp + "/" + minTemp);
	}
}
```

**`현재날씨, 날씨예보, 기상통계` 화면을 불러오는 메인함수입니다.**
```java
public class WeatherStation {

	public static void main(String[] args) {
		WeatherData weatherData = new WeatherData();
	
		CurrentConditionsDisplay currentDisplay = 
			new CurrentConditionsDisplay(weatherData);
		StatisticsDisplay statisticsDisplay = new StatisticsDisplay(weatherData);
		ForecastDisplay forecastDisplay = new ForecastDisplay(weatherData);

		weatherData.setMeasurements(80, 65, 30.4f);
		weatherData.setMeasurements(82, 70, 29.2f);
		weatherData.setMeasurements(78, 90, 29.2f);
		
		weatherData.removeObserver(forecastDisplay);
		weatherData.setMeasurements(62, 90, 28.1f);
	}
}
```
> 위 코드에 대한 자세한 설명은 Head first Design Patterns 책 p95를 보시기 바랍니다~~(광고 아님)~~

---
### 6. 한 걸음 더 - 자바 내장 옵저버 패턴 사용하기
#### 1) 자바 내장 옵저버 라이브러리로 설계하기
지금까지 우리는 옵저버 패턴을 직접 구현해서 사용했습니다. 하지만 자바에서 자체적으로 옵저버 패턴을 지원하는 라이브러리가 있습니다. Observer 인터페이스와 Observable 클래스인데요. 이들은 우리가 만들었던 Subject와 Observer 인터페이스와 꽤 비슷하지만, 더 많은 기능을 제공합니다. 예를들면 갱신방식을 pull혹은 push로 선택 할 수 있죠.

Observer 인터페이스와 Observable 클래스로 날씨앱을 새로 디자인해봅시다!

**1. Observable 클래스를 상속하여 WeatherData를 주제객체로 만듭니다.**
```java
class Observable { // 이미 자바에서 제공하는 Observable에 다양한 메소드들이 있군요
	addObserver()
	deleteObserver()
	notifyObserver()
	setChanged()
}

class WeatherData extends Observable { 
// Observable를 상속했으므로 옵저버 관리 메소드를 직접 구현하지 않아도 됩니다.
	getTemperature()
	getHumidity()
	getPressure()
	measurementsChanged()
}
```

**2. Observer 인터페이스를 구현하여 `현재날씨, 날씨예보, 기상통계` 클래스를 만듭니다. 이 부분은 이전에 했던 방법과 똑같군요!**
```java
interface Observer { 
	update()
}

class CurrentWeather implements Observer, DisplayElement{
	update()
	display() 
}

class ForcastWeather implements Observer, DisplayElement{
	update()
	display() 
}

class StatisticsDisplay implements Observer, DisplayElement{
	update()
	display() 
}
```

#### 2) 자바 내장 옵저버 패턴 작동 방식
Observable 은 클래스이기 때문에 `extends`를 통해 `addObserver()`, `deleteObserver()`와 같은 옵저버들을 관리하는 메소드들을 상속받습니다.

**1. 옵저버를 만드는 방법**
옵저버로 등록하려는 클래스에 java.util.Observer 인터페이스를 구현하고 구독하려는 Observable 객체의 `addObserver()`메소드를 호출하면 됩니다.

**2. Observable 객체가 연락을 돌리는 방법**
Observable를 상속받은 주제객체는 `setChanged()` 메소드를 호출해서 객체의 상태가 바뀌었다는 것을 알립니다. 그리고 `notifyObservers()` 혹은 `notifyObservers(Object arg)`를 호출합니다.

> 보충설명
- `setChanged()`를 하지 않으면 `notifyObservers()`를 하여도 옵저버에게 연락이 가지 않습니다. `setChanged()` 가 일종의 필터 역할을 하는 것이지요. 예를들어 센서가 1단위로 자주 움직인다고 합시다. 바뀔 때마다 연락이간다면 비효율적입니다. 따라서 5단위 이상 바뀔때만 연락이 갈 수 있게끔 중간다리 역할을 하는게 `setChanged()`입니다.
- `notifyObservers(Object arg)` 는 옵저버들에게 임의의 Object를 전달합니다.

**3. 옵저버가 연락을 받는 방법**
옵저버에게 데이터를 보낼 때는 `notifyObservers(arg)` 처럼 인자로 전달합니다. 즉 푸쉬 방식이구요. 반대로 옵저버쪽에서 Observable 객체로부터 원하는 데이터를 가져가는 풀방식도 있습니다.

#### 3) 자바 내장 옵저버의 단점

**1. Observable은 클래스이다**
인터페이스가 아니기 때문에 주제로 할 객체를 서브클래스로 만들 수 밖에 없습니다. 만약 이미 다른 수퍼클래스가 있다면 사용을 할 수 없는 것이죠. 인터페이스가 없다는 것은 해당 라이브러리를 커스텀하여 사용할 수 없다는 점도 있습니다.

**2. Observable API는 Protected로 선언되었다**
`setChanged()` 메소드를 보면 Protected 선언이 되어있습니다. 즉 Observable의 서브클래스에서만 `setChanged()`를 호출할 수 있죠. 즉 Observable를 구현한 주제객체를 다른 패키지에서 인스턴스변수로 써먹을 수 없습니다. 이런 디자인은 상속보다는 구성을 사용한다는 디자인 원칙에도 위배됩니다.


만약 이런 단점들이 설계에 크게 영향을 미치지 않는다면 자바 내장 옵저버를 쓰는 방법도 괜찮습니다. 여의치 않다면 처음 했던 방법처럼 직접 인터페이스를 구현하는 방법을 쓰면 됩니다. 결국 중요한 건 패턴을 활용한다는 것이니까요.

이상 옵저버 패턴에 대한 설명이었습니다.
다음엔 데코레이터 패턴을 소개하겠습니다 ^^

>
[(이전 게시물) 1. 스트래티지 패턴 개념과 예제 (strategy pattern)](https://velog.io/@hanna2100/%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4-1.-%EC%8A%A4%ED%8A%B8%EB%9E%98%ED%8B%B0%EC%A7%80-%ED%8C%A8%ED%84%B4-strategy-pattern)