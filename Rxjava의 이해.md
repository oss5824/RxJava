# 기본 RxJava의 이해
+ RxJava는 결과에 반응하는 것  
+ 아이템을 리액티브 방식으로 처리하고 매우 조작하기 쉬운 인터페이스로 복잡한 조작과 처리 체계를 만들 수 있는 프레임워크를 제공  
+ RxJava의 기본 요소  
  - 옵저버블(Observable): 데이터의 출처(source)
  - 서브스크립션(Subscription): 데이터를 수신하는 옵저버블에 대해 활성화된 핸들  
  - 스케줄러(Scheduler): 스레드에서 데이터가 처리되는 위치를 정의하는 방법  

## 옵저버블  
+ 모든 것은 옵저버블과 함께 시작
+ 옵저버블은 전송된 데이터에 관해서 관찰할 수 있는 데이터의 출처  
+ just는 옵저버블을 생성할 수 있는 가장 간단한 방법이고 구독한다고 하기 전까지는 데이터를 전송하지 않음
  - .subscribe()를 호출해서 구독하고 보통은 누군가 구독해야 옵저버블이 활성화 됨
+ 핫 그리고 콜드 옵저버블  
  - 콜드 옵저버블  
    * 콜드 옵저버블은 구독자가 있기 전까지는 옵저버블로 아이템을 전송하지 않겟다는 의미
    * 어떤 이벤트가 진행중이어도 .subscribe()가 호출되기 전에 아이템은 생성되거나 전송받을 수 없음  
  - 핫 옵저버블  
    * 옵저버블이 만들어지면서 내부적으로 아이템을 생성(전송)함
    * 생성된 상태나 정보들이 계속해서 업데이트되고 이 자료를 받을 준비가 됐는지 아닌지 전혀 신경쓰지 않기 때문에 구독자가 아무도 없다면 업데이트 된 데이터들은 잃어버리게 됨  

## 디스포저블 
+ 옵저버블의 생애 주기를 제어하기 위해 사용하는 도구  
+ 참조를 얻는 방법
```java
  Disposable disposable = Observable.just("first","second").subscribe();
```
+ 디스포저블은 간단한 인터페이스로 dispose()와 isDisposed() 메소드가 있음  
  - dispose()는 지금 존재하는 구독(disposable)을 취소할 때 사용
  - isDisposed()는 구독이 활성화 상태인지 점검 가능(일반적으로는 구독되어 있지 않거나 생각할 필요가 없기 때문에 일반적인 코드에서 잘 사용하지 않음)   
+ 디스포저블은 CompositeDisposable을 이용해 그룹화할 수 있음  
```java
  Disposable disposable = new CompositeDisposable(
      Observable.just("first","second").subscribe(),
      Observable.just("1","2").subscribe()
  )
  //액티비티를 제거할 때처럼 많은 옵저버블을 한번에 취소해야하는 경우 유용
```

## 스케줄러 
+ 스케줄러는 현재나 나중에 동작해야 할 단위의 일정을 만드는 것
+ 코드가 어디서 실행되어야할지 제어할 수 있다는 의미(특정 스레드를 선택할 수 있다는 의미)  
+ 대부분의 구독자는 백그라운드 스레드에서 긴 시간 동안 실행되는 작업을 하므로 UI스레드를 방해하지는 않을 것
+ 스케줄러는 .subscribeOn()을 호출하면서 설정할 수 있음
+ 주요 스케줄러
  - Schedulers.io(), Schedulers.computation()->Cpu에 대응하는 계산용 스케줄러, Schedulers.newThread(), AndroidSchedulers.mainThread()
  - AndroidSchedulers.mainThread()는 안드로이드 시스템에서만 사용 가능  
+ 안드로이드에서는 모든 UI변경이 메인스레드에서 동작해야하므로 백그라운드에서 긴시간동안 동작하는 프로세스의 결과를 메인 스레드에 보여줘야하는 경우 .observeOn()을 사용할 수 있음  
+ subscribeOn()함수는 Observable에서 구독자가 subscribe() 함수를 호출했을 때 데이터 흐름을 발행하는 스레드를 지정하고, observeOn()함수는 처리된 결과를 구독자에게 전달하는 스레드를 지정
+ subscribeOn()함수는 처음 지정한 스레드를 고정시키므로 다시 호출된다해도 무시함, observeOn()은 변경이 가능(활용범위 넓음)

## 옵저버블의 흐름 조사
+ doOnComplete : 마지막 요소가 방출될 때 onComplete바로 전에 실행 됨. 에러가 발생하면 실행되지 않음
+ doOnDispose : 구독을 해지했을 때 실행 됨
+ doOnNext : 요소를 방출할 때, onNext 바로 전에 실행 됨
+ doOnError : 오퍼레이션 중 에러가 발생했을 경우, onError바로전에 실행 됨  
+ doOnEach : doOnNext,doOnError,doOnComplete의 통합버전으로 Notification<타입>을 아규먼트로 받아 각각에 맞는 처리를 할 수 있음
+ doOnSubscribe : 구독시, onSubscribe바로전에 실행 됨  
+ doOnLifeCycle : doOnSubscribe,doOnDispose의 통합버전
+ doOnTerminate : Observable이 종료될 때나 onComplete또는 onError바로 전에 실행 됨
+ doAfterNext : 요소 방출시 onNext바로 후 실행 됨  
+ doAfterTerminate : Observable이 종료될 때 onComplete나 onError후에 실행 됨
+ doFinally : Observable이 종료될 때 실행 됨

## 플로어블  
+ 플로어블은 옵저버블의 특별한 타입으로 간주(내부적으로는 아님)할 수 있지만 대부분은 옵저버블과 비슷한 메소드의 특징을 갖고 있음  
+ 플로어블을 이용하면 출처로부터 다음 단계 중 일부가 처리하는 것보다 더 빨리 방출된 아이템들을 처리할 수 있음(받는쪽의 속도가 주는 쪽의 속도를 따라가지 못할 때)
```java
  observable.toFlowable(BackpressureStrategy.MISSING).observeOn(Schdulers.computation())
  .subscribe(v->log(~~));
  // 처리할 수 있는 아이템을 다루는 법을 정해주지 않았기 때문에 MissingBackpressureException을 던짐
  // 몇 개의 Backpressure 전략이 있고 이것들은 많은 양의 아이템을 어떻게 다루는지 정의해줌
```

### 아이템 버리기
+ 아이텝 버리기는 속도를 따라잡지 못한다면 처리하지 못하는 아이템을 그냥 버리는 것이며 데이터를 버려도 문제없을 때만 사용할 수 있고 방출되는 데이터의 가치를 처음부터 신경써야 함
```java
  observable.toFlowable(BackpressureStrategy.DROP)
  //or observable.toFlowable(BackpressureStrategy.MISSING).onBackpressureDrop()
  //or observable.toFlowalbe(BackpressureStrategy.MISSING).sample(10,TimeUnit,MILLISECONDS).observeOn(Schedulers.computation).subscribe(v->log(~~));
  // samlpe은 주기적으로 아이템을 방출하고 사용가능한 마지막 값만 취함(위는 10밀리세컨드간격으로 수행)
``` 

### 버퍼링  
+ 보통 방출되고 소비되는 아이템의 속도가 다른 것을 다룰때는 좋지 않은 방법이지만 컨슈머 중 하나에서 일시적인 속도 둔화라면 문제없이 동작
+ 방출된 아이템은 남겨진 처리가 끝날 때까지 저장되고 속도 둔화가 끝나면 다시 컨슈머가 따라잡을 것임(컨슈머가 따라잡지 못한다면 일정 시점에 버퍼는 고갈되어 원래문제인 메모리 고갈과 비슷한 증상을 보일 것임)
```java
  observable.toFlowable(BackpressureStrategy.BUFFER)
  // or observable.toFlowable(BackpressureStrategy.MISSING).buffer(10) ->버퍼에 특정 수치를 지정하고싶을 때
```

## 컴플리터블, 싱글, 메이비 타입
+ 옵저버블, 플로어블 외에 RxJava에 존재하는 타입(거의 사용되지 않음)
+ Complitable: 미래에 완료될 결과가 없는 행위
  - 기본적으로 onComplete와 onError 두가지 타입의 행위만 처리
+ Single: 옵저버블과 비슷하지만 스트림 대신에 하나의 아이템을 반환
  - 단일 아이템을 리턴할 옵저버블을 표현하는 방법을 제공
+ Maybe: 어떤 값의 반환 없이 완료할 수 있는 행위를 의미. 하지만 싱글과 같이 아이템을 반환할 수 있음
  - 메이비 타입은 싱글타입과 매우 유사하며 마지막에 아이템이 반환되지 않을 수도 있음
