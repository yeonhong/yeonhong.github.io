---
layout: single
title: "PART 2 - Sequence basics : Transformation of sequences"
related: false
classes: wide
permalink: /docs/intro to RX/
categories: 
  - 프로그래밍
tags:
  - intro to RX
  - Reactive Programming
link: "http://introtorx.com/Content/v1.0.10621.0/08_Transformation.html#TransformationOfSequences"
---

# Transformation of sequences
우리가 소비하는 순서의 값이 항상 필요한 형식이 아닙니다. 때로는 데이터에 너무 많은 노이즈가있어 값을 제거합니다. 때로는 각 값을 더 풍부한 객체 또는 더 많은 값으로 확장해야 할 수도 있습니다. 연산자를 작성함으로써, Rx는 사용자가 관찰 할 수있는 시퀀스의 값의 양과 품질을 제어 할 수 있습니다.

지금까지 시퀀스의 생성, 시퀀스로의 전환, 필터링, 집계 또는 폴딩을 통한 시퀀스 감소에 대해 살펴 보았습니다. 이 장에서는 변형 시퀀스를 살펴 보겠습니다. 이를 통해 우리는 세 번째 범주의 기능적 메소드 인 bind를 소개 할 수 있습니다. Rx의 바인드 함수는 시퀀스를 취하여 각 요소에 대한 일련의 변환을 적용하여 새로운 시퀀스를 생성합니다.

* Ana(morphism) T --> IObservable<T>
* Cata(morphism) IObservable<T> --> T
* Bind IObservable<T1> --> IObservable<T2>

이제 우리는 3 가지 고차 함수를 모두 소개 했으므로 이미 알고 있습니다. Bind와 Cata (morphism)는 Google의 MapReduce 프레임 워크로 유명 해졌습니다. 여기에서 Google은 Bind와 Cata를 아마도 더 일반적인 별칭으로 참조합니다. map 과 reduce로.

우리의 용어를 고차원 함수의 ABC로 기억하는 것이 도움이 될 수 있습니다.
* Ana는 시퀀스를 입력합니다. T -> IObservable<T>
* Bind는 시퀀스를 수정합니다. IObservable<T1> -> IObservable<T2>
* Cata는 시퀀스를 떠납니다. IObservable<T> -> T

## Select
기본 변환 방법은 선택입니다. TSource의 값을 취하고 TResult의 값을 반환하는 함수를 제공 할 수 있습니다. Select의 시그니처는 멋지고 간단하며 가장 일반적인 용도는 한 유형에서 다른 유형으로 변환하는 것, 즉 IObservable <TSource>를 IObservable <TResult>로 변환하는 것입니다.
``` csharp
IObservable<TResult> Select<TSource, TResult>(
  this IObservable<TSource> source, 
  Func<TSource, TResult> selector)
```
TSource와 TResult가 동일한 것을 방지하는 제한이 없습니다. 첫 번째 예제에서는 일련의 정수를 취하여 3을 더하여 각 값을 변환하여 다른 정수 시퀀스를 생성합니다.
``` csharp
var source = Observable.Range(0, 5);
  source.Select(i=>i+3)
  .Dump("+3")
/*
Output:

+3-->3
+3-->4
+3-->5
+3-->6
+3-->7
+3 completed
*/
```
이 방법이 유용 할 수 있지만 일반적으로 한 유형에서 다른 유형으로 값을 변환하는 것이 일반적입니다. 이 예제에서는 정수 값을 문자로 변환합니다.
``` csharp
Observable.Range(1, 5);
  .Select(i =>(char)(i + 64))
  .Dump("char");
/*
Output:

char-->A
char-->B
char-->C
char-->D
char-->E
char completed
*/
```
우리가 정말로 LINQ를 이용하기 원한다면 정수 시퀀스를 일련의 익명 형식으로 변환 할 수 있습니다.
``` csharp
Observable.Range(1, 5)
  .Select(
  i => new { Number = i, Character = (char)(i + 64) })
  .Dump("anon");
/*
Output:

anon-->{ Number = 1, Character = A }
anon-->{ Number = 2, Character = B }
anon-->{ Number = 3, Character = C }
anon-->{ Number = 4, Character = D }
anon-->{ Number = 5, Character = E }
anon completed
*/
```
LINQ를 더 활용하기 위해 위의 쿼리를 쿼리 이해 구문을 사용하여 작성할 수 있습니다.
``` csharp
var query = from i in Observable.Range(1, 5)
  select new {Number = i, Character = (char) (i + 64)};
  query.Dump("anon");
```
Rx에서 Select에는 또 다른 overload가 있습니다. 두 번째 overload는 Select함수에 두 개의 값을 제공합니다. 추가 인수는 시퀀스의 요소 인덱스입니다. 시퀀스의 요소 인덱스가 Select함수에서 중요한 경우이 메서드를 사용합니다.

## Cast and OfType
IObservable <object> 객체의 시퀀스를 가져 오는 경우 유용하지 않을 수 있습니다. 주어진 형식으로 각 요소를 캐스팅하는 IObservable <object>에 대한 메소드가 있으며, 논리적으로는 Cast <T> ()입니다.
``` csharp
var objects = new Subject<object>();
objects.Cast<int>().Dump("cast");
objects.OnNext(1);
objects.OnNext(2);
objects.OnNext(3);
objects.OnCompleted();
/*
Output:

cast-->1
cast-->2
cast-->3
cast completed
*/
```
그러나 시퀀스에 캐스트 할 수없는 값을 추가하면 오류가 발생합니다.
``` csharp
var objects = new Subject<object>();
objects.Cast<int>().Dump("cast");
objects.OnNext(1);
objects.OnNext(2);
objects.OnNext("3");//Fail
/*
Output:

cast-->1
cast-->2
cast failed -->Specified cast is not valid.
*/
```
고맙게도 이것이 우리가 원하는 것이 아니라면 OfType<T>() 대체 확장 방법을 사용할 수 있습니다.
``` csharp
var objects = new Subject<object>();
objects.OfType<int>().Dump("OfType");
objects.OnNext(1);
objects.OnNext(2);
objects.OnNext("3");//Ignored
objects.OnNext(4);
objects.OnCompleted();
/*
Output:

OfType-->1
OfType-->2
OfType-->4
OfType completed
*/
```
이들이 가지고있는 편리한 방법이지만, 우리가 이미 알고있는 연산자로 만들 수 있다고 말하는 것은 공평합니다.
``` csharp
//source.Cast<int>(); is equivalent to
source.Select(i=>(int)i);
//source.OfType<int>();
source.Where(i=>i is int).Select(i=>(int)i);
```

## Timestamp and TimeInterval
관찰 가능한 시퀀스는 비동기 적이기 때문에 요소가 수신되는 시간을 파악하는 것이 편리 할 수 있습니다. Timestamp 확장 메서드는 시퀀스의 요소를 경량 Timestamped <T> 구조로 래핑하는 편리한 편리한 메서드입니다. Timestamped <T> 형식은 래핑하는 요소의 값과 DateTimeOffset으로 만든 Timestamp를 노출하는 구조체입니다.

이 예제에서 우리는 3 초 간격으로 3 개의 값으로 구성된 시퀀스를 생성 한 다음 타임 스탬프 된 시퀀스로 변환합니다. Timestamped <T>에서 ToString ()을 편리하게 구현하면 읽을 수있는 결과를 얻을 수 있습니다.
``` csharp
Observable.Interval(TimeSpan.FromSeconds(1))
  .Take(3)
  .Timestamp()
  .Dump("TimeStamp");
/*
Output

TimeStamp-->0@01/01/2012 12:00:01 a.m. +00:00
TimeStamp-->1@01/01/2012 12:00:02 a.m. +00:00
TimeStamp-->2@01/01/2012 12:00:03 a.m. +00:00
TimeStamp completed
*/
```
0, 1, 2 값은 각각 1 초 간격으로 생성된다는 것을 알 수 있습니다. TimeStamp을 얻는 대신 **마지막 요소가 찍힌 이후로 간격을 가져 올 수 있습니다.** TimeInterval 확장 메서드는이를 제공합니다. Timestamp 메서드에 따라 요소는 경량 구조로 래핑됩니다. 이 때 구조는 TimeInterval <T> 유형입니다.
``` csharp
Observable.Interval(TimeSpan.FromSeconds(1))
  .Take(3)
  .TimeInterval()
  .Dump("TimeInterval");
/*
Output:

TimeInterval-->0@00:00:01.0180000
TimeInterval-->1@00:00:01.0010000
TimeInterval-->2@00:00:00.9980000
TimeInterval completed
*/
```
출력에서 볼 수 있듯이 타이밍은 정확히 1 초가 아니지만 꽤 가깝습니다.

## Materialize and Dematerialize
