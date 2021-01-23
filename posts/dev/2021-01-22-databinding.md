
# 데이터 바인딩이란?
---
**데이터 바인딩**은 프로그래매틱 방식이 아니라 선언적 형식으로 레이아웃의 UI 구성요소를 앱의 데이터 소스와 결합할 수 있는 라이브러리입니다. *백문이 불여일코* 이므로 코드를 봅시다.

**TextView에 userName을 넣는 방법**

```kotlin
// 프로그래매틱 방식
findViewById<TextView>(R.id.sample_text).apply {
	text = viewModel.userName
    }
```
```xml
<!-- 데이터 바인딩 이용 -->
<TextView
        android:text="@{viewmodel.userName}" />
```
 데이터 바인딩을 이용하면  레이아웃 파일에서 직접 데이터를 할당할 수 있습니다. 따라서 파일이 더욱 단순화되고 유지관리 또한 쉬워집니다. UI 프레임워크를 호출하지 않으므로 앱 성능이 향상되며 메모리 누수 및 null 포인터 예외를 방지할 수도 있습니다.
 
 # 데이터 바인딩 시작하기
 ---
 
레이아웃의 뷰와, 데이터가 있는 객체를 연결하려면 이를 결합하는 클래스가 필요합니다. 데이터 바인딩은 이러한 클래스를 자동으로 생성하고 결합시켜 줍니다.

데이터 바인딩은 데이터가 변경되었을때 이를 View에 알려주는 기능도 들어있습니다. 따라서 프로그래머는 UI 새로고침에 대해 전혀 신경쓰지 않아도 됩니다. 데이터가 변경되면, 이를 코드적으로 View에 알릴 필요 없이 자동으로 View에 전달이 될 테니까요.

## 1. 빌드 환경
---
Android 4.0 (API 수준 14) 이상

```json
android {
	...
	dataBinding {
		enabled = true
	}
}
```
1. build.gradle(Module: ..) 의 `android{ .. }`에 `dataBinding`을 추가합니다. 

```json
plugins {
	...
	id 'kotlin-kapt' // 추가
}

...

dependencies {
	...
	def archLifecycleVersion = '2.2.0'
	implementation "androidx.lifecycle:lifecycle-extensions:$archLifecycleVersion"
	kapt "androidx.lifecycle:lifecycle-compiler:$archLifecycleVersion"
	implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$archLifecycleVersion"

	def activity_version = "1.1.0"
	implementation "androidx.activity:activity-ktx:$activity_version"
}
```
2. `AAC ViewModel`과 함께 쓸 예정이므로 `lifecycle` 관련 라이브러리도 추가합니다

## 2. 레이아웃에 `<layout>`과 `<data>` 추가
---
```kotlin
package com.example.databinding

import androidx.lifecycle.ViewModel

class MainViewModel: ViewModel() {
    var userName =  ObservableField<String>()
    var phone =  ObservableField<String>()
    var point =  ObservableField<String>()
    
    fun onClickNextUser()  {

    }
}
```
MainViewModel 클래스에 있는 userName, phone, point를 레이아웃에 바인딩해보겠습니다.
여기서 주의할 점은 뷰에 즉각적으로 보여지기 위해 바인딩할 변수를 Observable한 타입으로 선언해주어야 합니다. 대표적으로 `ObservableField`와`LiveData` 있습니다. 일반적으로 안드로이드의 생명주기를 알고있는 `LiveData`를 더 권장합니다. 하지만 예제에서는 `ObservableField` 를 쓰겠습니다.

`onClickNextUser()` 메소드는 데이터가 뷰와 결합된 걸 보여주기 위해 미리 선언한 메소듭니다. 버튼을 누르면 `onClickNextUser()`가 호출되어 `userName`등의 데이터가 바뀌고 그 데이터가 뷰에 보이는지 확인해보겠습니다.



```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">
    <data>
        <variable name="viewModel" type="com.example.databinding.MainViewModel"/>
    </data>
  
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity"
        android:orientation="vertical"
        android:gravity="center">

    </LinearLayout>
</layout>
```
데이터 바인딩할 준비가 된 activity_main.xml 코드입니다.

생소한 태그가 보이네요. `<layout>` 과 `<data>` 태그 입니다. 데이터바인딩은 `<layout>`이라는 루트태그로 시작합니다. 그리고 그 뒤에 `<data>`태그로 결합할 객체의 루트(`com.example.databinding.MainViewModel`)를 적어주면 됩니다. `name`은  이 MainViewModel을 가리키는 변수명입니다.

```xml
    ...
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity"
        android:orientation="vertical"
        android:gravity="center">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{viewModel.userName}"
            android:textSize="30dp"
            android:gravity="center"/>

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{viewModel.phone}"
            android:textSize="30dp"
            android:gravity="center" />

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{viewModel.point}"
            android:textSize="30dp"
            android:gravity="center" />
      
        <Button
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Next User" 
            android:onClick="@{() -> viewModel.onClickNextUser()}" />

    </LinearLayout>
    ...
```

이제 `<LinearLayout>`안에 텍스트뷰 3개와 버튼을 만들고, text값에 `MainViewModel`의 멤버변수들을 바인딩해줍니다. 아까 `<data>`태그에서 `MainViewModel`의 `name`을 viewModel로 했으니 `@{viewModel.userName}` 로 적어주시면 됩니다. viewModel만 쳐도 자동완성으로 다 나옵니다. 만약 나오지 않는다면 빌드를 한번 해주세요.
만약 변수가 아닌 메소드를 바인딩하고 싶다면`@{() -> viewModel.onClickNextUser()}` 와 같이 람다식으로 해주면 됩니다. 버튼을 클릭하면 MainViewModel의 `onClickNextUser()` 메소드가 호출되겠네요.

## 3. MainViewModel에서 바인딩할 데이터 관리하기
---
```kotlin
class MainViewModel: ViewModel() {
    var index = -1
    private val userNameList = arrayListOf("짱구", "철수", "맹구")
    private val phoneList = arrayListOf("01020203030", "01040405050", "0101234567")
    private val pointList = arrayListOf(1200, 10900, 99000)

    var userName =  ObservableField<String>()
    var phone =  ObservableField<String>()
    var point =  ObservableField<Int>()

    init {
        onClickNextUser()
    }

    fun onClickNextUser()  {
        if (index == userNameList.size-1) {
            index = 0
        } else {
            index += 1
        }
        userName.set(userNameList[index])
        phone.set(phoneList[index])
        point.set(pointList[index])
    }
}
```

이제 보여줄 데이터를 `userName`, `phone` `point` 에 세팅해주면 됩니다. 리스트를 만들어서  `onClickNextUser()`가 호출될 때 마다 리스트 데이터를 하나씩 돌아가면서 보여주도록 코드를 짰습니다.

## 4. MainActivity에서 ViewModel 바인딩
---
```kotlin
class MainActivity : AppCompatActivity() {
    private val viewModel: MainViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val binding = DataBindingUtil.setContentView<ActivityMainBinding>(this, R.layout.activity_main)
        binding.viewModel = viewModel
    }
}
```

ViewModel을 생성해줍니다. `dependencies`에 `activity-ktx` 라이브러리를 추가했기 때문에 	`by viewModels()`로 손쉽게 만들 수 있습니다.

아까 `activity_main.xml`레이아웃 파일에서 `<layout>`태그로 감쌌던 것 기억하시나요? 그렇게 하고 빌드를 한 번 하면 `ActivityMainBinding` 이라는 클래스가 자동으로 생성됩니다. 만약 `apple.xml` 파일을 데이터 바인딩하면 `AppleBinding`이라는 클래스가 생기겠죠? `파일명 + Binding` 이 규칙입니다.
`ActivityMainBinding`을 생성한 후, `<data>`태그로 정의했던 viewModel을 주입해 주면 바인딩이 완료됩니다.
![](https://images.velog.io/images/hanna2100/post/87947247-3faf-463e-ba08-3cd350c9f36c/%EC%A0%9C%EB%AA%A9%20%EC%97%86%EC%9D%8C.png)

NEXT USER 버튼을 누르면 변경된 데이터가 TextView에 보여지는 걸 확인 할 수 있습니다.


## 5. 바인딩 어댑터 사용하기
---
```kotlin
import android.widget.TextView
import androidx.databinding.BindingAdapter
import java.text.DecimalFormat

object TextViewBindingAdapter {
    @BindingAdapter("bindPhoneNumber")
    @JvmStatic
    fun TextView.bindPhoneNumber(phone: String) {
        this.text = ""
        val builder = StringBuilder()
        for (index in phone.indices) {
            builder.append(phone[index])
            when (index) {
                2 -> {
                    builder.append("-")
                }
                5 -> {
                    builder.append("-")
                }
                10 -> {
                    builder.delete(7, 8)
                    builder.insert(8, "-")
                }
            }
        }
        this.append(builder.toString())
    }
    
    @BindingAdapter("bindComma")
    @JvmStatic
    fun TextView.bindComma(amount: Int) {
        val formatter = DecimalFormat("###,###")
        this.text = formatter.format(amount)
    }

}
```

전화번호가 01012345678 로 보이는 게 불편합니다. 중간에 `-`가 있었으면 좋겠네요.
이처럼 viewModel의 데이터를 가공해서 뷰에 보여주고 싶을 때 BindingAdapter를 활용하면 메소드를 따로 분리해서 관리할 수 있습니다. 뿐만 아니라 재사용을 할 수도 있죠.

`TextViewBindingAdapter`라는 Object를 하나 만들고, 바인딩어댑터를 정의해 줍니다.  `@BindingAdapter("bindPhoneNumber")` 라는 어노테이션을 사용합니다. 파라미터인 `bindPhoneNumber`는 레이아웃에 바인딩 할 때 사용할 이름이니 적당한 걸로 적어줍니다.

`@JvmStatic`은 이 메소드를 static으로 선언한다는 뜻 입니다. 코틀린은 자바처럼 메소드에 static을 붙이지 않으므로 어노테이션을 사용합니다. static으로 선언하는 이유는 모든 레이아웃에서 이 데이터 바인딩 객체에 동일하게 접근할 수 있어야합니다. 즉 메모리상에 올려두고 써야하기 때문에 정적으로 선언해주는 것 입니다. (정확하게는 인스턴스로 생성해도 큰 문제는 없으나, 메모리가 낭비되고 정적인 데이터와 바인딩 될 때 크래쉬가 나는 현상이 있어 static으로 하는 것이 관리하기 좋습니다.)

```xml
	...
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            bindPhoneNumber="@{viewModel.phone}"
            android:textSize="30dp"
            android:gravity="center" />

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            bindComma="@{viewModel.point}"
            android:textSize="30dp"
            android:gravity="center" />
 	...
```

레이아웃 파일에서 바인딩어댑터를 사용합니다. 이제 `android:text` 대신에 `bindPhoneNumber`로 TextView에 text 값을 넣어주게 됩니다.

![](https://images.velog.io/images/hanna2100/post/398cf0be-1366-4858-a0b4-bc3212da03a5/%EC%A0%9C%EB%AA%A9%20%EC%97%86%EC%9D%8C.png)

바인딩 어댑터를 사용해서 TextView에 text를 넣은 화면입니다. TextView 뿐만 아니라, 버튼, 이미지등 View와 관련된 모든 태그에 데이터바인딩을 활용 할 수 있습니다. 데이터 바인딩은 한 번 만들어 놓으면 모든 레이아웃 파일에서 사용 할 수 있기때문에, 작업이 훨씬 편해집니다.

이 포스트가 데이터 바인딩을 배우는 입문자들에게 도움이 됐기를 바랍니다.

