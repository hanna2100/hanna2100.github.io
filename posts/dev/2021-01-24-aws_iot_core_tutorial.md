# AWS IoT Core란?
---
![](https://images.velog.io/images/hanna2100/post/9a5a8821-8845-40d2-917f-6d840f5148de/what-is-aws-iot.png)

AWS IoT는 IoT 디바이스간의 연결 및 AWS 클라우드 서비스와의 연동기능을 제공합니다. 디바이스를 AWS IoT에 연결할 수 있는 경우 AWS IoT는 AWS가 제공하는 클라우드 서비스에 디바이스를 연결할 수 있습니다.

AWS에서는 4가지 프로토콜을 지원합니다.

1. [MQTT(메시지 대기열 및 원격 측정 전송)](https://docs.aws.amazon.com/ko_kr/iot/latest/developerguide/mqtt.html)
2. [MQTT over WSS(Websockets Secure)](https://docs.aws.amazon.com/ko_kr/iot/latest/developerguide/mqtt.html)
3. [HTTPS(Hypertext Transfer Protocol - Secure)](https://docs.aws.amazon.com/ko_kr/iot/latest/developerguide/http.html)
4. [LoRaWAN (Long Range Wide Area Network)](https://docs.aws.amazon.com/ko_kr/iot/latest/developerguide/connect-iot-lorawan.html)

AWS IoT Core에서는 MQTT와 MQTT over WSS를 위한 메세지 브로커가 제공되며, HTTPS 웹소켓 사용도 가능합니다. LoRaWAN를 사용하면 무선 LoRaWAN(저출력 장거리 광대역 네트워크) 디바이스를 연결하고 관리할 수 있습니다.

# AWS IoT Core의 작동방법
---
## 1. 디바이스 게이트웨이
**IoT 사물을 디바이스 게이트웨이에 연결**

디바이스 게이트웨이를 사용하여 안전하고 효율적으로 통신할 수 있습니다. 사물이 서로 다른 프로토콜을 사용하는 경우라도 디바이스 게이트웨이를 통해 서로 통신할 수 있습니다.

![](https://images.velog.io/images/hanna2100/post/1d958008-98d1-4fc2-86ed-0e5840e8f7f8/1.gif)

위의 예는 두 개의 사물(제어장치, 전등)이 디바이스 게이트웨이에 연결되어 있는 상태를 보여 줍니다. 제어 장치가 디바이스 게이트웨이로 명령을 게시하면, 전등이 해당 명령을 구독하고 읽을 수 있습니다.

## 2. 규칙엔진
**규칙 엔진으로 데이터를 처리**

규칙 엔진이 AWS IoT에 게시된 메시지를 수신하여 평가한 다음 정의된 비즈니스 규칙을 바탕으로 다른 사물이나 클라우드로 변환하여 전송합니다.

![](https://images.velog.io/images/hanna2100/post/db9b8453-94df-49e7-b574-91d13021ada0/2.gif)

1. 제어 장치에서 게시한 명령을 평가
2. 명령이 'B'인지 여부 판단
3. 명령이 'B'인 경우 규칙은 메시지를 'G'로 변환한 다음 'G'를 전등에 전달

## 3. 규칙작업
**규칙 엔진으로 특정 작업을 수행**

규칙 엔진은 메시지를 AWS Lambda 함수와 같은 클라우드 엔드포인트나 DynamoDB 테이블에 라우팅할 수도 있습니다.  [// 자세히보기](https://docs.aws.amazon.com/ko_kr/iot/latest/developerguide/iot-rule-actions.html?icmpid=docs_iot_console)

![](https://images.velog.io/images/hanna2100/post/dade4447-bc26-4d18-81c1-3c2524c3f2f6/3.gif)

1. 제어 장치에서 게시한 명령을 평가
2. 명령이 'R'인지 여부 판단
3. 명령이 'R'인 경우, 모바일 푸시 알림을 위해 메시지의 사본을 엔드포인트(DynamoDB, Lambda, SNS)로 전달

## 4. 디바이스 섀도우
**디바이스 섀도우로 디바이스 상태를 확인하고 설정**

AWS IoT에는 디바이스 레지스트리와 디바이스 섀도우가 포함되어 있으므로, 클라우드에 표현하고자 하는 임의의 사물을 이름, 일부 속성 및 영구 가상 '섀도우'를 사용하여 등록할 수 있습니다.

아래 예는 전등을 나타내는 사물이 클라우드에 가상 전등으로 생성되었음을 보여 줍니다.

![](https://images.velog.io/images/hanna2100/post/3a3b956e-7f94-48f3-a6d8-d38c8bfc5fd6/4.gif)

1. 전등이 꺼짐
2. 제어장치에서 게시한 명령을 디바이스 섀도우에 전달
3. 전등이 다시 켜질 때, 섀도우와 동기화

## 5. 솔루션 구축
**디바이스 섀도우로 디바이스 상태를 확인하고 설정**

AWS IoT를 사용하면 IoT 사물과 상호 작용하는 앱을 쉽게 만들 수 있습니다.

![](https://images.velog.io/images/hanna2100/post/c16fc7a7-c9cb-4262-b916-27fb656991a2/5.gif)

위의 예는 전등 색상을 바꾸는 모바일 앱입니다. 모바일 앱은 전등과 직접 통신하지 않습니다. 대신 REST API를 사용하여 전등의 디바이스 섀도우 상태를 읽고 설정합니다.



> 자료참고
https://docs.aws.amazon.com/ko_kr/iot/latest/developerguide/what-is-aws-iot.html
https://ap-northeast-1.console.aws.amazon.com/iot/home?region=ap-northeast-1#/tutorial?step=1

