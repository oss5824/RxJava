# 고급 옵저버블
## 옵저버블 벗기기
### map
+ 함수형 메소드로 새로운 값을 반환하거나 값의 유형을 변경할 수 있음
```java
    Observable.just(1).map(i->i+1)
    Observable.jusy(1).map(i->i.toString())
```

### FlatMap 옵저버블
+ .flatMap()은 맵 연산으로 하나의 값을 다른 값으로 반환
+ 연산 결과가 반드시 Observable로 나옴
+ map이 1:1 데이터 연산이라면 flatMap은 1:N,1:1 Observable 함수
+ map은 동기방식이라 비동기와 함께 사용하면 효과를 보기 어렵고 비동기로 처리하고자하는 경우 flatMap을 사용해야함
+ flatMapSingle(): 싱글 형식을 취해 옵저버블 형식으로 자동 변환
+ flatMapxxx(): xxx형식을 취해 옵저버블 형식으로 자동 변환

### switchMap
+ flatMap과 매우 유사하지만 flatMap은 내부에서 생성된 옵저버블에서 모든요소를 반환하기 위해 wait를 반환하고 switchMap은 최신 값에서 생성된 최신 옵저버블의 요소만 반환 함
+ 다음 옵저버블이 반환되자마자 이전에 사용된 옵저버블을 종료하고 최슨 옵저버블에서만 항복을 생성
+ 새 파라미터가 도착하자마자 이전 값으로 생성된 옵저버블을 종료하고 싶을 때 사용

### Tuple
+ map에서 여러 값을 반환하는 문제의 해결책 중 하나로 Tuple이 있음
+ Pair
```java
    Observable.just("id1","id2","id3")
    .map(id->Pair.create(id,id+"-NAME"))
    .subscribe(pair->log("subscribe - ",pair.second));
```

+ Truplet
    - Triplet은 Pair에 비해 메모리에 훨씬 많은 오베허드가 있어서 수백만개 크기의 아이템을 작업해야하는 경우에는 피해야 하고 그럴경우 커스텀 클래스를 선호해야 함
```java
    Observable.just("id1","id2","id3")
    .map(id->Triplet.with(id,id+"-NAME",id+"-position"))
    .subscribe(triplet->log("subscribe - ",triplet.getValue0(),triplet.getValue1(),triplet.getValue2()));
```

+ 커스텀 클래스
    - 여러 단계에서 재사용해야하는 복잡한 데이터로 작업한다면 자체 커스텀 클래스를 만드는 것이 좋음
    - 일반 data를 받는 class처럼 만들어주면 됨
```java
    class User{
        String id,name,num;
        public User(String id,String name,String num){
            this.id=id;
            this.name=name;
            this.num=num;
        }
    }
    Observable.just(new User("id1","이름1","num1"))
    .subscribe(user->log(user.id,user.name,user.name));
```

## 아이템 결합
+ 두 비동기 작업을 실행하고 작업 결과를 기다린 후 다음 단계를 수행하기 위해 사용

### zip
+ 옵저버블 아이템을 가져와 하나로 결합하고 새로운 값을 생성하는 작업
+ 다음흐름을 진행하기 전 두 옵저버블이 값을 생성할 때까지 기다림
+ 옵저버블 중 하나가 완료된다면 zip 연산자는 자동으로 다른 것의 구독을 취소함

```java
    Observable.zip(
        Observable.just("test1","test2"),
        Observable.interval(1,TimeUnit.SECONDS),
        (number,interval)->number+"-"+interval
    ).subscribe(e->log(e));
    // just의 완료에 의해 zip이 interval의 구독을 취소(dispose)하더라도 종료되지는(terminate) 않음 
```

### combineLatest
+ zip과 유사하게 여러 옵저버블의 아이템을 병합하는데 사용할 수 있지만 양쪽 모두에서 값이 생성되기를 기다리지 않고 하나가 아이템을 내보낼 때마다 새로운 값 생성
```java
    Observable.combineLatest(
        Observable.just("test1","test2"),
        Observable.interval(1,TimeUnit.SECONDS),
        (number,interval)->number+"-"+interval
    ).subscribe(e->log(e));
    // 기다리지 않기 때문에 interval이 생성되기전에 just옵저버블이 완료될 수가 있음
```

### merge와 concat
+ concat은 앞쪽의 옵저버블이 완료될 때까지 기다리고 완료된 후 다음의 옵저버블이 실행
```java
    Observable.concat(
        Observable.interval(1,TimeUnit.SECONDS),
        Observable.just("test1","test2")
    ).subscribe(e->log(e));
    // interval이 절대 완료되지 않기 때문에 just는 실행되지 않음
```

+ merge는 앞쪽의 옵저버블의 완료를 기다리지 않음
```java
    Observable.merge(
        Observable.interval(1,TimeUnit.SECONDS),
        Observable.just("test1","test2")
    ).subscribe(e->log(e));
```

## 필터링
### filter
+ 지정된 조건과 일치하지 않은 값을 건너 뜀

### distinct
+ 내부적으로 세트를 이용해 아이템의 존재 여부를 확인 가능함

### groupBy
+ 주어진 key함수를 통해 .groupBy()는 새로운 옵저버블 세트를 방출할 것이고 key함수에 의해 반환된 값과 같은 값을 가진 아이템을 포함

```java
    Observable.just(
        new Stock("sam","1"),new Stock("sam","2"),
        new Stock("ka","3"),new Stock("sam","4")
    ).groupBy(Stock::getStockSymbol)//symbol값에 의해 group을 지어줌
    .flatMapSingle(groupedObservable->
        groupedObservalbe.count()//이로인해 sam에 대해서 3출력 ka에 대해서 1이 출력될 것
    ).subscribe(this::log);
```
