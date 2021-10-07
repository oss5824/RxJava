# subject
+ observer나 observable처럼 행동하는 reactiveX의 일부 구현체에서 사용할 수 있는 일종의 교각이나 프록시
+ observer기 때문에 observable을 구독할 수 있고, 또한 observable이기 때문에 항목들을 하나하나 거치면서 재배출하고  관찰하며, 새로운 항목을 동시에 emit(방출)할 수도 있음
+ Hot observable의 대표적인 클래스 중 하나가 Subject 클래스
+ Cold Observable(웹 요청, DB쿼리 등)을 Hot Observable(마우스 이벤트,키보드 이벤트, 센터 데이터 등)로 변환해줌

## AsyncSubject
+ 소스 observable이 배출한 마지막 값만을 배출하고 소스 observable이 complete된 이후에야 동작
+ 소스 observable에서 어느 것도 배출되지 않으면 AsyncSubject역시 아무것도 배출하지 않음
+ 옵저버블이 오류로인해 종료된다면 AsyncSubject에서 아무것도 배출하지 않고 오류 그대로를 전달

## BehaviorSubject
+ observer가 구독하기 시작하면 observer는 소스 observable이 가장 최근에 발행한 아이템이나 기본 값의 발생을 시작하고 그 이후 observable에서 발행된 아이템들을 계속 발행함
+ observable이 오류로 인해 종료되면 아무것도 배출하지 않고 오류 그대로를 전달

## PublishSubject
+ 구독 이후 observable이 배출한 아이템만 observer로 배출
+ create시점에 즉시 항목에 대한 배출이 시작되서 create시점과 구독시작 시점 사이에 배출되는 아이템들은 잃어버릴 수 있음
+ 모든 항목에 대한 보장을 해주기 위해서는 명시적으로 cold observable을 생성하거나 replaysubject를 사용해야 함
+ observable이 오류로 인해 종료되면 아무것도 배출하지 않고 오류 그대로를 전달
+ 안드로이드의 EditText의 자동검색에서 유용하게 사용할 수 있음

## ReplaySubject
+ observer의 구독 시작시점과 상관없이 observable이 배출한 모든 아이템을 observer에게 배출
+ 생성자 오버로드 몇가지를 제공하고 이것을 통해 특정 이상으로 버퍼의 크기가 증가하거나, 첫 배출 이후 지정 시간이 경과한 아이템들에 대한 제거를 해줌
