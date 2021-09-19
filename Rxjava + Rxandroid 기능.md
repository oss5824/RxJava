# Rxjava + Rxandroid 기능

+ just
데이터를 인자로 넣으면 자동으로 알림 이벤트 발생
System.out::println은 수신한 데이터를 Syste
m.out::println을 통해 노출한다는 의미
```kotlin
    fun main(){
        Observable.just("test").subscribe(System::.out::println)
    }
```
+ create
onNext, onCoomplete, onError같은 알림을 개발자가 직접 호출해야함
```kotlin
    Observable.create<Integer> { 
        it.onNext(100 as Integer) 
        it.onNext(200 as Integer) 
        it.onNext(300 as Integer) 
        }.subscribe { data-> println(data) }
```

+ array(immutable 경우,mutable경우 모두 toObservable사용)
```kotlin
    val arr = arrayOf(100, 200, 300) 
    val source = arr.toObservable() 
    source.subscribe{ println(it) }
```

+ fromCallable()
Runnable 인터페이스처럼 메서드가 하나이고 인자가 없다는 점에서 비슷하만 결과를 리턴한다는 점에 차이가 있고 Excutor 인터페이스에서 인자로 활용되기 때문에 잠재적으로 다른 스레드에서 실행된다는 것을 의미 
```kotlin
    Observable.fromCallable{
        "test"
    }.subscribe(System.out::println)
```

+ fromFuture()
계산의 결과를 구할 때 사용하고 일반적으로 Executor 인터페이스 구현한 클래스에 Callable객체를 인자로 넣어 Future객체를 반환
```kotlin
    val future = Executors.newSingleThreadExecutor().submit<String> { "test" } 
    val source = Observable.fromFuture(future) 
    source.subscribe(System.out::println)
```

+ 클릭이벤트
```kotlin
    fun processClickEvent(view: View){
        val observable = view.clicks()
        obsrevable.subscreibe{Toast.makeText(context,"clicked",Toast.LENGTH_SHORT).show()}
    }
```

+ TextWatcher
```kotlin
    fun processTextWatcher(tv:TextView){
        val observable = tv.textChanges()
        observable.subscribe{charSequnce->Toast.makeText(context,charSequence.toString(),Toast.LENGTH_SHORT).show()}
    }
    //textChanges()의 경우 onTextChanged()에서 호출하지만 실제로 beforeTextChanged()나 afterTextChanged()에서와 같이 특정 코드작업이 필요할 수 있으므로 여러 api제공
```

+ Uservul Operators
debounce를 이용한 SingleClick(동일 view연속 클릭방지)
View에 singleClick()이라는 이름으로 Extension function을 만들어놓으면 편리
```kotlin
    button.clicks().debounce(500,TimeUnit.MILLISECONDS)
    .subscribe{Toast.makeText(context,"singleClick",Toast.LENGTH_SHORT).show()}
```

여러 view를 동시에 누르는 경우(여러개의 view 동시클릭 처리)
merge이용
```kotlin
class ClickActivity: Activity(){
    private val btn1=Button(this)
    private val btn2=Button(this)
    private val btn3=Button(this)
    private val disposables=CompositeDisposable()

    override fun onCreate(ssavedInstanceState: Bundle?){
        super.onCrete(savedInstanceState)
        val btn1Ob=bt1.clicks().map{btn1}
        val btn2Ob=bt2.clicks().map{btn2}
        val btn3Ob=bt3.clicks().map{btn3}

        val disposable=Observable.mergeArray(btn1Ob,btn2Ob,btn3Ob).subscribe{
            when(it){
                btn1->{}
                btn2->{}
                btn3->{}
                else ->{}
            }
        }
        disposables.add(disposable)
    }
    override fun onDestroy(){
        super.onDestroy()
        disposables.clear()
    }
}
```
             

