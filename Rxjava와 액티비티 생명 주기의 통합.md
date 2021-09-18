# RxJava와 액티비티 생명주기의 통합
+ 생명 주기와의 인식과 통합이 없으면 구독과 옵저버블을 통제하기 어려움
+ 메모리 누수와 절대로 끝나지 않은 작업으로 이어지는 경우가 있기 때문에 이것을 예방해야함
+ 느슨한 구독을 방지해 생명주기 안에서 적절한 타이밍에 구독을 취소해야 함

## 안드로이드 생명주기
+ 액티비티는 안드로이드에서 사용자 인터페이스를 표시하는 핵심 구성요소이며 대부분의 모든 작업이 액티비티에 연결됨
+ 배경이미지를 가져오기 위해서는 onCreate()메소드를 사용할 것이고 액티비티가 시작될 때 업데이트할 데이터가 있는 경우 onStart() 또는 onResume()을 선택하는 것이 좋음
+ onCreate()에서 시작된 항목이 있으면 onDestroy()에서 중지해야함
+ 백그라운드 스레드가 onStart()에서 시작된 경우 onStop()에서 파기해야함. onResume과 onPause도 마찬가지
+ onCreate()호출에 대해 알아야 할 사항
  - 액티비티가 생성될 때 한 번만 호출되지만 까다로운 부분(startActivity()를 사용해)은 액티비티가 실제로 시작될 때가 아니더라도 파괴되고 생성될 수 있음
  - 액티비티가 파괴되고 생성될 수 있는 상황
    * 가로 방향에서 세로 방향으로 또는 그 반대로 방향이 바뀌도록 장치가 회전 됨
    * 시스템언어가 변경됨
    * 여러 액티비티 또는 애플리케이션 구성에서 활동 공간의 크기가 조정됨
    * 하드웨어 키보드가 튀어나옴
  - 위의 경우는 onPause->onStop->onDestroy이후 onCreate()부터 다시 시작 됨   

## 리소스 누수
+ 액티비티의 적절한 생명주기가 유지되지 않고 자원 저일를 돌보지 않으면 누수 발생
+ 메모리 누수는 가비지 컬렉터가 사용되지 않은 메모리 일부를 되찾지 못하게 되고 결국에는 사용 가능한 메모리가 모두 소모되어 전체 앱이 종료 됨
  - 가비지 컬렉터
    * 프로그램을 개발하다 보면 유효하지 않은 메모리인 가비지가 발생하고 C언어는 free라는 함수를 통해 직접 메모리를 해제해주지만 Java나 Kotlin은 JVM의 가비지 컬렉터가 불필요한 메모리를 알아서 정리해주기 때문에 개발자가 메모리를 직접 해제해주는일은 없음
    * 객체는 대부분 일회성되며 메모리에 오랫동안 남아있는 경우가 드물다는 것을 가정하여 객체의 생존기간에 따라 물리적인 Heap영역을 Young, Old 총 두가지 영역으로 나눔
    * Young영역: 새롭게 생성된 객체가 할당되는 영역이고 대부분의 객체가 금방 Unreachable상태가 되기 때문에 Young영역에 생성되었다가 사라짐. Young 영역에 대한 가비지 컬렉션을 Minor GC(빠름)라고 부름  
    * Old영역: Young영역에서 Reachable상태를 유지하여 살아남은 객체가 복사되는 영역이며 복사되는 과정에서 대부분 Young영역보다 크게 할당되며 크기가 큰 만큼 가비지는 적게 발생. Old영역에 대한 가비지 컬렉션을 Major GC(느림)또는 Full GC라고 부름
    * Stop The World: 가비지 컬렉션을 실행하기 위해 JVM이 앱의 실행을 멈추는 작업으로 GC가 실행될 때는 GC를 실행하는 쓰레드를 제외한 모든 쓰레드들의 작업이 중단되고 GC가 완료된 후 작업이 재개 됨
    * Mark and Sweep: Mark는 사용되는 메모리와 사용되지 않은 메모리를 식별하는 작업이고 Sweep은ㅇ Mark단게에서 사용되지 않음으로 식별된 메모리를 해제하는 작업.
+ 스레드(or 옵저버블) 누수는 액티비티가 종료되도 백그라운드에서 실행중인 작업이 있을 수 있으며 무의미하게 CPU를 사용하게 하고 베터리를 소모함
+ 메모리 누수
  - 사용되지 않는 메모리 블록에 여전히 참조를 가리키는 메모리 블록이 있다면 GC에서 회수 할 수 없어서 메모리 누수 발생
  - 일반적으로 액티피티 파과 후 액티비티 인스턴스를 여전히 가리키는 전역 참조가 있는 경우 발생
+ 옵저버블 누수
  - 특정 시점에서 계산 결과를 UI로 업데이트해야하기 때문에 옵저버블은 액티비티 인스턴스에 액세스할 수 있음
  - 액티비티 자체를 변경하지 않아도 onCreate()메소드가 여러번 호출될 수 있다는 것을 보았고 그로 인해 액티비티는 파괴되었지만 메모리는 정리되지 않고 누수된 옵저버블이 남아있을 수 있음
  - 아래 예제의 경우 this 참조를 통해 액티비티에 대한 강력한 참조가 유지 되서 옵저버블과 구독이 여전히 활성화 되어 있는 동안 액티비티가 GC에 의해 절대 처리될 수 없음을 의미함. 이런 경우 옵저버블은 절대 종료되지 않고 수동으로 정리해야함(옵저버블이 액티비티와 제거되지 않는 이유는 옵저버블은 액티비티에 의해 소유되는것기 아니라 이를 실행 중인 스레드가 소유하고 있기 때문)
```java
  Observable.interval(0,5,TimeUnit.SECONDS)
  .doOnError(error->{
    Toast.makeText(this,"error message",Toast.LENGTH_SHORT).show();
  })
```

## 구독정리
+ 구독은 메모리와 스레드 누수를 유발할 수 있으므로 정리를 해주어야 함
+ 가장 간단한 방법으로는 디스포저블 인터페이스를 사용하는 것이고 이 외에도 RxLifecycle 라이브리러를 사용해 생명주기 관리를 거의 자동으로 수행하는 방법 등이 있음
+ 모든 구독을 수동으로 파기하거나 취소해야하는 것은 아니며 아래와 같은 단순한 호출은 자동으로 종료됨
```java
  Observable.just(1).subscribe();
```
+ 디스포저블 사용
  - 모든 subscribe()호출은 디스포저블 인터페이스를 반환하기 때문에 더 필요 없는 구독을 종료하는 가장 쉬운 방법이고 반환된 객체에 대한 참조로 취소할 수 있음
```java
  private Disposable disposable;
  @Override
  protected void onCreate(Bundle savedInstanceState){
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_mock);
    disposable=Observable.interval(1,TimeUnit.SECONDS);
  }
  @Override
  protected void onDestroy(){
    if(disposable!=null){
      disposable.dispose();
    }
    super.onDestroy();
  }
  //위처럼 onStart-onStop과 onResume-onPause 쌍에서도 같은 작업을 수행할 수 있으나 쌍을 혼합하지 않으면 누수 또는앱의 일관성 없는 동작으로 이어질 가능성이 높음
```
+ 컴포짓디스포저블 사용
  - 같은 액티비티에서 여러 구독을 추적해야하는 경우 수동으로 추적하는 접근 방식이 까다로울 수 있으므로 이 경우 여러 디스포저블에 대한 참조를 유지할 수 있고 동시에 모든 디스포저블에 대한 구독을 취소할 수 있는 클래스인  컴포짓디스포저블이 매우 편리함

```java
  private CompositeDisposable disposable;
  disposable = new CompositeDisposable();

  Disposable disposable1 = Observable.interval(1,TimeUnit.SECONDS);
  ...
  //N개 있다고 했을 때
  disposable.addAll(
    disposable1,disposable2,...,disposableN
  );
  @Override
  protected void onDestroy(){
    if(disposable!=null){
      disposable.dispose();
    }
    super.onDestroy();
  }
  //위처럼 각 디스포저블에 대해서 개별적인 dispose를 호출할 필요가 없음
```
+ RxLifecycle 라이브러리 활용
  - 이 라이브러리가 옵저버블 흐름에 한 줄만 추가함으로 생명주기 관리 문제를 해결할 수 있음을 알 수 있을 것
  - 활동주기에 바인딩
    * RxLifecycle을 사용하는 가장 간단한 방법은 RxActivity를 상속하는 것(extends RxAppCompatActivity)
    * RxActivity를 상속하면 구독을 종료하는데 사용할 수 있는 bindToLifeCycle()메소드를 액세스할 수 있음
```java
  public class ExampleLifeCycleActivity extends RxAppCompatActivity{
    @Override
    protected void onCreate(Bundle savedInstanceState){
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_mock);
      Observable.interval(1,TimeUnit.SECONDS)
      .compose(bindToLifecycle())
      //액티비티의 생명주기에서 이벤트를 수신하고, 적절한 이벤트를 발견하면(이 경우에 onDestroy())구독이 종료됨
      .subscribe();
    }
  }
```

+ 서브클래싱 없이 액티비티에 바인딩하기
  - 클래스가 이미 제어할 수 없는 다른 클래스를 상속하는 경우가 자주 발생하므로 클래스가 항상 RxActivity를 상속할 수는 없을 수도 있음
  - 이 경우 가장 좋은 방법은 서브젝트를 도입해 RxLifeCycle클래스를 다시 구현하는 것
```java
  BehaviorSubject<ActivityEvent> lifecycleSubject = BehaviorSubject.create();
  void onCreate(){
    super.onCreate();
    lifecycleSubject.onNext(ActivityEvent.CREATE);
  }
  void onDestroy(){
    lifecycleSubject.onNext(ActivityEvent.DESTROY);
    super.onDestroy();
  }
```
