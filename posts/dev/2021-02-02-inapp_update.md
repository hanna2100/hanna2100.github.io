# 인앱 업데이트 테스트 방법
---
[Google Play Core]('https://developer.android.com/guide/playcore?hl=ko') 라이브러리를 이용하면 유저가 플레이스토어 앱을 열지 않아도, 인앱에서 바로 업데이트를 진행 할 수 있다.

개발하는 방법도 그리 어렵지 않으나, 문제는 테스트를 어떻게 하느냐이다. 테스트를 위해 스토어에 업로드를 할 순 없으니 말이다.

[FakeAppUpdateManager]('https://developer.android.com/reference/com/google/android/play/core/appupdate/testing/FakeAppUpdateManager') 라는 테스트 라이브러리가 제공되기는 하나, `userCancelsDownload()` 처럼 각 상황별로 시뮬레이션 코드를 새로 작성해야하는 번거로움이 있다.


# 내부 앱 공유로 테스트하기
---
내부 앱 공유를 이용하면 인앱 업데이트를 테스트 할 수 있다. 참고로 내부 테스트 트랙과는 다른 것이다. 내부 앱 공유는 앱을 업로드 하는 즉시 사용할 수 있다.

## 0. 사전 준비
>
1. Google Play 스토어 앱의 메뉴 > 설정에 들어간다.
2. 정보란의 Play 스토어 버전을 6번 터치한다.
2. 내부 앱 공유 옵션이 표시되면 사용을 설정한다.


## 1. 앱 번들 빌드

커맨드 창에서 `./gradlew bundleRelease` 를 입력하여 앱 번들을 빌드한다.

## 2. 내부 앱 공유에 업로드

`project-name/module-name/build/outputs/bundle` 에 빌드된 앱 번들을 [내부 앱 공유]('https://play.google.com/apps/publish/internalappsharing/')에 업로드 한다.

## 3. 업로드 한 버전 설치

업로드가 완료되면 다운로드 링크를 복사한다. 링크를 통해 앱을 다운받는다.

## 4. 버전코드를 올리고 앱 번들 빌드

`build.gradle`의 버전코드를 한단계 올리고 번들을 한번 더 빌드한다.

## 5. 내부 앱 공유에 새로운 버전 업로드

새로운 버전의 앱번들을 업로드 한다. 링크가 생성되면 클릭을 하되 **'업데이트' 옵션이 표시되는 것 만 확인하고, 업데이트는 진행하지 않는다.**

## 6. 설치한 앱을 실행

앱을 열면 앱이 새로운 버전을 감지하여 인앱 업데이트를 실행한다. 