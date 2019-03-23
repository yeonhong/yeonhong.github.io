---
layout: single
title: "PART 2 - Sequence basics : Reducing a sequence"
related: true
classes: wide
categories: 
  - 프로그래밍
tags:
  - intro to RX
  - Reactive Programming
link: "http://introtorx.com/Content/v1.0.10621.0/05_Filtering.html#Reduction"
---

# Reducing a sequence
우리는 정보화 시대에 살고 있습니다. 놀라운 속도로 데이터가 생성, 저장 및 배포되고 있습니다. 소화 호스에서 직접 마시려고하는 것처럼이 데이터를 소비하는 것이 압도적 일 수 있습니다. 우리는 필요한 데이터를 골라 내고 관련성이없는 데이터를 선택하고 관련성 높은 데이터를 롤업 할 수있는 능력이 필요합니다. 사용자, 고객 및 관리자는 더 높은 성능과 엄격한 마감 시간을 제공하면서 이전보다 많은 데이터로이 작업을 수행해야합니다.

관찰 가능한 시퀀스를 생성하는 방법을 알면, 관찰 가능한 시퀀스를 줄일 수있는 다양한 방법을 살펴 보겠습니다. 시퀀스를 줄이는 연산자를 다음과 같이 분류 할 수 있습니다.

Filter and partition operators
* 같은 수의 요소가있는 시퀀스로 소스 시퀀스 줄이기
Aggregation operators
* 단일 요소를 사용하여 소스 시퀀스를 시퀀스로 줄입니다.
Fold operators
* 소스 시퀀스를 스칼라 값으로 단일 요소로 줄입니다.

스칼라 값으로부터 관찰 가능한 시퀀스를 생성하는 것은 anamorphism으로 정의되거나 'unfold'되는것으로 설명됩니다. 우리는 T에서 I Observable <T>의 anamorphism을 'unfold'된것으로 생각할 수 있습니다. 이것은 "entering the monad"이라고도 할 수 있습니다.이 경우 (이 책의 대부분의 경우) 모나드는 IObservable <T>입니다. 우리가 이제부터 살펴보기 시작하는 것은 궁극적으로 catamorphism 또는 fold로 정의 된 역행렬을 얻는 방법입니다. 다른 인기있는 접기 이름은 'reduce', 'accumulate'및 'inject'입니다.
> 함수형 프로그래밍의 개념으로 생각됨.

## Where
시퀀스에 필터를 적용하는 것은 매우 일반적인 연습이며 가장 일반적인 필터는 Where 절입니다. Rx에서 Where 절을 Where 확장 메서드와 함께 적용 할 수 있습니다. 익숙하지 않은 사용자의 경우 Where 메서드의 서명은 다음과 같습니다.
``` csharp
IObservable<T> Where(this IObservable<T> source, Fun<T, bool> predicate)
```
소스 매개 변수와 리턴 유형이 모두 같습니다. 이것은 유창한 인터페이스를 허용하며, 이는 Rx 및 다른 LINQ 코드 전반에 걸쳐 많이 사용됩니다. 이 예에서는 범위 시퀀스에서 생성 된 모든 짝수 값을 필터링 할 위치를 사용합니다.
``` csharp
var oddNumbers = Observable.Range(0, 10)
  .Where(i => i % 2 == 0)
  .Subscribe(
    Console.WriteLine, 
    () => Console.WriteLine("Completed"));
/* output
0
2
4
6
8
Completed
/*
```
Where 연산자는 많은 표준 LINQ 연산자 중 하나입니다. 이 LINQ 연산자와 다른 LINQ 연산자는 쿼리 연산자의 다양한 구현, 특히 IEnumerable <T> 구현에서 주로 사용됩니다. 대부분의 경우 연산자는 IEnumerable <T> 구현과 마찬가지로 작동하지만 몇 가지 예외가 있습니다. 우리는 각 구현에 대해 논의하고 우리가가는대로 어떤 변화를 설명 할 것입니다. 이러한 공통 연산자를 구현함으로써 Rx는 C # 쿼리 이해 구문을 통해 무료로 언어 지원을받습니다. 그러나이 책의 예제에서는 일관성을 위해 확장 메서드를 계속 사용합니다.

## Distinct and DistinctUntilChanged
대부분의 독자는 IEnumerable <T>의 Where 확장 메서드에 익숙하며 일부는 Distinct 메서드도 알고있을 것입니다. Rx에서, Distinct 메소드가 관찰 가능한 시퀀스에도 사용 가능하게되었습니다. Distinct에 익숙하지 않은 사람들을 위해, 그리고 그것들을 요약하면, **Distinct는 이전에 보지 못한 소스의 값만 전달할 것입니다. (중복제거)**
``` csharp
var subject = new Subject<int>();
var distinct = subject.Distinct();
subject.Subscribe(
  i => Console.WriteLine("{0}", i),
  () => Console.WriteLine("subject.OnCompleted()"));
distinct.Subscribe(
  i => Console.WriteLine("distinct.OnNext({0})", i),
  () => Console.WriteLine("distinct.OnCompleted()"));

subject.OnNext(1);
subject.OnNext(2);
subject.OnNext(3);
subject.OnNext(1);
subject.OnNext(1);
subject.OnNext(4);
subject.OnCompleted();
/* output
1
distinct.OnNext(1)
2
distinct.OnNext(2)
3
distinct.OnNext(3)
1
1
4
distinct.OnNext(4)
subject.OnCompleted()
distinct.OnCompleted()
*/
```
특히 값 1은 3 번 푸시되지만 첫 번째 시간 만 통과한다는 점에 유의하십시오. Distinct에 overloads가있어 항목이 뚜렷한 것으로 판별되는 방식을 전문화 할 수 있습니다. 한 가지 방법은 비교에 사용할 다른 값을 반환하는 함수를 제공하는 것입니다. 여기에서는 사용자 정의 클래스의 속성을 사용하여 값이 고유한지 여부를 정의하는 예제를 살펴 봅니다.
``` csharp
public class Account
{
  public int AccountId { get; set; }
  //... etc
}
public void Distinct_with_KeySelector()
{
  var subject = new Subject<Account>();
  var distinct = subject.Distinct(acc => acc.AccountId);
}
```
제공 될 수있는 keySelector 함수 외에도 IEqualityComparer <T> 인스턴스를 사용하는 오버로드가 있습니다. 이는 유형 T의 인스턴스를 비교하기 위해 재사용 할 수있는 사용자 정의 구현이있는 경우 유용합니다. 마지막으로 keySelector 및 IEqualityComparer <TKey>의 인스턴스를 사용하는 오버로드가 있습니다. 이 경우의 동등 비교자는 T 타입이 아닌 선택된 키 타입 (TKey)을 대상으로합니다.

Rx 특유의 변형 인 DistinctUntilChanged가 있습니다. 이 메서드는 이전 값과 다른 경우에만 값을 나타냅니다. 첫 번째 Distinct 예제를 사용하여 출력 변경 사항을 기록하십시오.
``` csharp
var subject = new Subject<int>();
var distinct = subject.DistinctUntilChanged();
subject.Subscribe(
  i => Console.WriteLine("{0}", i),
  () => Console.WriteLine("subject.OnCompleted()"));
distinct.Subscribe(
  i => Console.WriteLine("distinct.OnNext({0})", i),
  () => Console.WriteLine("distinct.OnCompleted()"));
subject.OnNext(1);
subject.OnNext(2);
subject.OnNext(3);
subject.OnNext(1);
subject.OnNext(1);
subject.OnNext(4);
subject.OnCompleted();
/* output
1
distinct.OnNext(1)
2
distinct.OnNext(2)
3
distinct.OnNext(3)
1
distinct.OnNext(1)
1
4
distinct.OnNext(4)
subject.OnCompleted()
distinct.OnCompleted()
*/
```
두 예제의 차이점은 값 1이 두 번 푸시됩니다. 그러나 소스가 값 1을 푸시하는 세 번째 시간은 두 번째 시간 값 1이 푸시 된 직후입니다. 이 경우 무시됩니다. 내가 함께 작업 한 팀은 이 방법이 시퀀스가 제공 할 수있는 노이즈를 줄이는데 매우 유용하다는 것을 알게되었습니다.
> DistinctUntilChanged의 경우 noti된 직후에 중복된것을 filtering함.

## IgnoreElements
IgnoreElements 확장 메서드는 OnCompleted 또는 OnError 알림을받을 수있는 기발한 작은 도구입니다. 항상 false를 반환하는 조건자를 사용하여 Where 메서드를 사용하여 효과적으로 다시 만들 수 있습니다.
``` csharp
var subject = new Subject<int>();
//Could use subject.Where(_=>false);
var noElements = subject.IgnoreElements();
subject.Subscribe(
  i=>Console.WriteLine("subject.OnNext({0})", i),
  () => Console.WriteLine("subject.OnCompleted()"));
noElements.Subscribe(
  i=>Console.WriteLine("noElements.OnNext({0})", i),
  () => Console.WriteLine("noElements.OnCompleted()"));
subject.OnNext(1);
subject.OnNext(2);
subject.OnNext(3);
subject.OnCompleted();
/* output
subject.OnNext(1)
subject.OnNext(2)
subject.OnNext(3)
subject.OnCompleted()
noElements.OnCompleted()
*/
```
앞에서 제안했듯이 Where를 사용하여 동일한 결과를 얻을 수 있습니다.
``` csharp
subject.IgnoreElements();
//Equivalent to 
subject.Where(value=>false);
//Or functional style that implies that the value is ignored.
subject.Where(_=>false);
```
Where와 IgnoreElements를 떠나기 직전에 코드의 마지막 줄을 빨리 보려고했습니다. **최근까지 저는 개인적으로 '_'이 유효한 변수 이름이라는 것을 인식하지 못했습니다. 그러나 무시 된 매개 변수를 나타 내기 위해 함수 프로그래머가 일반적으로 사용합니다.** 위의 예에 적합합니다. 우리가받는 각 값에 대해 무시하고 항상 false를 반환합니다. 의도는 규칙을 통해 코드의 가독성을 향상시키는 것입니다.

### Skip and Take
필터링의 다른 주요 방법은 매우 유사하므로 우리는 하나의 큰 그룹으로 볼 수 있다고 생각합니다. 먼저 Skip and Take를 살펴 보겠습니다. 이것들은 IEnumerable <T> 구현물처럼 동작합니다. 이것들은 Skip / Take 방법 중 가장 간단하고 아마도 가장 많이 사용됩니다. 두 가지 방법 모두 하나의 매개 변수를 갖습니다. 건너 뛸 값 또는 취할 값의 수.

먼저 Skip을 살펴보면이 예제에서 10 개의 항목으로 구성된 범위 시퀀스가 있고 Skip (3)을 적용합니다.
``` csharp
Observable.Range(0, 10)
  .Skip(3) //처음부터 3개의 값까지 skip.
  .Subscribe(Console.WriteLine, () => Console.WriteLine("Completed"));
/* output
3
4
5
6
7
8
9
Completed
*/
```
처음 세 값 (0, 1 및 2)은 출력에서 모두 무시되었습니다. 또는 Take (3)을 사용하면 반대 결과를 얻습니다. 즉, 처음 세 개의 값만 얻은 다음 Take 연산자가 시퀀스를 완료합니다.
``` csharp
Observable.Range(0, 10)
  .Take(3)
  .Subscribe(Console.WriteLine, () => Console.WriteLine("Completed"));
/* output
0
1
2
Completed
*/
```  
독자를 지나쳐 버린 경우를 대비하여, Take가 계산되면 Take 연산자가 완료됩니다. 무한 시퀀스에 적용하여 이를 증명할 수 있습니다.
``` csharp
Observable.Interval(TimeSpan.FromMilliseconds(100))
  .Take(3)
  .Subscribe(Console.WriteLine, () => Console.WriteLine("Completed"));
/* output
0
1
2
Completed
*/
```

#### SkipWhile and TakeWhile
다음 메소드 세트는 술어가 참으로 평가되는 동안 순서에서 값을 건너 뛰거나 취할 수 있게합니다. 
SkipWhile 연산의 경우, 값이 술어에 실패 할 때까지 모든 값을 필터링하고 나머지 시퀀스를 리턴 할 수 있습니다.
``` csharp
var subject = new Subject<int>();
subject
  .SkipWhile(i => i < 4)
  .Subscribe(Console.WriteLine, () => Console.WriteLine("Completed"));
subject.OnNext(1);
subject.OnNext(2);
subject.OnNext(3);
subject.OnNext(4);
subject.OnNext(3);
subject.OnNext(2);
subject.OnNext(1);
subject.OnNext(0);
subject.OnCompleted();
/* output
4
3
2
1
0
Completed
*/
```
TakeWhile은 술어가 전달되는 동안 모든 값을 리턴하고, 첫 x 째 값이 실패하면 시퀀스가 완료됩니다.
``` csharp
var subject = new Subject<int>();
subject
  .TakeWhile(i => i < 4)
  .Subscribe(Console.WriteLine, () => Console.WriteLine("Completed"));
subject.OnNext(1);
subject.OnNext(2);
subject.OnNext(3);
subject.OnNext(4);
subject.OnNext(3);
subject.OnNext(2);
subject.OnNext(1);
subject.OnNext(0);
subject.OnCompleted();
/* output
1
2
3
Completed
*/
```

#### SkipLast and TakeLast
Skip / Take 및 SkipWhile / TakeWhile을 이해 했으므로이 메소드는 아주 자명 해집니다. 두 가지 방법 모두 건너 뛰거나 걸릴 시퀀스의 끝에 여러 요소가 필요합니다. SkipLast를 구현하면 모든 값을 캐시하고 소스 시퀀스가 완료 될 때까지 기다린 다음 마지막 요소 수를 제외한 모든 값을 재생할 수 있습니다. 그러나 Rx 팀은 그보다 약간 똑똑했습니다. 실제 구현에서는 지정된 수의 알림을 대기열에 넣고 대기열 크기가 값을 초과하면 대기열에서 값이 유출 될 수 있습니다.
``` csharp
var subject = new Subject<int>();
subject
  .SkipLast(2)
  .Subscribe(Console.WriteLine, () => Console.WriteLine("Completed"));
Console.WriteLine("Pushing 1");
subject.OnNext(1);
Console.WriteLine("Pushing 2");
subject.OnNext(2);
Console.WriteLine("Pushing 3");
subject.OnNext(3);
Console.WriteLine("Pushing 4");
subject.OnNext(4);
subject.OnCompleted();
/* output
Pushing 1
Pushing 2
Pushing 3
1
Pushing 4
2
Completed
*/
```
SkipLast와는 달리, TakeLast는 결과를 푸시 (push) 할 수 있도록 소스 시퀀스가 완료되기를 기다려야합니다. 위의 예제에 따라 각 단계에서 프로그램이 수행중인 작업을 나타내는 Console.WriteLine 호출이 있습니다.
``` csharp
var subject = new Subject<int>();
subject
  .SkipLast(2)
  .Subscribe(Console.WriteLine, () => Console.WriteLine("Completed"));
Console.WriteLine("Pushing 1");
subject.OnNext(1);
Console.WriteLine("Pushing 2");
subject.OnNext(2);
Console.WriteLine("Pushing 3");
subject.OnNext(3);
Console.WriteLine("Pushing 4");
subject.OnNext(4);
Console.WriteLine("Completing");
subject.OnCompleted();
/*
Output:
Pushing 1
Pushing 2
Pushing 3
Pushing 4
Completing
3
4
Completed
*/
```

#### SkipUntil and TakeUntil
우리의 마지막 두 가지 방법은 이전에 보았던 방법을 흥미 진진하게 변화시킵니다. 이것들은 우리가 함께 발견 한 두 개의 관찰 가능한 순서가 필요한 처음 두 가지 방법 일 것입니다.

SkipUntil은 보조 관찰 가능 시퀀스에 의해 **값이 생성 될 때까지** 모든 값을 건너 뜁니다.
``` csharp
var subject = new Subject<int>();
var otherSubject = new Subject<Unit>();
subject
  .SkipUntil(otherSubject)
  .Subscribe(Console.WriteLine, () => Console.WriteLine("Completed"));
subject.OnNext(1);
subject.OnNext(2);
subject.OnNext(3);
otherSubject.OnNext(Unit.Default); //생성이후 수신.
subject.OnNext(4);
subject.OnNext(5);
subject.OnNext(6);
subject.OnNext(7);
subject.OnNext(8);
subject.OnCompleted();
/*
Output:
4
5
6
7
Completed
*/
```

분명히, TakeUntil은 그 반대이다. 보조 시퀀스가 값을 생성하면 TakeUntil 연산자가 출력 시퀀스를 완료합니다
``` csharp
var subject = new Subject<int>();
var otherSubject = new Subject<Unit>();
subject
.TakeUntil(otherSubject)
.Subscribe(Console.WriteLine, () => Console.WriteLine("Completed"));
subject.OnNext(1);
subject.OnNext(2);
subject.OnNext(3);
otherSubject.OnNext(Unit.Default);
subject.OnNext(4);
subject.OnNext(5);
subject.OnNext(6);
subject.OnNext(7);
subject.OnNext(8);
subject.OnCompleted();
/*
Output:
1
2
3
Completed
*/
```

Rx에서 사용할 수있는 필터링 방법을 빠르게 실행했습니다. 그것들은 매우 단순하지만, 우리가 보게 될 것처럼, Rx의 힘은 연산자의 구성 가능성에 달려 있습니다.

이 연산자는 Rx 필터링에 대한 좋은 소개를 제공합니다. 필터 운영자는 정보화 시대에 직면 할 수있는 잠재적 인 데이터 홍수를 관리하기위한 첫 번째 단계입니다. 이제는 불일치 데이터를 제거하고 중복 데이터 또는 초과 데이터를 제거하는 방법을 알게되었습니다. 다음으로, 감축 연산자, 검사 및 집계의 다른 두 가지 하위 분류로 넘어갈 것입니다.