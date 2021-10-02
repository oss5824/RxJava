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
