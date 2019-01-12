---
layout: posts
title:  "LongPolling(지연응답) 구현"
date:   2018-09-18 23:52:21 +0900
comments: true
categories: study
tags:
  - longpolling
  - 지연응답
redirect_from:
  - /study/2018/09/18/long_polling/
---


### 이론

얼마전에 개발팀 팀장님을 통해 실시간 프로토콜을 흉내 낸 Polling과 LongPolling(지연응답)이라는 기술을 알게되었고 이에 대해 궁금증이 생겨서 훑어본 내용과 실제 작성한 소스를 공유하고자 합니다.

<br>
폴링과 롱폴링 두 가지다 앞서 말한것 처럼 __실시간 프로토콜을 흉내 낸 기술__ 이기때문에 웹 소켓과 다르게 서버와 클라이언트 간의 연결이 유지되는 방식이 아니며 HTTP프로토콜 기반으로 요청을 보냄으로써 서버의 상태값을 얻어옴을 목적으로 사용합니다.
성능상으로는 웹소켓보다 떨어지지만 브라우저의 영향을 받지 않는다는 점과 간단하게 구현할 수 있다는 장점이 존재합니다.

폴링과 롤폴링의 차이점은 즉시응답과 지연응답 딱 하나입니다.
이것에 대해서는 [Naver 기술블로그](https://d2.naver.com/helloworld/1052)에서 잘 설명해 놓았네요.
요약해보면 폴링은 클라이언트에서 주기적으로 요청을 보내어 서버의 상태값을 알아옵니다.

롱폴링도 마찬가지로 클라이언트에서 주기적으로 요청을 보내지만 서버단에서 상태값이 변경되기 전까지 응답을 지연시키기 때문에 폴링에 비해 요청/응답 트래픽이 줄어든다는 장점이 있습니다. 추가적으로 마냥 요청을 지연시키는 것 또한 손해이기 때문에 일정시간이 지나면 서버는 상태값이 요청/응답을 해제시킵니다.

### 실습

여기까지 이론이고 실제로 저는 롱폴링 구현화를 쉽게하도록 돕기위해 __라이브러리로 작성하였습니다.__
추가적으로 해야될 것은 라이브러리를 사용하기위해 의존성 추가
```groovy
// https://github.com/JeHuiPark/LongPolling
repositories{
  // another repositories...
  maven { url "https://jitpack.io" }
}
dependencies {
  // another dependencies...
  compile 'com.github.JeHuiPark:LongPolling:0.1.2'
}
```

그리고 간단한 소스 몇줄
```java
public ResponseMap<Integer> progress(String id) {
	PollingManager pollingManager	= PollingManager.Lazy.INSTANCE;
	long LIFE_TIME = 10000;
	long TRANSACTION_TIME = 5000;
	long INTERVAL = 100;

	Builder<Integer> builder = new Builder<>();
	builder.setInterval(INTERVAL)
	.setTransactionTime(TRANSACTION_TIME)
	.setObserve(()->{
		return testMapper.count();
	})
	.setValidation((value, origin)->{ //param1: 최신 값, origin: 이전 값
		boolean isUpdate = false;
		if(!value.equals(origin))
			isUpdate = true;
		return isUpdate;
	});

	return (ResponseMap<Integer>) pollingManager.start(id, LIFE_TIME, builder);
}
```
  - 상태값의 제너릭을 빌더에 명시합니다. ``` Builder<Integer> ```
  - 상태값의 감시간격을 명시합니다. (ms) ``` setInterval(INTERVAL) ```
  - 폴링 한개의 최대 생명주기를 명시합니다. (ms) ``` setTransactionTime(TRANSACTION_TIME) ```
  - 어떤 상태값을 감시할것인지 명시합니다. ```setObserve(()->{return value}) ```
  - 상태값을 검증하는 로직을 명시합니다. ``` setValidation((value, origin)->{return true}) ``` true:상태값 변경을 의미, false: 상태값이 변경되지 않았음을 의미
  - Polling 초기화(이미 Polling이 생성된 상태라면 Builder및 LIFE_TIME은 무시되며 생성된 Polling에 접근) 및 폴링 프로세스 수행 ``` pollingManager.start(id, LIFE_TIME, builder)``` 여기서 __LIFE_TIME은 최대 지연시간입니다.__ 추후 클라이언트는 id를 이용하여 LIFE_TIME이내에 해당 Polling에 접근할 수 있습니다.

### 라이브러리 설명

  1. PollingManager는 싱글톤입니다.
  1. Polling은 단독수행 및 초기화 되지 않습니다.
  1. PollingManager의 start메소드를 사용하여 새로운 폴링을 생성하거나 접근할 수 있습니다.
  1. PollingManager의 destroy메소드를 사용하여 특정 Polling을 강제로 완전 삭제시킬 수 있습니다.
  1. Polling은 PollingManager에 의해 수행되며 __id로 관리됩니다.__
  1. Polling은 상태값이 변화하거나 TRANSACTION_TIME을 초과하면 자동으로 프로세스를 종료합니다.
  1. LIFE_TIME은 해당 Polling의 __최대 생명주기__ 이며 마지막 요청 이후 클라이언트로부터 __LIFE_TIME이내로 요청이 오지않는다면 해당 Polling은 destroy 됩니다.__
  1. __상태값이 변하지 않는다면 TRANSACTION_TIME 만큼 응답이 지연됩니다.__
  1. INTERVAL 주기로 setObserve를 통해 전달받은 메소드가 최신 상태값을 반환합니다. 그 후 setValidation을 통해 전달받은 메소드가 상태값 검증결과를 반환합니다.
  1. __상태값이 바뀌었다고 판단되면 TRANSACTION_TIME과 무관하게 즉시 Polling 프로세스가 종료됩니다.__
  1. Polling은 프로세스를 종료하면서 해당결과를 ResponseMap으로 반환합니다.
