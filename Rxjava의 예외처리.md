# RxJava의 예외처리
## 예외처리
+ subscribe()사용
  - 오류 및 예외를 처리하는 가장 일반적인 방법은 subscribe()메소드를 사용하는 것
  - subscribe() 메소드는 옵저버블 또는 연산자가 던진 예외를 처리하는 데 이용하는 추가 인수를 사용
```java
  Observable.just("One").doOnNext(i->{
    throw new RuntimeException("Very wrong");
  })
  .subscribe(item->log("subscribe",item), this::log);
```

+ onExceptionResumeNext() 사용
  - 이것은 예외 발생 후 다른 옵저버블에서 흐름 처리를 복원하는 좋은 방법
  - 원본이 실패한 경우 백업 옵저버블에 플러그인하는 메커니즘으로 사용 가능
```java
  Observable.<String>error(new RuntimeException("Crash!"))
  .onExceptionResumeNext(Observable.just("Second"))
  .subscribe(item->{
    log("subscribe",item);
  },e->log("subscribe",e));
  //예외가 발생하면 onExceptionRexumeNext()연산자에 도달하고 두 번째 옵저버블에서 시퀀스를 재개
  //에러를 발생시키고 작동을 멈추는 특수 옵저버블 사용(첫번째 줄)
```

+ doOnError()사용
  - doOnError()는 위의 것들과 약간 다름
  - subscribe()에 아직 도달하지 않은 오류를 가로채기 위해 사용
  - 안드로이드 UI에 알림을 표시하거나 일반적으로 실패한 것을 기록해야할 때 매우 유용한 도구
```java
  Observable.<String>error(new RuntimeException("Crash!"))
  .doOnError(e->log("doOnNext"),e)
  .onExceptionResumeNext(Observable.just("Second"))
  .subscribe(item->{
    log("subscribe",item);
  },e->log("subscribe",e));
  // onExceptionResumeNext() 블록에 도달하기 전에 예외를 가로챔
```

## 기타 오류 처리
+ onErrorResumeNext()
  - onExceptionResumeNext()와 매우 비슷하지만 더욱 일반적인 오류를 처리할 수 있음
  - onExceptionResumeNext()는 java.lang.Exception 유형과 이것의 서브클래스의 throwable만 처리하지만 onErrorResumeNext()는 java.lang.Throwalbe 및 java.lang.Error를 잡을 수 있음
  - OutOfMemoryError와 같은 시스템 오류를 잡는데 도움이 되는 더 일반적인 핸들러
  
+ onErrorReturn()
  - onErrorResumeNext()와 비슷한 방식으로 작동하지만 흐름을 재개하기 위해 옵저버블을 사용하는 대신 옵저버블에서 래핑되지 않은 값을 사용

+ onErrorReturnItem
  - onErrorReturn()과 비슷하지만 더 간단하고, 이 경우 옵저버블이 생성된 시점에 지정된 상수가 반환 됨

## 중앙 집중식 에러 로깅
+ 각 subscribe()블록 자체에 오류처리 메커니즘이 있는 경우 코드가 엉망이 될 뿐더러 같은 핸들러를 복사해 붙여 넣는 것은 시간낭비이므로 중앙 집중식 로깅 솔루션을 구현해 유지 보수 및 사용이 간편한 방법이 필요
+ 중앙 처리기
  - 중앙 집중식 로깅을 구현하는 경우 매우 편리하고 유연한 방법은 일반적인 예외 처리가 필요한 곳에서 재사용되는 핸들러를 사용하는 것
  - 기본적으로 이것은 Consumer<Throwable>인터페이스를 구현하는 클래스가 될것이며 Rxjava가 해당 인터페이스를 사용하는 곳이면 어디서든 연결할 수 있고 doOnError() 또는 subscribe()블록과 같은 메소드가 포함 됨


