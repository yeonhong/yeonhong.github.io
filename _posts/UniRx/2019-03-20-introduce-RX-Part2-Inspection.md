---
layout: single
title: "PART 2 - Sequence basics : Inspection (검사)"
related: true
permalink: /docs/intro to RX/
categories: 
  - 프로그래밍
tags:
  - intro to RX
  - Reactive Programming
link: "http://introtorx.com/Content/v1.0.10621.0/06_Inspection.html#Inspection"
---

# Inspection
우리가 소비하는 모든 데이터를 이해하는 것은 항상 중복되고 불필요한 것을 필터링하는 것만은 아닙니다. 때로는 시퀀스가 우리의 기대치를 충족시킬 수있는 데이터를 추출해야합니다. 이 데이터가이 사양을 충족시키는 값을 가지고 있습니까? 시퀀스의 특정 값입니까? 시퀀스에서 특정 가치를 얻으십시오!

지난장에서는 다양한 필터를 통해 관찰 가능한 시퀀스를 줄이는 일련의 방법을 살펴 보았습니다. 다음으로 검사 기능을 제공하는 연산자를 살펴 보겠습니다. 이 연산자의 대부분은 관찰 가능 시퀀스를 단일 값을 갖는 시퀀스로 줄입니다. 이러한 메소드의 반환 값은 스칼라가 아니기 때문에 (여전히 IObservable <T>) 이러한 메소드는 실제로 catamorphism에 대한 정의를 만족시키지 않지만 시퀀스를 단일 값으로 줄이기위한 검사에 적합합니다.

**다음에 살펴볼 일련의 메소드는 주어진 시퀀스를 검사하는 데 유용합니다. 각각은 결과를 포함하는 단일 값으로 관찰 가능한 시퀀스를 반환합니다**. 이것은 본성 상 비동기 적이기 때문에 유용합니다. 그들은 모두 매우 간단하므로 우리는 그들 각각에 대해 간략하게 설명 할 것입니다.

## Any
먼저 확장 메서드 Any에 대한 매개 변수없는 오버로드를 살펴볼 수 있습니다. 소스가 값없이 완료되면 단일 값 false를 갖는 관찰 가능 시퀀스를 리턴합니다. 그러나 소스가 값을 생성하는 경우 첫 번째 값이 생성되면 결과 시퀀스가 즉시 true로 설정되고 완료됩니다. 첫 번째 알림이 도착할때 오류가 발생하면 해당 오류가 전달됩니다.
``` csharp
var subject = new Subject<int>();
subject.Subscribe(Console.WriteLine, () => Console.WriteLine("Subject completed"));
var any = subject.Any();
any.Subscribe(b => Console.WriteLine("The subject has any values? {0}", b));
subject.OnNext(1);
subject.OnCompleted();
/*
Output:
1
The subject has any values? True
subject completed
*/
```
OnNext (1)을 제거하면 출력이 다음과 같이 변경됩니다.
``` csharp
// subject completed
// The subject has any values? False
```
소스 에러의 경우, 최초의 통지의 경우 만 흥미가있다. 그렇지 않은 경우, Any 메소드는 벌써 true로 설정되어있다. 첫 번째 알림이 오류 인 경우 Any는 OnError 알림으로 전달합니다.
``` csharp
var subject = new Subject<int>();
subject.Subscribe(Console.WriteLine,
  ex => Console.WriteLine("subject OnError : {0}", ex),
  () => Console.WriteLine("Subject completed"));
var any = subject.Any();
any.Subscribe(b => Console.WriteLine("The subject has any values? {0}", b),
  ex => Console.WriteLine(".Any() OnError : {0}", ex),
  () => Console.WriteLine(".Any() completed"));
subject.OnError(new Exception());
/*
Output:
subject OnError : System.Exception: Fail
.Any() OnError : System.Exception: Fail
*/
```
또한 Any 메서드에는 조건자를 사용하는 오버로드가 있습니다. 이것은 효과적으로 그것을 Any가 붙은 Where로 만듭니다.
``` csharp
subject.Any(i => i > 2);
//Functionally equivalent to 
subject.Where(i => i > 2).Any();
```

Any를 Observable.Create로 만들었을때 아래와 같이 할 수 있습니다. (조건자들은 사실 만들수 있다는걸 보여주는듯.)
``` csharp
public static IObservable<bool> MyAny<T>(
  this IObservable<T> source)
{
  return Observable.Create<bool>(
  o =>
  {
    var hasValues = false;
    return source
      .Take(1)
      .Subscribe(
        _ => hasValues = true,
        o.OnError,
        () =>
        {
          o.OnNext(hasValues);
          o.OnCompleted();
        });
  });
}
public static IObservable<bool> MyAny<T>(
  this IObservable<T> source, 
  Func<T, bool> predicate)
{
  return source
    .Where(predicate)
    .MyAny();
}
```

## All
All () 확장 메서드는 모든 값이 술어를 충족시켜야한다는 점을 제외하고는 Any 메서드와 마찬가지로 작동합니다. 값이 술어를 충족시키지 않으면 false 값이 리턴되고 출력 시퀀스가 완료됩니다. 소스가 비어 있으면 All이 해당 값으로 true를 푸시합니다. Any 메서드 처럼, 오류가 All 메서드의 구독자에게 전달됩니다.
``` csharp
var subject = new Subject<int>();
subject.Subscribe(Console.WriteLine, () => Console.WriteLine("Subject completed"));
var all = subject.All(i => i < 5);
all.Subscribe(b => Console.WriteLine("All values less than 5? {0}", b));
subject.OnNext(1);
subject.OnNext(2);
subject.OnNext(6);
subject.OnNext(2);
subject.OnNext(1);
subject.OnCompleted();
/*
Output:
1
2
6
All values less than 5? False
all completed
2
1
subject completed
*/
```
Rx의 얼리 어답터는 IsEmpty 확장 방법이 없음을 알 수 있습니다. All 확장 메서드를 사용하여 누락 된 메서드를 쉽게 복제 할 수 있습니다
``` csharp
//IsEmpty() is deprecated now.
//var isEmpty = subject.IsEmpty();
var isEmpty = subject.All(_ => false);
```

## Contains
Contains 확장 메소드 오버로드는 모든 확장 메소드에 오버로드가 될 수 있습니다. Contains 확장 메서드는 Any와 동일한 동작을하지만, 특히 조건 자 사용 대신 IComparable 사용을 대상으로하며 조건 자에 맞는 값 대신 특정 값을 찾도록 설계되었습니다. 나는 이것이 IEnumerable과의 일관성을 위해 Any의 오버로드가 아니라고 생각한다.
``` csharp
var subject = new Subject<int>();
subject.Subscribe(
  Console.WriteLine, 
  () => Console.WriteLine("Subject completed"));
var contains = subject.Contains(2);
contains.Subscribe(
  b => Console.WriteLine("Contains the value 2? {0}", b),
  () => Console.WriteLine("contains completed"));
subject.OnNext(1);
subject.OnNext(2);
subject.OnNext(3);
subject.OnCompleted();
/*
Output:
1
2
Contains the value 2? True
contains completed (조건에 맞는게 보이는 즉시 완료)
3
Subject completed
*/
```
또한 형식에 대한 기본값 이외의 IEqualityComparer <T> 구현을 지정할 수 있도록 Contains에 대한 오버로드가 있습니다. This can be useful if you have a sequence of custom types that may have some special rules for equality depending on the use case.

## DefaultIfEmpty
DefaultIfEmpty 확장 메서드는 소스 시퀀스가 비어 있으면 단일 값을 반환합니다. 사용 된 과부하에 따라 기본값으로 제공된 값 또는 Default(T)이됩니다. Default(T)은 구조체 유형의 경우 0이며 클래스의 경우 null입니다. 소스가 비어 있지 않으면 모든 값이 똑바로 전달됩니다.

이 예제에서 소스는 값을 생성하므로 DefaultIfEmpty의 결과는 소스입니다.
``` csharp
var subject = new Subject<int>();
subject.Subscribe(
  Console.WriteLine,
  () => Console.WriteLine("Subject completed"));
var defaultIfEmpty = subject.DefaultIfEmpty();
defaultIfEmpty.Subscribe(
  b => Console.WriteLine("defaultIfEmpty value: {0}", b),
  () => Console.WriteLine("defaultIfEmpty completed"));
subject.OnNext(1);
subject.OnNext(2);
subject.OnNext(3);
subject.OnCompleted();
/* output
1
defaultIfEmpty value: 1
2
defaultIfEmpty value: 2
3
defaultIfEmpty value: 3
Subject completed
defaultIfEmpty completed
*/
```
소스가 비어있는 경우 유형의 기본값 (예 : int의 경우 0)을 사용하거나, 지정된 값을 제공 합니다.
``` csharp
var subject = new Subject<int>();
subject.Subscribe(
  Console.WriteLine,
  () => Console.WriteLine("Subject completed"));
var defaultIfEmpty = subject.DefaultIfEmpty();
defaultIfEmpty.Subscribe(
  b => Console.WriteLine("defaultIfEmpty value: {0}", b),
  () => Console.WriteLine("defaultIfEmpty completed"));
var default42IfEmpty = subject.DefaultIfEmpty(42);
default42IfEmpty.Subscribe(
  b => Console.WriteLine("default42IfEmpty value: {0}", b),
  () => Console.WriteLine("default42IfEmpty completed"));
subject.OnCompleted();
/*
Output:

Subject completed
defaultIfEmpty value: 0
defaultIfEmpty completed
default42IfEmpty value: 42
default42IfEmpty completed
*/
```
## ElementAt
ElementAt 확장 메서드를 사용하면 지정된 인덱스에서 값을 "체리픽"할 수 있습니다. IEnumerable <T> 버전과 마찬가지로 0 기반 인덱스를 사용합니다.
``` csharp
var subject = new Subject<int>();
subject.Subscribe(
  Console.WriteLine,
  () => Console.WriteLine("Subject completed"));
var elementAt1 = subject.ElementAt(1);
  elementAt1.Subscribe(
  b => Console.WriteLine("elementAt1 value: {0}", b),
  () => Console.WriteLine("elementAt1 completed"));
subject.OnNext(1);
subject.OnNext(2);
subject.OnNext(3);
subject.OnCompleted();
/*
Output

1
2
elementAt1 value: 2
elementAt1 completed
3
subject completed
*/
```
관찰 가능한 시퀀스의 길이를 검사 할 수 없기 때문에 때때로이 방법이 문제를 일으킬 수 있다고 가정하는 것이 적절합니다. 소스 시퀀스가 5 개의 값만 생성하고 ElementAt (5)를 요청하면 소스가 완료되면 결과 시퀀스가 ArgumentOutOfRangeException 오류와 함께 오류가 발생합니다. 우리에게는 다음 세 가지 옵션이 있습니다.
* OnError를 정상적으로 처리하십시오.
* .Skip (5) .Take (1); 이것은 처음 5 개의 값을 무시하고 오직 6 번째 값만을 취합니다. 시퀀스의 요소 수가 6 개 미만인 경우 빈 시퀀스 만 얻을 수 있지만 오류는 발생하지 않습니다.
* ElementAtOrDefault 사용

ElementAtOrDefault 확장 메소드는 인덱스가 범위를 벗어나는 경우 기본값 (T) 값을 눌러 보호합니다. 현재 자신의 기본값을 제공하는 옵션이 없습니다.

## SequenceEqual
마지막으로 SequenceEqual 확장 메서드는 아마도 catamorphism과 fold에 대한 이야기를 시작하는 장에 넣을 스트레칭이지만 검사의 주제에 대해 잘 작동합니다. 이 방법을 사용하면 두 개의 관찰 가능한 시퀀스를 비교할 수 있습니다. 각 소스 시퀀스가 값을 생성 할 때, 다른 시퀀스의 캐시와 비교되어 각 시퀀스가 동일한 순서로 동일한 값을 가지며 시퀀스가 동일한 길이가되도록합니다. 즉, 소스 시퀀스가 분기 값을 생성하자 마자 결과 시퀀스가 false를 반환하거나 두 소스가 같은 값으로 완료되면 true를 반환 할 수 있습니다.
``` csharp
var subject1 = new Subject<int>();
subject1.Subscribe(
  i=>Console.WriteLine("subject1.OnNext({0})", i),
  () => Console.WriteLine("subject1 completed"));
var subject2 = new Subject<int>();
subject2.Subscribe(
  i=>Console.WriteLine("subject2.OnNext({0})", i),
  () => Console.WriteLine("subject2 completed"));
var areEqual = subject1.SequenceEqual(subject2);
areEqual.Subscribe(
  i => Console.WriteLine("areEqual.OnNext({0})", i),
  () => Console.WriteLine("areEqual completed"));
subject1.OnNext(1);
subject1.OnNext(2);
subject2.OnNext(1);
subject2.OnNext(2);
subject2.OnNext(3);
subject1.OnNext(3);
subject1.OnCompleted();
subject2.OnCompleted();
/*
Output:

subject1.OnNext(1)
subject1.OnNext(2)
subject2.OnNext(1)
subject2.OnNext(2)
subject2.OnNext(3)
subject1.OnNext(3)
subject1 completed
subject2 completed
areEqual.OnNext(True)
areEqual completed
*/
```

이 장에서는 관찰 가능한 시퀀스를 검사 할 수있는 일련의 메소드에 대해 설명했습니다. 각각의 결과는 일반적으로 단일 값을 갖는 시퀀스를 반환합니다. 우리는 파악하기 어려운 기능적 폴드 기능을 발견 할 때까지 시퀀스를 줄이는 방법을 계속 검토 할 것입니다.