---
layout: single
title: "PART 2 - Sequence basics : Aggregation (집합)"
related: true
permalink: /docs/intro to RX/
categories: 
  - 프로그래밍
tags:
  - intro to RX
  - Reactive Programming
link: "http://introtorx.com/Content/v1.0.10621.0/07_Aggregation.html#Aggregation"
---

# Aggregation
데이터는 원시 형식이 항상 귀중한 것은 아닙니다. 때때로 우리는 거대한 데이터를 통합하고, 조합하고, 결합하거나, 응축시켜 더 소비가 가능한 한 입 크기의 덩어리로 만들 필요가 있습니다. 계측, 재무, 신호 처리 및 운영 정보와 같은 영역에서 빠르게 이동하는 데이터를 고려하십시오. 이러한 종류의 데이터는 초당 10 개 이상의 값으로 변할 수 있습니다. 사람이 실제로 이것을 소비 할 수 있습니까? 아마도 사람이 소비하기 때문에 평균, 최소값 및 최대 값과 같은 집계 값을 더 많이 사용할 수 있습니다.

관찰 가능한 시퀀스를 줄이는 것을 계속하면서 Rx에서 사용할 수있는 집계 함수를 살펴 보겠습니다. 우리의 첫 번째 세트는 관찰 가능한 시퀀스를 취하여 하나의 값을 갖는 시퀀스로 줄이기 때문에 마지막 장에서 계속됩니다. 그런 다음 시퀀스를 다시 스칼라 값으로 전환 할 수있는 연산자를 찾습니다.

새로운 연산자를 도입하기 바로 전에 우리는 자체 확장 메서드를 신속하게 만들 것입니다. 샘플을 만드는 데 도움이되는 '덤프'확장 메서드를 사용합니다.
``` csharp
public static class SampleExtentions
{
  public static void Dump<T>(this IObservable<T> source, string name)
  {
    source.Subscribe(
      i=>Console.WriteLine("{0}-->{1}", name, i), 
      ex=>Console.WriteLine("{0} failed-->{1}", name, ex.Message),
      ()=>Console.WriteLine("{0} completed", name));
  }
}
```

## Count
Count는 IEnumerable <T>에서 LINQ를 사용하는 사람들에게 익숙한 확장 메서드입니다. 모든 좋은 메소드 이름과 마찬가지로, "주석에서 말하는 것을 수행합니다". Rx 버전은 스칼라 값이 아닌 관찰 가능한 시퀀스를 반환하므로 Rx 버전은 IEnumerable <T> 버전에서 벗어납니다. 반환 시퀀스에는 소스 시퀀스의 값 개수를 나타내는 단일 값이 있습니다. 소스 시퀀스가 완료 될 때까지 카운트를 제공 할 수 없습니다.
``` csharp
var numbers = Observable.Range(0,3);
numbers.Dump("numbers");
numbers.Count().Dump("count");
/*
Output:

numbers-->1
numbers-->2
numbers-->3
numbers Completed
count-->3
count Completed
*/
```
시퀀스에 32 비트 정수가 보유 할 수있는 값보다 많은 값을 가질 것으로 예상되는 경우 LongCount 확장 메서드를 사용하는 옵션이 있습니다. 이것은 IObservable <long>을 반환한다는 것을 제외하고는 Count와 동일합니다.

## Min, Max, Sum and Average
다른 일반적인 집계는 Min, Max, Sum 및 Average입니다. Count와 마찬가지로이 모든 함수는 단일 값을 갖는 시퀀스를 반환합니다. 소스가 완료되면 결과 시퀀스가 값을 생성 한 다음 완료됩니다.
``` csharp
var numbers = new Subject<int>();
numbers.Dump("numbers");
numbers.Min().Dump("Min");
numbers.Average().Dump("Average");
numbers.OnNext(1);
numbers.OnNext(2);
numbers.OnNext(3);
numbers.OnCompleted();
/*
Output:

numbers-->1
numbers-->2
numbers-->3
numbers Completed
min-->1
min Completed
avg-->2
avg Completed
*/
```
Min 및 Max 메서드에는 사용자 지정 방식으로 값을 정렬하는 IComparer <T>의 사용자 지정 구현을 제공 할 수있는 오버로드가 있습니다. Average 확장 메서드는 특히 시퀀스의 평균 (중앙값 또는 모드와 반대)을 계산합니다. 정수 시퀀스 (int 또는 long)의 경우 Average 출력은 IObservable <double>이됩니다. 소스가 널값을 가질 수있는 정수이면 출력은 IObservable <double?>입니다. 다른 모든 숫자 유형 (float, double, decimal 및 nullable 해당 항목)은 출력 순서가 입력 순서와 동일한 유형이됩니다.

## Functional folds
마지막으로 Rx에서 catamorphism / fold의 기능적 설명을 충족하는 일련의 메소드에 도달합니다. 이 메소드는 IObservable <T>을 취해 T를 생성합니다.

관찰 가능한 순서에 대해 이러한 fold 방법을 사용하면 항상 차단되어야 하므로주의를 기울여야합니다. 메소드를 차단하는 데 주의해야하는 이유는 비동기식 패러다임에서 동기식 패러다임으로 이동하고 UI 및 교착 상태 잠금과 같은 동시성 문제를 일으킬 수 있으므로 주의해야합니다. 동시성을 살펴볼 때 다음 장에서 이러한 문제에 대해 자세히 살펴볼 것입니다.

### First
First () 확장 메서드는 단순히 시퀀스의 첫 번째 값을 반환합니다.
``` csharp
var interval = Observable.Interval(TimeSpan.FromSeconds(3));
//Will block for 3s before returning
Console.WriteLine(interval.First());
```
소스 시퀀스에 값이없는 경우 (즉, 빈 시퀀스 인 경우) First 메서드는 예외를 throw합니다. 다음 세 가지 방법으로이 문제를 해결할 수 있습니다.
* First () 호출에 try / catch 블록을 사용하십시오.
* 대신 Take (1)을 사용하십시오. 그러나 이것은 비동기식이 될 것이고, 논블로킹 방식입니다.
* 대신 FirstOrDefault 확장 메소드 사용

소스가 알림을 생성 할 때까지 FirstOrDefault는 여전히 차단됩니다. 통지가 OnError 인 경우는 Throw됩니다. 알림이 OnNext이면 해당 값이 반환되고, 그렇지 않으면 OnCompleted 인 경우 기본값이 반환됩니다. 이전 방법에서 보았 듯이 기본값을 기본값 (T) (즉 참조 유형의 경우 null 또는 값 유형의 경우 0)으로 사용하는 매개 변수없는 메소드를 사용하도록 선택할 수도 있고, 아니면 자체 기본값을 제공 할 수도 있습니다.

BehaviorSubject와 First () 확장 메소드의 고유 한 관계에 대해 특별히 언급해야합니다. BehaviorSubject가 값, 오류 또는 완료가 될 것이라는 알림이 보장된다는 이유가 있습니다. BehaviorSubject와 함께 사용할 때 첫 번째 확장 메서드의 차단 특성이 효과적으로 제거됩니다. 이것은 행동 주체가 프로퍼티처럼 행동하도록 만드는 데 사용될 수 있습니다.

### Last
Last 및 LastOrDefault는 소스가 완료 될 때까지 차단 한 다음 마지막 값을 반환합니다. First () 메서드와 마찬가지로 OnError 알림도 throw됩니다. 시퀀스가 비어 있으면 Last ()가 InvalidOperationException을 던지지 만 LastOrDefault를 사용하면이 문제를 피할 수 있습니다.

### Single
Single 확장 메서드는 시퀀스에서 단일 값을 가져 오는 메서드입니다. First() 또는 Last()의 차이점은 시퀀스에 단일 값만 포함된다고 가정한다는 것입니다. 메서드는 소스가 값을 생성하고 완료 될 때까지 차단됩니다. 시퀀스가 다른 알림 조합을 생성하면 메서드가 throw합니다. 이 메서드는 단일 값 시퀀스 만 생성하므로 AsyncSubject 인스턴스와 특히 잘 작동합니다.

## Build your own aggregations
제공된 집계가 필요를 충족시키지 못하면 자신 만의 집계를 작성할 수 있습니다. Rx는이를 수행하는 두 가지 다른 방법을 제공합니다.

### Aggregate
Aggregate 메서드를 사용하면 accumulator 함수를 시퀀스에 적용 할 수 있습니다. 기본 overload의 경우 누적 값의 현재 상태와 시퀀스가 밀고있는 값을 취하는 함수를 제공해야합니다. 함수의 결과는 새로운 누적 값입니다. 이 overload 선언은 다음과 같습니다.
``` csharp
IObservable<TSource> Aggregate<TSource>(
  this IObservable<TSource> source, 
  Func<TSource, TSource, TSource> accumulator)
```
int 값에 대한 합계 버전을 생성하려면 accumulator의 현재 상태에 추가하는 함수를 제공하면됩니다.
``` csharp
var sum = source.Aggregate((acc, currentValue) => acc + currentValue);
```

이러한 Aggregate의 overload에는 몇 가지 문제점이 있습니다. 
첫째, 집계 값은 시퀀스 값과 동일한 유형이어야합니다. 우리는 이미 Average와 같은 다른 집계에서 항상 이것을 보았습니다. 
둘째,이 overload는 소스에서 생성 된 값이 적어도 하나 이상 필요합니다. 그렇지 않으면 InvalidOperationException 오류가 발생합니다. Aggregate를 사용하여 빈 시퀀스에서 자체 Count 또는 Sum을 생성하는 것은 완전히 유효합니다. 이렇게하려면 다른 오버로드를 사용해야합니다. 이 오버로드는 seed (추가 매개 변수)를 사용합니다. seed 값은 초기 누적 값을 제공합니다. 또한 집계 유형이 값 유형과 다를 수 있습니다.
``` csharp
IObservable<TAccumulate> Aggregate<TSource, TAccumulate>(
  this IObservable<TSource> source, 
  TAccumulate seed, 
  Func<TAccumulate, TSource, TAccumulate> accumulator)
```
이 오버로드를 사용하기 위해 Sum 구현을 업데이트하는 것은 쉽습니다. 0이 될 시드를 추가하면됩니다. 시퀀스가 비어있을 때 합계로 0을 반환합니다. 이는 우리가 원하는 것입니다. 또한 이제 Count 버전을 만들 수도 있습니다.
``` csharp
var sum = source.Aggregate(0, (acc, currentValue) => acc + currentValue);
var count = source.Aggregate(0, (acc, currentValue) => acc + 1);
//or using '_' to signify that the value is not used.
var count = source.Aggregate(0, (acc, _) => acc + 1);
```

연습으로 집계를 사용하여 자신의 최소 및 최대 방법을 작성하십시오. 아마도 IComparer<T> 인터페이스, 특히 정적 Comparer<T>.Default 속성이 유용 할 것입니다.
``` csharp
public static IObservable<T> MyMin<T>(this IObservable<T> source)
{
  return source.Aggregate(
    (min, current) => Comparer<T>.Default.Compare(min, current) > 0  ? current : min);
}
public static IObservable<T> MyMax<T>(this IObservable<T> source)
{
var comparer = Comparer<T>.Default;
  Func<T, T, T> max = 
    (x, y) =>
    {
      if(comparer.Compare(x, y) < 0)
      {
        return y;
      }
      return x;
    };
  return source.Aggregate(max);
}
```

### Scan
Aggregate는 완료 할 시퀀스의 최종 값을 얻을 수있게 해주지만, 때로는 이것이 우리에게 필요한 것이 아닙니다. 값을 받으면 누적 합계를 얻도록 요구하는 유스 케이스를 고려할 때 집계가 적합하지 않습니다. Aggregate는 무한 시퀀스에도 적합하지 않습니다. 그러나 스캔 확장 방법은이 요구 사항을 완벽하게 충족시킵니다. 스캔 및 Aggregate 모두에 대한 선언은 동일합니다. **차이점은 Scan이 accumulator 기능에 대한 모든 호출에서 결과를 반환 하는것입니다.** 따라서 시퀀스를 단일 값 시퀀스로 축소하는 집계 대신 소스 시퀀스의 각 값에 대해 누적 값을 반환하는 accumulator입니다. 이 예제에서는 누적 합계를 산출합니다.
``` csharp
var numbers = new Subject<int>();
var scan = numbers.Scan(0, (acc, current) => acc + current);
numbers.Dump("numbers");
scan.Dump("scan");
numbers.OnNext(1);
numbers.OnNext(2);
numbers.OnNext(3);
numbers.OnCompleted();
/*
Output:

numbers-->1
sum-->1
numbers-->2
sum-->3
numbers-->3
sum-->6
numbers completed
sum completed
*/
```
TakeLast ()를 사용하여 Scan을 사용하여 집계를 생성하는 것이 좋습니다.
``` csharp
source.Aggregate(0, (acc, current) => acc + current);
//is equivalent to 
source.Scan(0, (acc, current) => acc + current).TakeLast();
```
또 다른 연습으로, 지금까지 책에서 다루었 던 방법을 사용하여 최소 및 최대 실행 최대 시퀀스를 생성하십시오. 여기서 핵심은 현재 누산기보다 작거나 (Max 연산자보다 많음) 값을받을 때마다 해당 값을 밀어 누산기 값을 업데이트해야한다는 것입니다. 그러나 우리는 중복 된 값을 강요하고 싶지는 않습니다. 예를 들어 [2, 1, 3, 5, 0]의 시퀀스가 주어지면 실행중인 최소값에 [2, 1, 0], 실행중인 최대 값에 [2, 3, 5]와 같은 출력이 표시되어야합니다. 우리는 [2, 1, 2, 2, 0] 또는 [2, 2, 3, 5, 5]를보고 싶지 않습니다. 예제 구현을 계속보십시오.

Example of a running minimum:
``` csharp
var comparer = Comparer<T>.Default;
Func<T,T,T> minOf = (x, y) => comparer.Compare(x, y) < 0 ? x: y;
var min = source.Scan(minOf)
.DistinctUntilChanged();
```
Example of a running maximum:
``` csharp
public static IObservable<T> RunningMax<T>(this IObservable<T> source)
{
  return source.Scan(MaxOf)
  .Distinct();
}

private static T MaxOf<T>(T x, T y)
{
  var comparer = Comparer<T>.Default;
  if (comparer.Compare(x, y) < 0)
  {
    return y;
  }
  return x;
}
```
두 예제 간의 유일한 기능적 차이는보다 작음 대신 큰 것을 확인하는 것이지만 예제는 두 가지 스타일을 보여줍니다. 어떤 사람들은 첫 번째 예제의 간결함을 선호하고, 다른 것은 중괄호와 두 번째 예제의 자세한 정보와 같은 것을 선호합니다. 여기서 핵심은 Distinct 또는 DistinctUntilChanged 메서드를 사용하여 Scan 메서드를 작성하는 것이 었습니다. **내부적으로 모든 값의 캐시를 보관하지 않도록 DistinctUntilChanged를 사용하는 것이 좋습니다.**

## Partitioning (분할하기)
Rx는 또한 표준 LINQ 연산자 GroupBy와 같은 기능으로 시퀀스를 분할하는 기능을 제공합니다. 이것은 단일 시퀀스를 취하고 많은 구독자에게 퍼지거나 파티션에서 집계를 사용하는 데 유용 할 수 있습니다.

### MinBy and MaxBy
MinBy 및 MaxBy 연산자를 사용하면 키 선택기 함수를 기반으로 시퀀스를 분할 할 수 있습니다. 키 선택기 함수는 IEnumerable <T> ToDictionary 또는 GroupBy 및 Distinct 메서드와 같은 다른 LINQ 연산자에서 일반적입니다. 각 메소드는 키의 값을 최소값 또는 최대 값으로 반환합니다.
``` csharp
// Returns an observable sequence containing a list of zero or more elements that have a 
//  minimum key value.
public static IObservable<IList<TSource>> MinBy<TSource, TKey>(
  this IObservable<TSource> source, 
  Func<TSource, TKey> keySelector)
{...}
public static IObservable<IList<TSource>> MinBy<TSource, TKey>(
  this IObservable<TSource> source, 
  Func<TSource, TKey> keySelector, 
  IComparer<TKey> comparer)
{...}
// Returns an observable sequence containing a list of zero or more elements that have a
//  maximum key value.
public static IObservable<IList<TSource>> MaxBy<TSource, TKey>(
  this IObservable<TSource> source, 
  Func<TSource, TKey> keySelector)
{...}
public static IObservable<IList<TSource>> MaxBy<TSource, TKey>(
  this IObservable<TSource> source, 
  Func<TSource, TKey> keySelector, 
  IComparer<TKey> comparer)
{...}
```
각 Min 및 Max 연산자에는 비교자를 사용하는 오버로드가 있습니다. 이렇게하면 사용자 정의 유형을 비교하거나 표준 유형을 사용자 정의하여 정렬 할 수 있습니다.

0부터 10까지의 시퀀스를 고려하십시오. 모듈러를 기준으로 그룹에 값을 분할하는 키 선택기를 적용하면 3 개의 값 그룹이 생깁니다. 값과 키는 다음과 같습니다.
``` csharp
Func<int, int> keySelector = i => i % 3;
```
* 0, key: 0
* 1, key: 1
* 2, key: 2
* 3, key: 0
* 4, key: 1
* 5, key: 2
* 6, key: 0
* 7, key: 1
* 8, key: 2
* 9, key: 0

여기에서 최소 키가 0이고 최대 키가 2임을 알 수 있습니다. 따라서 MinBy 연산자를 적용하면 시퀀스의 단일 값은 [0,3,6,9]의 목록이됩니다. MaxBy 연산자를 적용하면 목록 [2,5,8]이 생성됩니다. MinBy 및 MaxBy 연산자는 AsyncSubject와 같은 단일 값만 생성하며이 값은 0 이상의 값을 갖는 IList <T>가됩니다.

최소 / 최대 키 값 대신에 각 키의 최소값을 얻고 싶다면 GroupBy를 살펴 봐야합니다.

### GroupBy
GroupBy 연산자를 사용하면 IEnumerable <T>의 GroupBy 연산자처럼 시퀀스를 분할 할 수 있습니다. IEnumerable <T> 연산자가 IEnumerable <IGKeying <TKey, T>>를 반환하는 것과 비슷한 방식으로 IObservable <Group> GroupBy 연산자는 IObservable <IGroupedObservable <TKey, T>를 반환합니다.
``` csharp
// Transforms a sequence into a sequence of observable groups, 
//  each of which corresponds to a unique key value, 
//  containing all elements that share that same key value.
public static IObservable<IGroupedObservable<TKey, TSource>> GroupBy<TSource, TKey>(
  this IObservable<TSource> source, 
  Func<TSource, TKey> keySelector)
{...}
public static IObservable<IGroupedObservable<TKey, TSource>> GroupBy<TSource, TKey>(
  this IObservable<TSource> source, 
  Func<TSource, TKey> keySelector, 
  IEqualityComparer<TKey> comparer)
{...}
public static IObservable<IGroupedObservable<TKey, TElement>> GroupBy<TSource, TKey, TElement>(
  this IObservable<TSource> source, 
  Func<TSource, TKey> keySelector, 
  Func<TSource, TElement> elementSelector)
{...}
public static IObservable<IGroupedObservable<TKey, TElement>> GroupBy<TSource, TKey, TElement>(
  this IObservable<TSource> source, 
  Func<TSource, TKey> keySelector, 
  Func<TSource, TElement> elementSelector, 
  IEqualityComparer<TKey> comparer)
{...}
```
같은 기능을 얻기 위해 쿼리에 Select 연산자를 쉽게 구성 할 수 있기 때문에 마지막 두 개의 오버로드가 약간 중복됨을 알 수 있습니다.

IGrouping <TKey, T> 형식이 IEnumerable <T>을 확장하는 것과 비슷한 방식으로 IGroupedObservable <T>은 Key 속성을 추가하여 IObservable <T>을 확장합니다. GroupBy의 사용은 효과적으로 우리에게 중첩 된 관찰 가능한 시퀀스를 제공합니다.

GroupBy 연산자를 사용하여 각 키의 최소 / 최대 값을 얻으려면 먼저 시퀀스를 분할 한 다음 각 파티션을 최소 / 최대로 분할 할 수 있습니다.
``` csharp
var source = Observable.Interval(TimeSpan.FromSeconds(0.1)).Take(10);
var group = source.GroupBy(i => i % 3);
group.Subscribe(
  grp => 
    grp.Min().Subscribe(
      minValue => 
      Console.WriteLine("{0} min value = {1}", grp.Key, minValue)),
  () => Console.WriteLine("Completed"));
```
위의 코드는 작동하지만 이러한 중첩 구독 호출을 사용하는 것은 좋지 않습니다. 우리는 중첩 구독에 대한 통제권을 잃었으며 읽기가 어렵습니다. 중첩 된 가입을 만들 때 더 나은 패턴을 적용하는 방법을 고려해야합니다. 이 경우 우리는 다음 장에서 살펴볼 SelectMany를 사용할 수 있습니다.
```  csharp
var source = Observable.Interval(TimeSpan.FromSeconds(0.1)).Take(10);
var group = source.GroupBy(i => i % 3);
group.SelectMany(grp => grp.Max().Select(value => new { grp.Key, value })).Dump("group");
```

### Nested observables
The concept of a sequence of sequences은 처음에는 다소 압도적 일 수 있습니다. 특히 두 시퀀스 유형 모두가 IObservable 인 경우 특히 그렇습니다. 고급 주제이지만 Rx에서 흔히 볼 수있는 주제이므로 여기에서 다루겠습니다. 개념을 더 잘 이해하기 위해 시나리오 나 예제를 개념화 할 수 있다면 더 쉽습니다.

Examples of Observables of Observables:

* 데이터 파티션
  * 단일 소스에서 데이터를 분할하여 여러 소스로 쉽게 필터링하고 공유 할 수 있습니다. 우리가 보았 듯이 파티셔닝 데이터는 집계에도 유용 할 수 있습니다. 이는 일반적으로 GroupBy 연산자로 수행됩니다.
* 온라인 게임 서버
  * 일련의 서버를 생각해보십시오. 새 값은 온라인 상태의 서버를 나타냅니다. 값 자체는 일련의 대기 시간 값으로 소비자는 사용 가능한 서버의 수량과 품질에 대한 실시간 정보를 볼 수 있습니다. 서버가 다운 된 경우 내부 시퀀스는 완료하여 이를 나타낼 수 있습니다.
* 재무 데이터 스트림
  * 새로운 시장이나 도구가 하루 종일 개장 할 수 있습니다. 그런 다음 가격 정보를 스트리밍하고 시장이 끝날 때 완료 할 수 있습니다.
* 채팅방
  * 사용자는 채팅 (외부 순서), 메시지 남기기 (내부 순서) 및 채팅 종료 (내부 순서 완료)에 참여할 수 있습니다.
* 파일 감시자
  * 파일이 디렉토리에 추가되면 수정 사항 (외부 순서)을 볼 수 있습니다. 내부 시퀀스는 파일에 대한 변경을 나타낼 수 있으며 내부 시퀀스를 완료하면 파일을 삭제할 수 있습니다.

이 예제를 고려해 볼 때, 중첩 관측 가능 객체의 개념을 갖는 것이 얼마나 유용 할 수 있는지를 알 수 있습니다. SelectMany, Merge 및 Switch와 같은 중첩 된 관찰 가능 객체와 잘 작동하는 연산자 집합이 있습니다.

중첩된 observables을 사용하여 작업 할 때 새 시퀀스가 생성을 나타내는 규칙을 채택하는 것이 편리 할 수 있습니다 (예 : 새 파티션이 생성되고 새로운 게임 호스트가 온라인 상태가되고 시장이 열리고 사용자가 채팅에 참여하고 감시중인 폴더에서 파일을 생성하는 등) 완료된 내부 시퀀스가 나타내는 내용에 대해 규칙을 적용 할 수 있습니다 (예 : 게임 호스트가 오프라인 상태가 됨, 시장 폐쇄, 사용자 이탈 채팅, 관찰중인 파일이 삭제됨). 내포된 observables을 가진 위대한 점은 완성 된 내부 시퀀스가 새로운 내부 시퀀스를 생성함으로써 효과적으로 다시 시작될 수 있다는 것입니다.

이 장에서는 LINQ의 기능과 Rx에 대한 적용 방법을 알아보기 시작합니다. 우리는 다른 방법들이 이미 제공 한 효과를 재창조하기 위해 함께 방법들을 묶었습니다. 이것이 학문적으로 좋은 동안, 그것은 또한 우리가 기능적 구성의 관점에서 생각하기 시작하도록 허락합니다. First () + BehaviorSubject (T), Single () + AsyncSubject (T), Single () + Aggregate () 등 몇 가지 메소드가 특정 유형에서 잘 작동하는 것을 보았습니다. 연산자, catamorphism. 다음으로 우리는 기능 구성 툴 벨트에 추가 할 수있는 더 많은 방법을 발견하고 Rx가 우리의 제 3의 기능 개념 인 묶는 방법을 찾아 낼 것입니다.

데이터를 그룹 및 집계로 통합하면 대량 데이터를 합리적으로 소비 할 수 있습니다. 빠른 데이터 이동은 일괄 처리 시스템 및 사람의 소비를 위해 너무 압도적 일 수 있습니다. Rx는 즉석에서 집계 및 파티션을 수행 할 수있는 기능을 제공하므로 값 비싼 CEP 또는 OLAP 제품을 사용하지 않고도 실시간보고가 가능합니다.