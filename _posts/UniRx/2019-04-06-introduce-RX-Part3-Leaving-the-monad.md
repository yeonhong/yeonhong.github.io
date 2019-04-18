---
layout: single
title: "PART 3 - Taming the sequence : Leaving the monad"
related: true
classes: wide
categories: 
  - 프로그래밍
tags:
  - intro to RX
  - Reactive Programming
link: "http://introtorx.com/Content/v1.0.10621.0/10_LeavingTheMonad.html#LeavingTheMonad"
---

# Leaving the monad

관찰 가능한 시퀀스는 특히 LINQ를 사용하여 복잡한 쿼리를 작성할 때 유용합니다. 관찰 가능한 시퀀스의 이점을 인식 할지라도 때로는 기존 패러다임을 위해 IObservable <T> 패러다임을 남겨 두어야합니다 (예 : 이벤트 또는 작업 <T> 사용). 관찰하기 쉽다면 관찰 가능한 패러다임을 남기거나 관찰 가능한 패러다임과보다 친숙한 패러다임 사이를 이동하여 Rx를 배우는 것이 더 쉬울 수도 있습니다.

## What is a monad

우리는이 책의 앞부분에 모나드라는 용어를 우연히 언급했지만, 대부분은 매우 외래적인 용어입니다. 나는 모나드가 지나치게 복잡 해지는 것을 피하려고 노력할 것이지만, 우리의 다음 범주의 방법으로 우리를 도울 충분한 설명을 해 주겠다. 모나드의 완전한 정의는 매우 추상적입니다. 많은 다른 사람들이 우주 비행사에서 이상한 나라의 앨리스 (Alice in Wonderland)에 이르기까지 모든 종류의 은유를 사용하여 모나드를 정의하려고 노력했습니다. 모나 딕 프로그래밍을위한 튜토리얼의 대부분은 혼선에 추가 할 수있는 코드 예제를 위해 Haskell을 사용합니다. 우리에게는 사실 모나드가 계산을 나타내는 프로그래밍 구조입니다. 이것을 다른 프로그래밍 구조와 비교하십시오 :

Data structure
> Purely state e.g. a List, a Tree or a Tuple
Contract
> 계약 정의 또는 추상 기능 인터페이스 예) 추상 클래스
Object-Orientated structure
> State와 behavior를 같이

일반적으로 모나드 구조는 확장 메소드를 사용하는 것처럼 연산자를 연결하여 파이프 라인을 생성 할 수 있습니다.
> Monads는 도메인 모델의 데이터 대신 프로그램 논리를 캡슐화하는 일종의 추상 데이터 형식 생성자입니다.

Wikipedia에서 모나드를 깔끔하게 정의하면 시퀀스를 모나드로보기 시작할 수 있습니다. 이 경우 추상 데이터 유형은 IObservable <T> 유형입니다. 관찰 가능한 시퀀스를 사용할 때 함수를 추상 데이터 형식 (IObservable <T>)에 작성하여 쿼리를 만듭니다. 이 쿼리는 우리의 캡슐화 된 프로그래밍 로직이됩니다.

제어 흐름을 정의하기위한 모나드 (monads)의 사용은 IO, 동시성 및 예외와 같은 일반적으로 번거로운 프로그래밍 영역을 다룰 때 특히 유용합니다. 이것은 Rx의 장점 중 일부일뿐입니다!

## Why leave the monad?

다른 패러다임에서 관찰 가능한 시퀀스를 소비하려는 다양한 이유가 있습니다. 외부 적으로 기능을 노출해야하는 라이브러리는이를 이벤트 또는 태스크 인스턴스로 제시해야 할 수도 있습니다. 데모 및 샘플 코드에서 비동기식 이동 부분의 수를 제한하기 위해 차단 방법을 사용하는 것이 좋습니다. 이것은 Rx에 대한 학습 곡선을 약간 덜 가파르게하는 데 도움이 될 수 있습니다!

프로덕션 코드에서 관찰 할 수있는 시퀀스에서 차단 방법으로 이동하는 것은 '모나드를 깨뜨리는'일은 거의 없습니다. 비동기식 패러다임과 동기식 패러다임을 전환하는 것은 교착 상태 및 확장 성 문제와 같은 동시성 문제의 공통 근본 원인이므로주의해서 수행해야합니다.

이 장에서는 Rx에서 IObservable <T> 모나드를 남길 수있는 방법을 살펴 보겠습니다.

## ForEach

ForEach 메서드는 요소를받은대로 처리하는 방법을 제공합니다. ForEach와 Subscribe의 주요 차이점은 ForEach가 시퀀스가 완료 될 때까지 현재 스레드를 차단한다는 것입니다.

``` csharp
var source = Observable.Interval(TimeSpan.FromSeconds(1))
  .Take(5);
source.ForEach(i => Console.WriteLine("received {0} @ {1}", i, DateTime.Now));
Console.WriteLine("completed @ {0}", DateTime.Now);

/*
Output:

received 0 @ 01/01/2012 12:00:01 a.m.
received 1 @ 01/01/2012 12:00:02 a.m.
received 2 @ 01/01/2012 12:00:03 a.m.
received 3 @ 01/01/2012 12:00:04 a.m.
received 4 @ 01/01/2012 12:00:05 a.m.
completed @ 01/01/2012 12:00:05 a.m.
*/
```

완료된 행은 예상대로 마지막 행임을 유의하십시오. 분명히하기 위해 구독 확장 메서드에서 비슷한 기능을 얻을 수 있지만 Subscribe 메서드는 차단되지 않습니다. ForEach에 대한 호출을 Subscribe에 대한 호출로 대체하면 completed가 먼저 발생합니다.

``` csharp
var source = Observable.Interval(TimeSpan.FromSeconds(1))
  .Take(5);
source.Subscribe(i => Console.WriteLine("received {0} @ {1}", i, DateTime.Now));
Console.WriteLine("completed @ {0}", DateTime.Now);

/*
Output:

completed @ 01/01/2012 12:00:00 a.m.
received 0 @ 01/01/2012 12:00:01 a.m.
received 1 @ 01/01/2012 12:00:02 a.m.
received 2 @ 01/01/2012 12:00:03 a.m.
received 3 @ 01/01/2012 12:00:04 a.m.
received 4 @ 01/01/2012 12:00:05 a.m.
*/
```

구독 확장 방법과 달리 ForEach에는 하나의 오버로드 만 있습니다. 단일의 인수로서 Action <T>를 취하는 것. 이와 달리 Rx의 이전 버전 (시험판)에서는 ForEach 메소드가 Subscribe와 동일한 대부분의 오버로드를가집니다. ForEach의 이러한 overload는 더 이상 사용되지 않으며, 나는 그렇게 생각합니다. 동기 호출에는 OnCompleted 처리기가 있을 필요가 없으므로 필요하지 않습니다. 위에서 한 것처럼 ForEach 호출 바로 다음에 호출 할 수 있습니다. 또한 try / catch 블록을 사용하여 다른 동기 코드에 사용하는 것처럼 OnError 핸들러를 표준 구조적 예외 처리로 바꿀 수 있습니다. 또한 List <T> 형식의 ForEach 인스턴스 메서드에 대칭을 제공합니다.

``` csharp
var source = Observable.Throw<int>(new Exception("Fail"));
try
{
  source.ForEach(Console.WriteLine);
}
catch (Exception ex)
{
  Console.WriteLine("error @ {0} with {1}", DateTime.Now, ex.Message);
}
finally
{
  Console.WriteLine("completed @ {0}", DateTime.Now);
}
/*
Output:

error @ 01/01/2012 12:00:00 a.m. with Fail
completed @ 01/01/2012 12:00:00 a.m.
*/
```

ForEach 메서드는 다른 블로킹 명령(First and Last 등)와 마찬가지로 주의해서 사용해야합니다. 나는 스파이크, 테스트 및 데모 코드에 대해서만 ForEach 메소드를 남겨 둘 것입니다. 우리는 동시성을 볼 때 블로킹 호출을 도입 할 때의 문제점을 논의 할 것이다.

## ToEnumerable

IObservable <T> 밖으로 전환하는 다른 방법은 ToEnumerable 확장 메서드를 호출하는 것입니다. 간단한 예를 들면 다음과 같습니다.

``` csharp
var period = TimeSpan.FromMilliseconds(200);
var source = Observable.Timer(TimeSpan.Zero, period)
  .Take(5);
var result = source.ToEnumerable();
foreach (var value in result)
{
  Console.WriteLine(value);
}
Console.WriteLine("done");
/*
Output:

0
1
2
3
4
done
*/
```

소스 관찰 가능 시퀀스는 시퀀스를 열거하기 시작할 때 (즉, lately하게) 구독됩니다. ForEach 확장 메서드와 달리 ToEnumerable 메서드를 사용하면 다음 요소로 이동하려고 할 때만 차단되고 사용할 수 없다는 것을 의미합니다. 또한 시퀀스가 소비하는 것보다 빠르게 값을 생성하면 캐시됩니다.

오류를 해결하기 위해 다른 열거 가능한 시퀀스와 마찬가지로 foreach 루프를 try / catch로 래핑 할 수 있습니다.

``` csharp
try
{
  foreach (var value in result)
  {
    Console.WriteLine(value);
  }
}
catch (Exception e)
{
  Console.WriteLine(e.Message);
}
```

푸시 모델에서 푸시 모델 (비 차단에서 차단)으로 이동할 때 표준 경고가 적용됩니다.

## To a single collection

푸시와 풀 사이에서 뭘 사용할지 고민을 피하려면, 다음 네 가지 방법 중 하나를 사용하여 단일 알림에서 전체 목록을 다시 가져올 수 있습니다. 그것들은 모두 동일한 의미를 지니고 있지만 다른 형식으로 데이터를 생성합니다. 이들은 해당 IEnumerable <T> 연산자와 비슷하지만 비동기 동작을 유지하기 위해 반환 값이 다릅니다.

### ToArray and ToList

ToArray와 ToList는 모두 관찰 가능한 시퀀스를 가져 와서 List <T>의 배열 또는 인스턴스로 각각 패키지화합니다. 관찰 가능한 시퀀스가 완료되면 배열 또는 목록이 결과 시퀀스의 단일 값으로 푸시됩니다.

``` csharp
var period = TimeSpan.FromMilliseconds(200); 
var source = Observable.Timer(TimeSpan.Zero, period).Take(5); 
var result = source.ToArray();
result.Subscribe(
  arr => {
    Console.WriteLine("Received array");
    foreach (var value in arr)
    {
      Console.WriteLine(value);
    }
  },
  () => Console.WriteLine("Completed")
);
Console.WriteLine("Subscribed");
/*
Output:

Subscribed // 이게먼저 불렸다는건,, obsaveable이 끝날때까진 메세지가 안온다는 이야기.
Received array
0
1
2
3
4
Completed
*/
```

이러한 메서드는 여전히 관찰 가능한 시퀀스를 반환하므로 OnError 처리기를 사용하여 오류를 처리 할 수 있습니다. 소스 순서는 단일 통지로 패키지됩니다. 당신은 전체 시퀀스 또는 오류를 얻습니다. 소스가 값을 생성 한 다음 오류가 발생하면 해당 값을받지 못합니다. 다음 네개의 연산자(ToArray, ToList, ToDictionary 및 ToLookup)는 이와 같은 오류를 처리합니다.

### ToDictionary and ToLookup

배열과리스트의 대안으로 Rx는 ToDictionary와 ToLookup 메소드를 사용하여 관찰 가능한 시퀀스를 사전이나 룩업에 패키지화 할 수 있습니다. ToArray 및 ToList 메서드와 동일한 의미를 갖는 두 메서드는 단일 값으로 시퀀스를 반환하고 동일한 오류 처리 기능을 갖기 때문에 두 메서드 모두 동일한 의미를 갖습니다.

ToDictionary 확장 메서드가 오버로드됩니다.

``` csharp
// Creates a dictionary from an observable sequence according to a specified key selector
// function, a comparer, and an element selector function.
public static IObservable<IDictionary<TKey, TElement>> ToDictionary<TSource, TKey, TElement>(
  this IObservable<TSource> source,
  Func<TSource, TKey> keySelector,
  Func<TSource, TElement> elementSelector,
  IEqualityComparer<TKey> comparer)
{...}
// Creates a dictionary from an observable sequence according to a specified key selector
// function, and an element selector function.
public static IObservable<IDictionary<TKey, TElement>> ToDictionary<TSource, TKey, TElement>(
  this IObservable<TSource> source,
  Func<TSource, TKey> keySelector,
  Func<TSource, TElement> elementSelector)
{...}
// Creates a dictionary from an observable sequence according to a specified key selector
// function, and a comparer.
public static IObservable<IDictionary<TKey, TSource>> ToDictionary<TSource, TKey>(
  this IObservable<TSource> source,
  Func<TSource, TKey> keySelector,
  IEqualityComparer<TKey> comparer)
{...}
// Creates a dictionary from an observable sequence according to a specified key selector
// function.
public static IObservable<IDictionary<TKey, TSource>> ToDictionary<TSource, TKey>(
  this IObservable<TSource> source,
  Func<TSource, TKey> keySelector)
{...}
```

ToLookup 확장 메서드 오버로드

``` csharp
// Creates a lookup from an observable sequence according to a specified key selector
// function, a comparer, and an element selector function.
public static IObservable<ILookup<TKey, TElement>> ToLookup<TSource, TKey, TElement>(
  this IObservable<TSource> source,
  Func<TSource, TKey> keySelector,
  Func<TSource, TElement> elementSelector,
  IEqualityComparer<TKey> comparer)
{...}
// Creates a lookup from an observable sequence according to a specified key selector
// function, and a comparer.
public static IObservable<ILookup<TKey, TSource>> ToLookup<TSource, TKey>(
  this IObservable<TSource> source,
  Func<TSource, TKey> keySelector,
  IEqualityComparer<TKey> comparer)
{...}
// Creates a lookup from an observable sequence according to a specified key selector
// function, and an element selector function.
public static IObservable<ILookup<TKey, TElement>> ToLookup<TSource, TKey, TElement>(
  this IObservable<TSource> source,
  Func<TSource, TKey> keySelector,
  Func<TSource, TElement> elementSelector)
{...}
// Creates a lookup from an observable sequence according to a specified key selector
// function.
public static IObservable<ILookup<TKey, TSource>> ToLookup<TSource, TKey>(
  this IObservable<TSource> source,
  Func<TSource,
  TKey> keySelector)
{...}
```

ToDictionary와 ToLookup에는 키를 가져 오기 위해 각 값을 적용 할 수있는 함수가 필요합니다. 또한 ToDictionary 메서드는 모든 키가 고유해야한다는 위임을 오버로드합니다. 중복 키가 발견되면 DuplicateKeyException을 사용하여 시퀀스를 종료합니다. 한편, ILookup <TKey, TElement>는 키로 그룹화 된 여러 값을 갖도록 설계되었습니다. 키당 값이 많은 경우 ToLookup이 더 나은 옵션 일 수 있습니다.

## ToTask

우리는 AsyncSubject <T>를 Task <T>와 비교했으며 Task에서 관찰 가능한 시퀀스로 전환하는 방법을 보여주었습니다. ToTask 확장 메서드를 사용하면 관찰 할 수있는 시퀀스를 Task <T>로 변환 할 수 있습니다. AsyncSubject <T>와 마찬가지로 이 메서드는 여러 값을 무시하고 마지막 값만 반환합니다.

``` csharp
// Returns a task that contains the last value of the observable sequence.
public static Task<TResult> ToTask<TResult>(
  this IObservable<TResult> observable)
{...}
// Returns a task that contains the last value of the observable sequence, with state to
//  use as the underlying task's AsyncState.
public static Task<TResult> ToTask<TResult>(
  this IObservable<TResult> observable,
  object state)
{...}
// Returns a task that contains the last value of the observable sequence. Requires a
//  cancellation token that can be used to cancel the task, causing unsubscription from
//  the observable sequence.
public static Task<TResult> ToTask<TResult>(
  this IObservable<TResult> observable,
  CancellationToken cancellationToken)
{...}
// Returns a task that contains the last value of the observable sequence, with state to
//  use as the underlying task's AsyncState. Requires a cancellation token that can be used
//  to cancel the task, causing unsubscription from the observable sequence.
public static Task<TResult> ToTask<TResult>(
  this IObservable<TResult> observable,
  CancellationToken cancellationToken,
  object state)
{...}
```

이것은 ToTask 연산자를 사용하는 방법의 간단한 예입니다. 참고로 ToTask 메서드는 System.Reactive.Threading.Tasks 네임 스페이스에 있습니다.

``` csharp
var source = Observable.Interval(TimeSpan.FromSeconds(1))
  .Take(5);
var result = source.ToTask(); //Will arrive in 5 seconds.
Console.WriteLine(result.Result);
/*
Output:

4
*/
```

``` csharp
var source = Observable.Throw<long>(new Exception("Fail!"));
var result = source.ToTask();
try
{
  Console.WriteLine(result.Result);
}
catch (AggregateException e)
{
  Console.WriteLine(e.InnerException.Message);
}
/*
Output:

Fail!
*/
```

당신의 task를 가졌다면, 당신은 TPL의 모든 기능, 예를 들면 continuations 기능을 사용할 수 있습니다.

## ToEvent<T>

FromEventPattern에서 관찰 가능한 시퀀스의 소스로 이벤트를 사용할 수있는 것처럼, 관측 가능한 시퀀스를 ToEvent 확장 메서드를 사용하는 표준 .NET 이벤트처럼 보이게 만들 수도 있습니다.

``` csharp
// Exposes an observable sequence as an object with a .NET event. 
public static IEventSource<unit> ToEvent(this IObservable<Unit> source)
{...} 
// Exposes an observable sequence as an object with a .NET event. 
public static IEventSource<TSource> ToEvent<TSource>(
  this IObservable<TSource> source) 
{...} 
// Exposes an observable sequence as an object with a .NET event. 
public static IEventPatternSource<TEventArgs> ToEventPattern<TEventArgs>(
  this IObservable<EventPattern<TEventArgs>> source) 
  where TEventArgs : EventArgs 
{...} 
```

ToEvent 메서드는 IEventSource <T>를 반환합니다.이 이벤트 멤버에는 OnNext라는 단일 이벤트 멤버가 있습니다.

``` csharp
public interface IEventSource<T> 
{ 
  event Action<T> OnNext; 
} 
```

우리가 관찰 가능한 시퀀스를 ToEvent 메소드로 변환 할 때 Action <T>을 제공함으로써 구독 할 수 있습니다. 여기서 우리는 람다로합니다.

``` csharp
var source = Observable.Interval(TimeSpan.FromSeconds(1))
  .Take(5); 
var result = source.ToEvent(); 
result.OnNext += val => Console.WriteLine(val);
/*
Output:

0
1
2
3
4
*/
```

### ToEventPattern

이는 표준 이벤트 패턴을 따르지 않습니다. 일반적으로 이벤트에 가입하면 보낸 사람 및 EventArgs 매개 변수를 처리해야합니다. 위의 예제에서 값을 얻습니다. 표준 패턴을 따르는 이벤트로 시퀀스를 노출하려면 ToEventPattern을 사용해야합니다.

ToEventPattern은 IObservable <EventPattern <TEventArgs >>를 취해이를 IEventPatternSource <TEventArgs>로 변환합니다. 이러한 유형의 공용 인터페이스는 매우 간단합니다.

``` csharp
public class EventPattern<TEventArgs> : IEquatable<EventPattern<TEventArgs>>
where TEventArgs : EventArgs 
{ 
  public EventPattern(object sender, TEventArgs e)
  { 
    this.Sender = sender; 
    this.EventArgs = e; 
  } 
  public object Sender { get; private set; } 
  public TEventArgs EventArgs { get; private set; } 
  //...equality overloads
}

public interface IEventPatternSource<TEventArgs> where TEventArgs : EventArgs
{ 
  event EventHandler<TEventArgs> OnNext; 
} 
```

이것들은 작업하기 매우 쉽습니다. 따라서 EventArgs 형식을 만든 다음 Select를 사용하여 간단한 변환을 적용하면 표준 시퀀스를 패턴에 맞출 수 있습니다.

``` csharp
public class MyEventArgs : EventArgs 
{ 
  private readonly long _value; 

  public MyEventArgs(long value) 
  { 
    _value = value; 
  }

  public long Value 
  { 
    get { return _value; } 
  } 
} 

/*
var source = Observable.Interval(TimeSpan.FromSeconds(1))
  .Select(i => new EventPattern<MyEventArgs>(this, new MyEventArgs(i)));
*/
```

이제 호환 가능한 시퀀스가 생겼으므로 ToEventPattern을 사용하고 표준 이벤트 핸들러를 사용할 수 있습니다.

``` csharp
var result = source.ToEventPattern(); 
result.OnNext += (sender, eventArgs) => Console.WriteLine(eventArgs.Value);
```

이제 .NET 이벤트로 돌아갈 방법을 알았으므로 휴식을 취하고 Rx가 왜 더 나은 모델인지 기억하십시오.

* C #에서는 이벤트에 흥미로운 인터페이스가 있습니다. 일부에서는 + = 및 -= 연산자가 부 자연스러운 콜백 등록 방법을 찾았습니다
* 이벤트를 작성하기 어렵습니다.
* 이벤트는 시간이 지남에 따라 쉽게 쿼리 할 수있는 기능을 제공하지 않습니다.
* 이벤트는 우발적 인 메모리 누수의 일반적인 원인입니다.
* 이벤트에는 신호 완료를 위한 표준 패턴이 없습니다.
* 이벤트는 동시성 또는 다중 스레드 응용 프로그램에 거의 도움이되지 않습니다. 예를 들어, 별도의 스레드에서 이벤트를 발생 시키려면 모든 배관 작업을 수행해야합니다

*****

이 장에서 살펴본 일련의 방법들은 시퀀스 생성 장에서 시작된 원을 완성합니다. 우리는 이제 관찰 가능한 시퀀스 모나드에 들어가고 나가는 방법을 가지고 있습니다. IObservable <T> 모나드를 선택하거나 선택 해제 할 때주의하십시오. 과도하게 그렇게하면 코드 기반이 엉망이 될 수 있으며 설계상의 결함이 있음을 나타낼 수 있습니다.

*****
