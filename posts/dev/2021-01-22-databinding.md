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
 
레이아웃의 뷰와, 데이터가 있는 객체(viewModel의 userName)를 연결하려면 이를 결합하는 클래스가 필요합니다. 데이터 바인딩은 이러한 클래스를 자동으로 생성하고 결합시켜 줍니다.

데이터 바인딩은 데이터가 변경되었을때 이를 View에 알려주는 기능도 들어있습니다. 따라서 프로그래머는 UI 새로고침에 대해 전혀 신경쓰지 않아도 됩니다. 데이터가 변경되면, 이를 코드적으로 View에 알릴 필요 없이 자동으로 View에 전달이 될 테니까요.

## 1. 빌드 환경

지원기기 : Android 4.0 (API 수준 14) 이상

```gradle
android {
	...
	dataBinding {
		enabled = true
	}
}
```
build.gradle(Module: ..) 의 `android{ .. }`에 `dataBinding`을 추가한다.

## 2. 레이아웃에 데이터 추가
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