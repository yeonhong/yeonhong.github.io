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

## Materialize and Dematerialize (구체화?)
Timestamp 및 TimeInterval 변환 연산자는 시퀀스 로깅 및 디버깅에 유용 할 수 있으므로 Materialize 연산자도 유용합니다. Materialize는 IObservable <T>를 IObservable <Notification <T>로 가져 와서 시퀀스를 시퀀스의 메타 데이터 표현으로 전환합니다. 알림 유형은 시퀀스 이벤트에 대한 메타 데이터를 제공합니다.

시퀀스를 구체화하면 래핑 된 값이 반환되는 것을 볼 수 있습니다.
``` csharp
Observable.Range(1, 3)
.Materialize()
.Dump("Materialize");
/*
Output:
Materialize-->OnNext(1)
Materialize-->OnNext(2)
Materialize-->OnNext(3)
Materialize-->OnCompleted()
Materialize completed
*/
```
원본 시퀀스가 완료되면 구체화 된 시퀀스가 'OnCompleted'알림 값을 생성 한 다음 완료됩니다. Notification <T>는 3 가지 구현을 가진 추상 클래스입니다.
* OnNextNotification
* OnErrorNotification
* OnCompletedNotification

Notification <T>은 네 가지 공개 속성 (Kind, HasValue, Value 및 Exception)을 제공합니다. 분명히 OnNextNotification 만 HasValue에 대해 true를 반환하고 유용한 Value 구현을 갖습니다. 또한 OnErrorNotification이 예외에 대한 값을 갖는 유일한 구현임을 분명히해야합니다. Kind 속성은 어떤 메서드가 사용하기에 적합한 지 알 수 있도록 열거 형을 반환합니다.
``` csharp
public enum NotificationKind
{
  OnNext,
  OnError,
  OnCompleted,
}
```
이 다음 예제에서는 오류가 발생한 시퀀스를 생성합니다. 구체화 된 시퀀스의 최종 값은 OnErrorNotification입니다. 또한 구체화 된 시퀀스는 오류가 아니며 성공적으로 완료됩니다.
``` csharp
var source = new Subject<int>();
source.Materialize()
.Dump("Materialize");
source.OnNext(1);
source.OnNext(2);
source.OnNext(3);
source.OnError(new Exception("Fail?"));

/*
Output:
Materialize-->OnNext(1)
Materialize-->OnNext(2)
Materialize-->OnNext(3)
Materialize-->OnError(System.Exception)
Materialize completed
*/
```
시퀀스의 구체화는 시퀀스 분석 또는 로깅 수행에 매우 유용 할 수 있습니다. Dematerialize 확장 메서드를 적용하여 구체화 된 시퀀스의 포장을 풀 수 있습니다. Dematerialize는 IObservable <Notification <TSource>에서만 작동합니다.

## SelectMany
변환 연산자 중 Select가 가장 유용하다는 것을 알 수 있습니다. 그것의 변환 출력에 매우 넓은 유연성을 허용하고 다른 변환 연산자의 일부를 재현하는 데 사용될 수 있습니다. 그러나 SelectMany 연산자는 훨씬 강력합니다. LINQ 및 Rx에서 바인딩 메서드는 SelectMany입니다. 대부분의 다른 변환 연산자는 SelectMany를 사용하여 작성할 수 있습니다.

개인적으로 Rx를 발견했을 때 나는 SelectMany 확장 방법을 고민하기 위해 애썼다. 제 동료 중 한 명이 SelectMany를 이해하는 데 도움이되었습니다. 더 나은 정의는 하나 이상에서 0 이상을 선택하는 것입니다. SelectMany에 대한 서명을 보면 소스 시퀀스와 함수를 매개 변수로 사용한다는 것을 알 수 있습니다.
``` csharp
IObservable<TResult> SelectMany<TSource, TResult>(
  this IObservable<TSource> source, 
  Func<TSource, IObservable<TResult>> selector)
```
selector 매개 변수는 T의 단일 값을 취하여 시퀀스를 반환하는 함수입니다. selector가 반환하는 시퀀스는 소스와 동일한 유형 일 필요는 없습니다. 마지막으로 SelectMany 반환 형식은 selector 반환 형식과 동일합니다.

이 방법은 Rx를 효과적으로 사용하기를 원한다면 이해하는 것이 매우 중요하므로이 단계를 천천히 시도해보십시오. 또한 IEnumerable <T>의 SelectMany 연산자와의 미묘한 차이점에 주목해야합니다.이 연산자는 곧 살펴 보겠습니다.

우리의 첫 번째 예제는 하나의 값 '3'이있는 시퀀스를 취합니다. 우리가 제공하는 셀렉터 함수는 더 많은 수의 시퀀스를 생성합니다. 이 결과 시퀀스는 1부터 제공된 값 즉 3까지의 범위가 될 것입니다. 따라서 우리는 시퀀스 [3]을 가져 와서 selector 함수에서 시퀀스 [1,2,3]를 반환합니다.
``` csharp
Observable.Return(3) // 3이라는 값을 i로 받아서 Range 시퀀스로 바꿔버림.
  .SelectMany(i => Observable.Range(1, i))
  .Dump("SelectMany");
/*
Output:
SelectMany-->1
SelectMany-->2
SelectMany-->3
SelectMany completed
*/
```
소스를 다음과 같이 [1,2,3] 시퀀스로 수정하면 ...
``` csharp
Observable.Range(1,3) // 결과로 나온 값 마다 시퀀스로 만들어 버리네..
.SelectMany(i => Observable.Range(1, i))
.Dump("SelectMany");
/*
SelectMany-->1
SelectMany-->1
SelectMany-->2
SelectMany-->1
SelectMany-->2
SelectMany-->3
SelectMany completed
*/
```
각 시퀀스 ([1], [1,2] 및 [1,2,3])의 결과가 [1,1,2,1,2,3]가 됨다.

이 마지막 예제는 SelectMany가 단일 값을 가져 와서 여러 값으로 확장하는 방법을 더 잘 보여줍니다. 이것을 우리가 일련의 값들에 적용하면 결과는 최종 시퀀스를 생성하기 위해 결합 된 각각의 자식 시퀀스입니다. 두 예제 모두 소스와 동일한 유형의 시퀀스를 반환했습니다. 그러나 이것은 제한 사항이 아니므로이 다음 예제에서는 다른 유형을 반환합니다. 정수를 ASCII 문자로 변환하는 Select 예제를 다시 사용합니다. 이렇게하기 위해 selector 함수는 단일 값을 갖는 char 시퀀스를 반환합니다.
``` csharp
Func<int, char> letter = i => (char)(i + 64);
Observable.Return(1)
  .SelectMany(i => Observable.Return(letter(i)));
  .Dump("SelectMany");
/*
SelectMany-->A
SelectMany completed
*/
```
많은 값을 갖도록 소스 시퀀스를 확장하면 많은 값을 가진 결과를 얻을 수 있습니다.
``` csharp
Func<int, char> letter = i => (char)(i + 64);
Observable.Range(1,3)
  .SelectMany(i => Observable.Return(letter(i)))
  .Dump("SelectMany");
/*
Now the input of [1,2,3] produces [[A], [B], [C]] which is flattened to just [A,B,C].
SelectMany-->A
SelectMany-->B
SelectMany-->C
*/
```

마지막 예제는 숫자를 문자에 매핑합니다. 글자가 26 자 밖에 없으므로 26보다 큰 값을 무시하는 것이 좋습니다. 이렇게하는 것이 쉽습니다. 소스의 각 요소에 대해 시퀀스를 반환해야하지만 빈 시퀀스가되지 않도록하는 규칙은 없습니다. 이 경우 요소 값이 1-26 범위를 벗어나는 숫자이면 빈 시퀀스를 반환합니다.
``` csharp
Func<int, char> letter = i => (char)(i + 64);
Observable.Range(1, 30)
.SelectMany(
  i =>
  {
   if (0 < i && i < 27)
    {
     return Observable.Return(letter(i));
    }
    else
    {
     return Observable.Empty<char>();
    }
  })
  .Dump("SelectMany");
/*
Output:

A
B
C
...
X
Y
Z
Completed
*/
```
소스 시퀀스 [1..30]에 대해 값 1은 시퀀스 [A]를 생성하고, 값 2는 시퀀스 [B]를 생성하고, 값 26이 시퀀스 [Z]를 생성 할 때까지 계속된다. 소스에서 값 27을 생성 할 때 선택기 함수가 빈 시퀀스 []를 반환했습니다. 값 28, 29 및 30도 빈 시퀀스를 생성했습니다. 선택기에 대한 호출의 모든 시퀀스가 최종 결과를 생성하기 위해 확장 된 후에는 [A..Z] 시퀀스로 끝납니다.

이제 세 가지 고차 함수 중 세 번째를 다뤘으니 이미 배운 방법을 생각해 보겠습니다. 먼저 Where 확장 방법을 고려할 수 있습니다. 먼저 시퀀스 감소에 관한 장에서이 방법을 살펴 보았습니다. 이 방법은 시퀀스를 감소 시키지만 그 결과는 여전히 시퀀스이므로 함수형 fold에 적합하지 않습니다. 이것을 고려해 볼 때 Where는 실제로 bind에 적합하다는 것을 알 수 있습니다. 연습으로 SelectMany 연산자를 사용하여 Where의 확장 메서드 버전을 직접 작성하십시오. 마지막 도움말에서 도움을 받으십시오.
``` csharp
public static IObservable<T> Where<T>(this IObservable<T> source, Func<T, bool> predicate)
{
  return source.SelectMany(
  item =>
    {
    if (predicate(item))
    {
      return Observable.Return(item);
    }
    else
    {
     return Observable.Empty<T>();
    }
  });
}
```
이제 SelectMany를 사용하여 Where를 생성 할 수 있으므로 Skip and Take와 같은 다른 필터를 재현하려면이 필터를 확장 할 수있는 자연스러운 진행이되어야합니다.

또 다른 연습으로 SelectMany를 사용하여 Select 확장 메서드의 당신만의 버전을 작성해보십시오. 도움이 필요하면 SelectMany를 사용하여 int 값을 char 값으로 변환하는 예제를 참조하십시오 ...
``` csharp
public static IObservable<TResult> MySelect<TSource, TResult>(
  this IObservable<TSource> source, Func<TSource, TResult> selector)
  {
    return source.SelectMany(value => Observable.Return(selector(value)));
  }
```

### IEnumerable<T> vs. IObservable<T> SelectMany
