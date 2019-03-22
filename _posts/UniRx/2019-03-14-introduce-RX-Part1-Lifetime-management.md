---
layout: single
title: "PART 1 - Getting started : Lifetime management"
related: true
categories: 
  - 프로그래밍
tags:
  - intro to RX
  - Reactive Programming
link: "http://introtorx.com/Content/v1.0.10621.0/03_LifetimeManagement.html#LifetimeManagement"
---

## Lifetime management
Rx 코드의 본질은 소비자가 시퀀스가 언제 값을 제공하거나 종료하는지 모를 수 있다는 것입니다. 이 불확실성으로 인해 코드가 확실성을 제공하지 못하는 것은 아닙니다. 값 수락시기와 값 수락을 중지 할 때를 제어 할 수 있습니다. 여전히 도메인의 마스터가되어야합니다. Rx 자원 관리의 기본 사항을 이해하면 응용 프로그램이 효율적이고 버그가없고 가능한 한 예측 가능할 수 있습니다.

Rx는 쿼리에 대한 가입 기간을 정밀하게 제어합니다. 익숙한 인터페이스를 사용하는 동안 쿼리와 관련된 리소스를 결정적으로 해제 할 수 있습니다. 이를 통해 리소스를 가장 효율적으로 관리하는 방법을 결정할 수 있으므로 범위를 최대한 좁게 유지하는 것이 이상적입니다.

이전 장에서 핵심 유형을 소개하고 몇 가지 예를 들어 보았습니다. 초기 샘플을 단순하게 유지하기 위해 우리는 IObservable <T> 인터페이스의 매우 중요한 부분을 무시했습니다. Subscribe 메서드는 IObserver <T> 매개 변수를 사용하지만 대신 Action <T>을 사용하는 확장 메서드를 사용할 때이를 제공 할 필요가 없습니다. 우리가 간과 한 중요한 부분은 두 Subscribe 메서드 모두 반환 값을가집니다. **반환 유형은 IDisposable입니다. 이 장에서는이 반환 값을 구독 수명 동안 관리 수명에 어떻게 사용할 수 있는지** 자세히 살펴 보겠습니다.

## Subscribing (구독하기)
계속하기 전에 Subscribe 확장 메소드의 오버로드를 간단히 살펴볼 가치가 있습니다. 이전 장에서 사용한 오버로드는 OnNext를 호출 할 때 수행 할 액션 <T> 만 전달할 수있는 간단한 오버로드 구독이었습니다. 이러한 추가 오버로드로 인해 IObserver <T> 인스턴스를 생성 한 다음 전달하지 않아도됩니다.
``` csharp
//Just subscribes to the Observable for its side effects. 
// All OnNext and OnCompleted notifications are ignored.
// OnError notifications are re-thrown as Exceptions.
IDisposable Subscribe<TSource>(this IObservable<TSource> source);
//The onNext Action provided is invoked for each value.
//OnError notifications are re-thrown as Exceptions.
IDisposable Subscribe<TSource>(this IObservable<TSource> source, 
Action<TSource> onNext);
//The onNext Action is invoked for each value.
//The onError Action is invoked for errors
IDisposable Subscribe<TSource>(this IObservable<TSource> source, 
Action<TSource> onNext, 
Action<Exception> onError);
//The onNext Action is invoked for each value.
//The onCompleted Action is invoked when the source completes.
//OnError notifications are re-thrown as Exceptions.
IDisposable Subscribe<TSource>(this IObservable<TSource> source, 
Action<TSource> onNext, 
Action onCompleted);
//The complete implementation
IDisposable Subscribe<TSource>(this IObservable<TSource> source, 
Action<TSource> onNext, 
Action<Exception> onError, 
Action onCompleted);
```

이러한 각 오버로드를 사용하면 IObservable 인스턴스에서 생성 할 수있는 각 알림에 대해 실행하려는 여러 대리자 조합을 전달할 수 있습니다. 주의해야 할 핵심 사항은 OnError 알림에 대한 대리자를 지정하지 않는 오버로드를 사용하면 모든 OnError 알림이 예외로 다시 throw된다는 것입니다. 언제든지 오류를 제기 할 수 있다고 생각하면 디버깅을 어렵게 만들 수 있습니다. **일반적으로 OnError 알림을 처리 할 대리자를 지정하는 오버로드를 사용하는 것이 가장 좋습니다.**

이 예제에서는 표준 .NET 구조적 예외 처리를 사용하여 오류를 catch하려고 시도합니다.
``` csharp
var values = new Subject<int>();
try
{
  values.Subscribe(value => Console.WriteLine("1st subscription received {0}", value));
}
catch (Exception ex)
{
  Console.WriteLine("Won't catch anything here!");
}
values.OnNext(0);
//Exception will be thrown here causing the app to fail.
values.OnError(new Exception("Dummy exception"));
```

예외를 처리하는 올바른 방법은 이 예제 에서처럼 OnError 알림에 대한 대리자를 제공하는 것입니다.
``` csharp
var values = new Subject<int>();
values.Subscribe(
  value => Console.WriteLine("1st subscription received {0}", value),
  ex => Console.WriteLine("Caught an exception : {0}", ex));
values.OnNext(0);
values.OnError(new Exception("Dummy exception"));
```
이 장의 뒷부분에서 시퀀스의 오류를 다루는 다른 흥미로운 방법을 살펴볼 것입니다.

## Unsubscribing
구독을 취소하는 방법을 아직 검토하지 않았습니다. Rx public API에서 Unsubscribe 메소드를 찾으려면 찾을 수 없습니다. 수신 거부 (Unsubscribe) 방법을 제공하는 대신 수신이 이루어질 때마다 Rx는 IDisposable을 반환합니다. 이 일회용 프로그램은 구독 자체 또는 구독을 나타내는 토큰으로 생각할 수 있습니다. 처분하면 구독을 처분하고 효과적으로 구독을 취소합니다. 구독 호출 결과에 대해 Dispose를 호출해도 다른 구독자에게는 부작용이 발생하지 않습니다. 그것은 관찰 대상의 내부 구독 목록에서 구독을 제거하기 만하면됩니다. 이렇게하면 단일 IObservable <T>에서 여러 번 구독을 호출 할 수 있으므로 서로 영향을주지 않고 구독을 시작하거나 종료 할 수 있습니다. 이 예에서는 초기에 구독이 두 개인 경우 다른 구독자가 기본 시퀀스의 게시물을 계속받을 수 있도록 구독을 일찍 처리합니다.

``` csharp
var values = new Subject<int>();
var firstSubscription = values.Subscribe(value => 
Console.WriteLine("1st subscription received {0}", value));
var secondSubscription = values.Subscribe(value => 
Console.WriteLine("2nd subscription received {0}", value));
values.OnNext(0);
values.OnNext(1);
values.OnNext(2);
values.OnNext(3);
firstSubscription.Dispose();
Console.WriteLine("Disposed of 1st subscription");
values.OnNext(4);
values.OnNext(5);
/* output
1st subscription received 0
2nd subscription received 0
1st subscription received 1
2nd subscription received 1
1st subscription received 2
2nd subscription received 2
1st subscription received 3
2nd subscription received 3
Disposed of 1st subscription
2nd subscription received 4
2nd subscription received 5
*/
```

팀 구축 Rx는 ISubscription 또는 IUnsubscribe와 같은 새 인터페이스를 만들어 구독 취소를 용이하게 할 수있었습니다. 기존 IObservable <T> 인터페이스에 구독 취소 메서드를 추가 할 수있었습니다. 대신 IDisposable 유형을 사용하면 다음과 같은 이점을 무료로 얻을 수 있습니다.
* type이 이미 있습니다.
* 사람들은 type을 이해합니다.
* IDisposable에는 표준 용도와 패턴이 있습니다.
* using 키워드를 통한 언어 지원
* FxCop과 같은 정적 분석 도구는 사용법을 도와 줄 수 있습니다.
* IObservable <T> 인터페이스는 매우 단순합니다.

IDisposable 지침에 따라 원하는만큼 Dispose를 호출 할 수 있습니다. 첫 번째 call은 가입을 취소하고 더 이상의 call는 구독이 이미 처리되었으므로 아무런 조치도 취하지 않습니다.

## OnError and OnCompleted
OnError와 OnCompleted는 모두 시퀀스 완료를 나타냅니다. 시퀀스가 OnError 또는 OnCompleted를 게시하면이 시퀀스가 마지막 게시가되며 OnNext를 더 이상 호출 할 수 없습니다. 이 예에서는 OnCompleted 다음에 OnNext 호출을 게시하려고 시도하며 OnNext는 무시됩니다.
``` csharp
var subject = new Subject<int>();
subject.Subscribe(
Console.WriteLine, 
() => Console.WriteLine("Completed"));
subject.OnCompleted();
subject.OnNext(2);
```
물론 OnCompleted 나 OnError 후에 퍼블리싱 할 수있는 독자적인 IObservable <T>을 구현할 수도 있지만 현재의 Subject 유형의 우선 순위를 따르지 않으며 비표준 구현이됩니다. 일관성없는 동작으로 인해 코드를 사용하는 응용 프로그램에서 예기치 않은 동작이 발생한다고 말하는 것이 안전하다고 생각합니다.

고려해야 할 흥미로운 점은 시퀀스가 완료되거나 오류가 발생하더라도 여전히 구독을 처리해야한다는 것입니다.

## IDisposable
IDisposable 인터페이스는 주변에있는 편리한 유형이며 Rx에도 필수 요소입니다. IDisposable을 명시 적 수명 관리로 구현하는 유형을 생각하고 싶습니다. 나는 Dispose () 메서드를 호출하여 "나는 그것으로 끝났습니다."라고 말할 수 있어야합니다.

이런 종류의 사고를 적용하고 C# using 문을 활용하면 범위를 만드는 편리한 방법을 만들 수 있습니다. 다시 말하면, using 문은 사실상 try / finally 블록이며 범위를 벗어날 때 인스턴스에서 항상 Dispose를 호출합니다.

IDisposable 인터페이스를 사용하여 스코프를 효율적으로 생성 할 수 있다고 생각하면, 이를 활용할 수있는 재미있는 작은 클래스를 만들 수 있습니다. 예를 들어 다음은 타이밍 이벤트를 기록하는 간단한 클래스입니다.
``` csharp
public class TimeIt : IDisposable
{
  private readonly string _name;
  private readonly Stopwatch _watch;
  public TimeIt(string name)
  {
    _name = name;
    _watch = Stopwatch.StartNew();
  }
  public void Dispose()
  {
    _watch.Stop();
    Console.WriteLine("{0} took {1}", _name, _watch.Elapsed);
  }
}
```

이 작은 클래스를 사용하면 범위를 만들고 코드 기반의 특정 섹션을 실행하는 데 걸리는 시간을 측정 할 수 있습니다. 다음과 같이 사용할 수 있습니다.
``` csharp
using (new TimeIt("Outer scope"))
{
  using (new TimeIt("Inner scope A"))
  {
    DoSomeWork("A");
  }
  using (new TimeIt("Inner scope B"))
  {
    DoSomeWork("B");
  }
  Cleanup();
}
/*
Inner scope A took 00:00:01.0000000
Inner scope B took 00:00:01.5000000
Outer scope took 00:00:02.8000000
*/
```

이 개념을 사용하여 콘솔 응용 프로그램에서 텍스트 색상을 설정할 수도 있습니다.
``` csharp
//Creates a scope for a console foreground color. When disposed, will return to 
//  the previous Console.ForegroundColor
public class ConsoleColor : IDisposable
{
  private readonly System.ConsoleColor _previousColor;
  public ConsoleColor(System.ConsoleColor color)
  {
    _previousColor = Console.ForegroundColor;
    Console.ForegroundColor = color;
  }
  public void Dispose()
  {
    Console.ForegroundColor = _previousColor;
  }
}
```
작은 스파이크 콘솔 응용 프로그램에서 색상을 쉽게 전환 할 수있는 편리한 기능입니다.
``` csharp
Console.WriteLine("Normal color");
using (new ConsoleColor(System.ConsoleColor.Red))
{
  Console.WriteLine("Now I am Red");
  using (new ConsoleColor(System.ConsoleColor.Green))
  {
    Console.WriteLine("Now I am Green");
  }
  Console.WriteLine("and back to Red");
}
```

따라서 IDisposable 인터페이스를 사용하여 결정적으로 관리되지 않는 리소스를 일반 적으로 사용하는 것 이상을 위해 사용할 수 있음을 알 수 있습니다. 그것은 수명이나 범위를 관리하는 데 유용한 도구입니다. 스톱워치 타이머에서부터 콘솔 텍스트의 현재 색까지, 일련의 알림 구독에 이르기까지 다양합니다.

Rx 라이브러리 자체는 IDisposable 인터페이스의 자유로운 사용을 채택하고 자체 구현의 몇 가지를 도입합니다.
* Disposable
* BooleanDisposable
* CancellationDisposable
* CompositeDisposable
* ContextDisposable
* MultipleAssignmentDisposable
* RefCountDisposable
* ScheduledDisposable
* SerialDisposable
* SingleAssignmentDisposable

각 구현에 대한 자세한 설명은 부록의 Disposables 참조를 참조하십시오. 지금은 매우 간단하고 유용한 Disposable 정적 클래스를 살펴 보겠습니다.
``` csharp
namespace System.Reactive.Disposables
{
  public static class Disposable
  {
    // Gets the disposable that does nothing when disposed.
    public static IDisposable Empty { get {...} }
    // Creates the disposable that invokes the specified action when disposed.
    public static IDisposable Create(Action dispose) {...}
  }
}
```
당신이 볼 수 있듯이, Empty와 Create의 두 멤버를 노출합니다. Empty 메서드를 사용하면 Dispose()가 호출 될 때 아무 작업도 수행하지 않는 IDisposable의 스텁 인스턴스를 가져올 수 있습니다. 이는 IDisposable을 반환하는 인터페이스 요구 사항을 충족해야 하지만 _관련성 높은 특정 구현이 없는 경우에 유용합니다._

다른 오버로드는 Create factory 메소드입니다.이 메소드를 사용하면 인스턴스가 폐기 될 때 호출 할 Action을 전달할 수 있습니다. Create 메서드는 표준 Dispose 의미를 보장하므로 Dispose ()를 여러 번 호출하면 한 번 제공 한 대리자 만 호출됩니다.
``` csharp
var disposable = Disposable.Create(() => Console.WriteLine("Being disposed."));
Console.WriteLine("Calling dispose...");
disposable.Dispose();
Console.WriteLine("Calling again...");
disposable.Dispose();

/* output
Calling dispose...
Being disposed.
Calling again...
*/
```
**"Being disposed"에 유의하십시오. 한 번만 인쇄됩니다.** 다음 장에서는 Observable.Using 메소드에서 자원의 수명을 subscription의 수명에 바인딩하는 또 다른 유용한 방법에 대해 설명합니다.

## Resource management vs. memory management
많은 .NET 개발자가 .NET 런타임의 가비지 수집기에 대해 모호하게 이해하고 있으며 특히 Finalizers 및 IDisposable과 상호 작용하는 방식을 모호하게 보여줄 수 있습니다. Framework Design Guidelines의 저자가 지적했듯이, 이것은 '자원 관리'와 '메모리 관리'사이의 혼란으로 인한 것일 수 있습니다.
> Dispose 패턴에 대해 처음 듣는 많은 사람들은 GC가 그 일을하지 않는다고 불평합니다. 그들은 자원을 수집해야한다고 생각합니다. 이는 관리되지 않는 세상에서했던 것처럼 자원을 관리해야하는 것과 같습니다. 사실 GC는 리소스를 관리하지 못했습니다. 그것은 메모리를 관리하도록 설계되었으며, 단지 그것을 수행하는 데 우수합니다. - Joe Duffy의 블로그에서 Krzysztof Cwalina

이는 .NET을 사용하기가 쉽도록 만든 Microsoft의 증거이자 오해의 핵심 요소 인 문제이기도합니다. 이를 고려할 때, **구독은 자동으로 처리되지 않는다는 점에 유의하는 것이 현명하다고 생각했습니다. 반환 된 IDisposable의 인스턴스에는 finalizer가없고 범위를 벗어날 때 수집되지 않는다고 가정 할 수 있습니다. Subscribe 메서드를 호출하고 반환 값을 무시하면 구독 취소 핸들 만 잃었습니다. 구독이 계속 존재하며이 리소스에 대한 액세스 권한을 실제로 잃어 버렸기 때문에 메모리가 누출되고 원치 않는 프로세스가 실행될 수 있습니다.**

이러한 주의 사항에 대한 예외는 Subscribe 확장 메소드를 사용할 때입니다. 이러한 메서드는 시퀀스가 완료되거나 오류가 발생하면 구독을 자동으로 분리하는 동작을 내부적으로 구성합니다. 자동 분리 동작이 있더라도; OnCompleted 또는 OnError에 의해 종료되지 않는 시퀀스를 고려해야합니다. 이러한 무한 시퀀스에 대한 가입을 명시 적으로 종료하려면 IDisposable 인스턴스가 필요합니다.
> 이 책의 많은 예제가 IDisposable 반환 값을 할당하지 않는다는 것을 알게 될 것이다. 이는 샘플의 간결성과 명확성을 위해서입니다. 사용 지침 및 모범 사례 정보는 부록에서 찾을 수 있습니다.

공통으로 IDisposable 인터페이스를 활용하여, 수신은 구독의 수명 기간 동안 결정 제어를 할 수있는 기능을 제공합니다. 구독은 독립적이므로 하나의 일회용 항목은 다른 항목에 영향을주지 않습니다. 일부 구독 확장 방법은 자동으로 분리 관찰자를 사용하지만 IDisposable을 구현하는 다른 리소스와 마찬가지로 **구독을 명시 적으로 관리하는 것이 좋습니다.** 이후 장에서 살펴볼 것처럼 **구독은 실제로 이벤트 핸들, 캐시 및 스레드와 같은 다른 리소스의 비용을 초래할 수 있습니다. 또한 항상 처리하기 어려운 방식으로 throw되는 예외를 방지하기 위해 항상 OnError 핸들러를 제공하는 것이 가장 좋습니다.**

구독 수명 관리에 대한 지식이 있으면 구독 및 기본 리소스에 긴밀한 연결 고리를 유지할 수 있습니다. Rx 코드에 표준 처리 패턴을 적절하게 적용하면 응용 프로그램을 예측 가능하고 유지 보수가 용이하며 확장이 용이하고 버그가 없기를 바랍니다.

> 즉, 구독하고 명시적으로 해지하라는 뜻.
