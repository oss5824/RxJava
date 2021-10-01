# 커스텀 옵저버블
## 콜러블
+ 러너블과 매우비슷하지만 러너블 인터페이스는 값을 반환하지 않고 콜러블은 값을 반환 함
+ 콜러블을 리액티브의 흐름의 일부로 만들기 위해 .fromCallable()메소드를 옵저저블에 사용할 수 있음
```java
    //call 함수는 인자가 없어 ()->{}방식으로 가독성 높임
    Callable<String>callable=()->{
        Thread.sleep(500);
        return "TEST";
    };
    Observable<String>observable=Observable.fromCallable(callable);
    observable.subscribe(System.out::println);
```
## 이미터
### 이미터 인터페이스
+ 이미터 인터페이스는 플로어블, 옵저버블, 싱글, 컴플리터블과 같은 모든 리액티브 유형에 사용할 수 있음
+ 이미터 인터페이스는 아이템이 옵저버블에 매우 자세한 방법으로 방출되는 방식을 제어할 수 있어 강력한 구조임
    - onNext(): 옵저버블에 새 값을 제공
    - onError(): 내부 에러나 예외를 알림
    - onComplete(): 옵저버블에 추가적인 값이 없는 것과 안전하게 종료할 수 있다는 것을 알려줌
+ .create()메서드로 옵저버블을 생성하면서 사용할 수 있음
```java
    Observable.<Integer>create(e->{
        e.onNext(1);
        e.onComplete();
        //onComplete는 값 제공이 끝날 때마다 호출해줘야하고 해주지 않으면 옵저버블은 종료되지 않고 수동으로 제거해주어야 함
    })
```
+ 이미터 인터페이스는 강력하지만 사용하기 어려운 경우가 있으므로 흐림이 간단하면 callable을 선호할 수 있음
+ 그래도 여러값이 반환될 때마다 외부 리스너에서 값을 받아야 한다면 .create()를 사용하는 것이 거의 필수적

### 이미터 청소
+ 옵저버블이 끝나게 되면 사용된 내부 자원의 정리가 필요할 때가 있음
```java
    Observable.create(e->{
        text.setOnClickListener(v->e.onNext(v));
    });
    // 이 옵저버블은 절대 완료되지 않지만 종료되도 이미터에 대한 참조가 항상 존재하기 때문에 문제가 있음
```
+ 위의 문제를 해결하기 위해 Cancellable액션을 추가해주어야 함
```java
    Observable.create(e->{
        e.setCancelable(()->text.setOnClickListener(null));
        text.setOnClickListener(v->e.onNext(v));
    });
```
+ 그리고 위처럼 뷰에 대한 작업을 하고 있을 때 MainThread의 Disposable을 사용하는 것이 좋음
```java
    Observable.create(e->{
        e.setDisposable(new MainThreadDisposable(){
            @Override
            protected void onDispose(){
                text.setOnClickListener(null);
            }
        });
        text.setOnClickListener(v->e.onNext(v));
    });
```
