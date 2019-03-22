---
layout: single
title: "PART 1 - Getting started : Key Types"
related: true
permalink: /docs/intro to RX/
categories: 
  - 프로그래밍
tags:
  - intro to RX
  - Reactive Programming
link: "http://introtorx.com/Content/v1.0.10621.0/02_KeyTypes.html#KeyTypes"
---

Rx로 작업 할 때 이해해야 할 두 가지 주요 유형과 Rx를보다 효과적으로 학습하는 데 도움이되는 보조 유형의 하위 집합이 있습니다. **IObserver <T>** 및 **IObservable <T>**은 Rx의 기본 구성 요소를 형성하지만 **ISubject <TSource, TResult>를 구현하면 Rx를 처음 사용하는 개발자의 학습 곡선이 줄어 듭니다.**

많은 사람들이 LINQ 및 LINQ to Objects, LINQ to SQL 및 LINQ to XML과 같이 많이 사용되는 형식에 익숙합니다. 이러한 공통 구현의 각각은 당신에게 데이터를 질의 할 수있게 해준다. Rx는 동작중인 데이터를 쿼리하는 기능을 제공합니다. **근본적으로 Rx는 Observer 패턴의 토대를 기반으로합니다.** .NET은 이미 멀티 캐스트 대리자 또는 이벤트 (일반적으로 멀티 캐스트 대리자)와 같은 Observer 패턴을 구현하는 몇 가지 다른 방법을 노출합니다. 그러나 다음과 같이 덜 바람직한 기능을 나타내므로 멀티 캐스트 대리자는 이상적이지 않습니다.

* C #에서는 이벤트에 흥미로운 인터페이스가 있습니다. 일부에서는 += 및 -= 연산자 같이 부자연스러운 콜백 등록 방법을 찾았습니다
* 이벤트는 조합하기 어렵습니다.
* 이벤트는 시간이 지남에 따라 쉽게 쿼리 할 수있는 기능을 제공하지 않습니다.
* 이벤트는 우발적인 메모리 누수의 일반적인 원인입니다.
* 이벤트에는 신호 완료를위한 표준 패턴이 없습니다.
* 이벤트는 동시성 또는 다중 스레드에 대한 거의 도움이되지 않습니다. (예 : 별도의 스레드에서 이벤트를 발생 시키려면 모든 연관 작업이 필요합니다.)

Rx는 이러한 문제를 해결하기 위해 노력합니다. 여기서는 building block과 Rx를 구성하는 기본 유형을 소개합니다.

## IObservable<T>
[IObservable <T>](https://docs.microsoft.com/en-us/dotnet/api/system.iobservable-1?redirectedfrom=MSDN&view=netframework-4.7.2) 는 Rx로 작업하기위한 두 개의 새로운 핵심 인터페이스 중 하나입니다. __Subscribe 메소드__ 만 있는 간단한 인터페이스입니다. Microsoft는이 인터페이스가 .NET 버전 4.0에서 BCL에 포함되어 있음을 잘 알고 있습니다. IObservable <T>을 T 객체의 스트리밍 시퀀스로 구현하는 모든 것을 생각할 수 있어야합니다. 그래서 방법이 IObservable <Price>를 반환하면 나는 그것을 가격의 흐름으로 생각할 수 있습니다.

```csharp
//Defines a provider for push-based notification.
public interface IObservable<out T>
{
  //Notifies the provider that an observer is to receive notifications.
  IDisposable Subscribe(IObserver<T> observer);
}
```

## IObserver<T>
[IObserver <T>](https://docs.microsoft.com/en-us/dotnet/api/system.iobserver-1?redirectedfrom=MSDN&view=netframework-4.7.2)는 Rx로 작업하기위한 두 개의 핵심 인터페이스 중 하나입니다. 또한 .NET 4.0부터는 BCL에 포함 시켰습니다. Rx 팀이 .NET 3.5 및 Silverlight 사용자를위한 별도의 어셈블리에이 두 인터페이스를 포함 시켰기 때문에 .NET 4.0에 아직 설치되어 있지 않아도 걱정하지 마십시오. IObservable <T>는 "IEnumerable <T>의 기능적 이중성"을 의미합니다. 마지막 성명서가 무엇을 의미하는지 알고 싶다면 [채널 9](https://channel9.msdn.com/tags/Rx/)에서 여러 시간의 비디오를 감상하여 유형의 수학적 순도에 대해 토론하십시오. 다른 모든 사람들은 IEnumerable <T>가 효과적으로 세 가지 (다음 값, 예외 또는 시퀀스의 끝)를 생성 할 수 있음을 의미하므로 IObserver <T>의 세 가지 메서드를 통해 IObservable <T>도 __OnNext(T)__, __OnError(Exception)__ 및 __OnCompleted ()__입니다.

```csharp
//Provides a mechanism for receiving push-based notifications.
public interface IObserver<in T>
{
  //Provides the observer with new data.
  void OnNext(T value);
  //Notifies the observer that the provider has experienced an error condition.
  void OnError(Exception error);
  //Notifies the observer that the provider has finished sending push-based notifications.
  void OnCompleted();
}
```
Rx는 암시 적 계약을 맺어야합니다. IObserver <T>의 구현에는 OnNext (T)에 대한 호출이 한 번 이상 있거나 OnError (Exception) 또는 OnCompleted () 호출이 선택적으로 이어질 수 있습니다. 이 프로토콜은 시퀀스가 종료되면 항상 OnError (Exception) 또는 OnCompleted ()에 의해 종료됩니다. 그러나이 프로토콜은 OnNext (T), OnError (Exception) 또는 OnCompleted ()를 호출 할 것을 요구하지 않습니다. 이것은 빈 및 무한 시퀀스의 개념을 가능하게합니다. 나중에 더 자세히 살펴 보겠습니다.

흥미롭게도, Rx로 작업하는 경우 IObservable <T> 인터페이스에 자주 노출되지만 일반적으로 IObserver <T>에 관심을 가질 필요는 없습니다. 이는 Rx가 Subscribe와 같은 메소드를 통해 익명 구현을 제공하기 때문입니다.

### Implementing IObserver<T> and IObservable<T>
각 인터페이스를 구현하는 것은 꽤 쉽습니다. 콘솔에 값을 출력하는 옵저버를 만들고 싶다면이 방법만큼이나 쉬울 것입니다.
```csharp
public class MyConsoleObserver<T> : IObserver<T>
{
  public void OnNext(T value)
  {
   Console.WriteLine("Received value {0}", value);
  }
  public void OnError(Exception error)
  {
   Console.WriteLine("Sequence faulted with {0}", error);
  }
  public void OnCompleted()
  {
   Console.WriteLine("Sequence terminated");
  }
}
```

관찰 가능한 시퀀스를 구현하는 것은 약간 어렵습니다. 일련의 숫자를 반환하는 지나치게 단순화 된 구현은 다음과 같이 보일 수 있습니다.
```csharp
public class MySequenceOfNumbers : IObservable<int>
{
  public IDisposable Subscribe(IObserver<int> observer)
  {
    observer.OnNext(1);
    observer.OnNext(2);
    observer.OnNext(3);
    observer.OnCompleted();
    return Disposable.Empty;
  }
}
```

이 두 가지 구현을 함께 묶어 다음 출력을 얻을 수 있습니다.
```csharp
var numbers = new MySequenceOfNumbers();
var observer = new MyConsoleObserver<int>();
numbers.Subscribe(observer);
```

output
```
Received value 1
Received value 2
Received value 3
Sequence terminated
```

여기에있는 문제는 이것이 전혀 react가 없다는 것입니다. 이 구현은 블로킹이므로 List <T> 또는 배열과 같은 IEnumerable <T> 구현을 사용할 수도 있습니다.

인터페이스를 구현하는 이 문제는 우리에게 너무 많은 관심을 가져서는 안됩니다. Rx를 사용할 때 실제로 이러한 인터페이스를 구현할 필요가 없으므로 Rx는 필요한 모든 구현을 제공합니다. 간단한 것들을 살펴 봅시다.

## Subject<T>
IObserver <T>와 IObservable <T>을 '독자', '작가'또는 '소비자'와 '게시자'인터페이스로 생각합니다. IObservable <T>의 구현을 직접 작성하려면 IObservable 특성을 공개적으로 노출하는 동안 구독자에게 항목을 게시하고 오류를 발생 시키며 시퀀스가 완료 될 때이를 알릴 수 있어야합니다. 왜 그것은 IObserver <T>에 정의 된 메서드와 같습니다. 하나의 유형이 두 인터페이스를 구현하는 것이 이상하게 보일 수 있지만, 이는 쉽습니다. 이것은 Subject가 당신을 위해 할 수있는 것입니다. Subject <T>는 가장 기본적인 주제입니다. 효과적으로 IObservable <T>를 반환하는 메서드 뒤에 Subject <T>를 노출 할 수 있지만 내부적으로 OnNext, OnError 및 OnCompleted 메서드를 사용하여 시퀀스를 제어 할 수 있습니다.

이 아주 기본적인 예제에서는 Subject를 만들고 해당 Subject를 구독 한 다음 subject.OnNext (T)를 호출하여 시퀀스에 값을 게시합니다.
```csharp
static void Main(string[] args)
{
  var subject = new Subject<string>();
  WriteSequenceToConsole(subject);
  subject.OnNext("a");
  subject.OnNext("b");
  subject.OnNext("c");
  Console.ReadKey();
}

//Takes an IObservable<string> as its parameter. 
//Subject<string> implements this interface.
static void WriteSequenceToConsole(IObservable<string> sequence)
{
  //The next two lines are equivalent.
  //sequence.Subscribe(value=>Console.WriteLine(value));
  sequence.Subscribe(Console.WriteLine);
}
```

WriteSequenceToConsole 메서드는 구독 메서드에 대한 액세스 만 원하는대로 IObservable<string>을 사용합니다. 잠깐만, 구독 방법에 인수로 IObserver<string>이 필요하지 않습니까? 확실히 Console.WriteLine은 해당 인터페이스와 일치하지 않습니다. 그렇지만 Rx 팀은 Extension Method를 IObservable<T>에 제공하고 Action<T> 만받습니다. 작업은 항목이 게시 될 때마다 실행됩니다. OnNext, OnCompleted 및 OnError에 대해 호출 할 대리자 조합을 전달할 수있는 Subscribe 확장 메서드에 대한 다른 오버로드가 있습니다. 이것은 IObserver <T>를 구현할 필요가 없다는 것을 의미합니다.

보시다시피 Subject <T>는 Rx 프로그래밍을 시작하는 데 매우 유용 할 수 있습니다. 그러나 Subject <T>는 기본 구현입니다. Subject <T>에는 세 가지 형제가 있습니다. 이는 프로그램 실행 방식을 크게 바꿀 수있는 약간 다른 구현을 제공합니다.

## ReplaySubject<T>
ReplaySubject<T>는 **캐시 값의 기능을 제공하고** 지연 구독에 대해 재생합니다. 첫 번째 발행물이 구독 전에 발생하도록 이동한 예제 입니다.
``` csharp
static void Main(string[] args)
{
  var subject = new Subject<string>();
  subject.OnNext("a");
  WriteSequenceToConsole(subject);
  subject.OnNext("b");
  subject.OnNext("c");
  Console.ReadKey();
}
```

이 결과는 'b'와 'c'가 콘솔에 기록되지만 'a'는 무시된다는 것입니다. subject를 ReplaySubject <T>로 변경하기 위해 사소한 변경을 수행하면 모든 발행물이 다시 표시됩니다.
``` csharp
var subject = new ReplaySubject<string>();
subject.OnNext("a");
WriteSequenceToConsole(subject);
subject.OnNext("b");
subject.OnNext("c");
```

이는 경쟁 조건을 제거하기 위해 매우 편리 할 수 있습니다. **ReplaySubject <T>의 기본 생성자는 게시 된 모든 값을 캐시하는 인스턴스를 만듭니다.** 많은 시나리오에서 이것은 응용 프로그램에 불필요한 메모리 부담을 줄 수 있습니다. ReplaySubject <T>를 사용하면이 메모리 문제를 완화 할 수있는 간단한 캐시 만료 설정을 지정할 수 있습니다. 한 가지 옵션은 캐시에서 버퍼의 크기를 지정할 수 있다는 것입니다. 이 예제에서는 버퍼 크기가 2 인 ReplaySubject <T>를 만들고 구독 전에 게시 된 마지막 두 값만 가져옵니다.
``` csharp
public void ReplaySubjectBufferExample()
{
  var bufferSize = 2;
  var subject = new ReplaySubject<string>(bufferSize);
  subject.OnNext("a");
  subject.OnNext("b");
  subject.OnNext("c");
  subject.Subscribe(Console.WriteLine);
  subject.OnNext("d");
}
```

여기서 출력은 값 'a'가 캐시에서 삭제되었지만 값 'b'와 'c'가 여전히 유효 함을 나타냅니다. 값 'd'는 우리가 가입 한 후에 발행되었으므로 콘솔에도 기록됩니다.
```
Output:
b
c
d
```

ReplaySubject <T>에 의한 값의 끝없는 캐싱을 막기위한 또 다른 옵션은 캐시 window를 제공하는 것입니다. 이 예에서는 버퍼 크기로 ReplaySubject <T>를 만드는 대신 캐시 된 값이 유효한 time window를 지정합니다.
``` csharp
public void ReplaySubjectWindowExample()
{
  var window = TimeSpan.FromMilliseconds(150);
  var subject = new ReplaySubject<string>(window);
  subject.OnNext("w");
  Thread.Sleep(TimeSpan.FromMilliseconds(100));
  subject.OnNext("x");
  Thread.Sleep(TimeSpan.FromMilliseconds(100));
  subject.OnNext("y");
  subject.Subscribe(Console.WriteLine);
  subject.OnNext("z");
}
```

위의 예제에서 창은 150 밀리 초로 지정되었습니다. 값은 100 밀리 초 간격으로 게시됩니다. 일단 우리가 주제에 가입하면, 첫 번째 값은 200ms 이전이므로 만료되어 캐시에서 제거됩니다.
```
Output:
x
y
z
```

## BehaviorSubject<T>
BehaviorSubject <T>는 ReplaySubject <T>와 유사하지만 **마지막 게시만 기억합니다. BehaviorSubject <T> 또한 기본값인 T를 제공해야합니다. 즉, 이미 완료되지 않은 경우 모든 구독자가 즉시 값을받습니다.**

이 예제에서 값 'a'는 콘솔에 기록됩니다.
``` csharp
public void BehaviorSubjectExample()
{
  //Need to provide a default value.
  var subject = new BehaviorSubject<string>("a");
  subject.Subscribe(Console.WriteLine);
}
```

이 예제에서 값 'b'는 콘솔에 기록되지만 'a'는 기록되지 않습니다.
``` csharp
public void BehaviorSubjectExample2()
{
  var subject = new BehaviorSubject<string>("a");
  subject.OnNext("b");
  subject.Subscribe(Console.WriteLine);
}
```

이 예제에서 'b', 'c'및 'd'값은 모두 콘솔에 기록되지만 'a'는 기록되지 않습니다.
``` csharp
public void BehaviorSubjectExample3()
{
  var subject = new BehaviorSubject<string>("a");
  subject.OnNext("b");
  subject.Subscribe(Console.WriteLine);
  subject.OnNext("c");
  subject.OnNext("d");
}
```

마지막으로이 예제에서는 시퀀스가 완료 될 때 값이 게시되지 않습니다. 콘솔에 아무 것도 기록되지 않습니다.
``` csharp
public void BehaviorSubjectCompletedExample()
{
  var subject = new BehaviorSubject<string>("a");
  subject.OnNext("b");
  subject.OnNext("c");
  subject.OnCompleted();
  subject.Subscribe(Console.WriteLine);
}
```
버퍼 크기가 1인 ReplaySubject <T> (일반적으로 'replay one subject'이라고 함)와 BehaviorSubject <T> 사이에는 차이가 있음을 참고하십시오. BehaviorSubject <T>에는 초기 값이 필요합니다. 두 주제 중 어느 것도 완료되지 않았다는 가정하에 BehaviorSubject <T>에 값이 있음을 확신 할 수 있습니다. 그러나 ReplaySubject <T>로는 확신 할 수 없습니다. 이를 염두에두고 BehaviorSubject <T>를 완성하는 것은 이례적인 일입니다. 또 다른 차이점은 재생 한 피사체가 완료되면 다시 피할 수 있다는 것입니다. 그래서 완성 된 BehaviorSubject <T>에 가입하면 값을받지 못하게 될 수 있지만 ReplaySubject <T>를 사용하면 가능합니다.

BehaviorSubject <T>는 종종 클래스 속성과 연결됩니다. 사용자는 항상 값을 가지며 변경 알림을 제공 할 수 있기 때문에 필드를 속성으로 보완 할 수 있습니다.

## AsyncSubject<T>
AsyncSubject <T>는 값을 캐시하는 방식에서 Replay 및 Behavior 제목과 유사하지만 **마지막 값만 저장하고 시퀀스가 완료 될 때만 게시합니다.** AsyncSubject <T>의 일반적인 사용법은 하나의 값을 게시 한 다음 즉시 완료하는 것입니다. 이것은 이것이 Task <T>와 꽤 유사하다는 것을 의미합니다.

이 예제에서는 시퀀스가 완료되지 않으므로 값이 게시되지 않습니다. 콘솔에 값이 기록되지 않습니다.
``` csharp
static void Main(string[] args)
{
  var subject = new AsyncSubject<string>();
  subject.OnNext("a");
  WriteSequenceToConsole(subject);
  subject.OnNext("b");
  subject.OnNext("c");
  Console.ReadKey();
}
```

이 예제에서는 OnCompleted 메서드를 호출하여 마지막 값 'c'가 콘솔에 기록됩니다.
``` csharp
static void Main(string[] args)
{
  var subject = new AsyncSubject<string>();
  subject.OnNext("a");
  WriteSequenceToConsole(subject);
  subject.OnNext("b");
  subject.OnNext("c");
  subject.OnCompleted();
  Console.ReadKey();
}
```

## Implicit contracts (암시적 계약)
위에서 언급 한 것처럼 Rx로 작업 할 때 암시 적으로 필요한 연락처가 있습니다. 가장 중요한 점은 시퀀스가 완료되면 해당 시퀀스에서 더 이상의 활동이 발생하지 않는다는 것입니다. 시퀀스는 OnCompleted() 또는 OnError(Exception)의 두 가지 방법 중 하나로 완료 할 수 있습니다.

이 장에서 설명한 네 가지 주제는 모두 시퀀스가 이미 종료 된 후에 값, 오류 또는 완료를 게시하려는 시도를 무시함으로써이 암시적 계약을 처리합니다.

여기에 완성 된 시퀀스에 값 'c'를 게시하려는 시도가 있습니다. 하지만 'a'와 'b'값만 콘솔에 기록됩니다.
``` csharp
public void SubjectInvalidUsageExample()
{
  var subject = new Subject<string>();
  subject.Subscribe(Console.WriteLine);
  subject.OnNext("a");
  subject.OnNext("b");
  subject.OnCompleted();
  subject.OnNext("c");
}
```

## ISubject interfaces
이 장에서 설명하는 네 개의 주제는 각각 IObservable <T> 및 IObserver <T> 인터페이스를 구현하지만 다른 인터페이스 세트를 통해 수행합니다.
``` csharp
//Represents an object that is both an observable sequence as well as an observer.
public interface ISubject<in TSource, out TResult> : IObserver<TSource>, IObservable<TResult>
{
}
```

여기에 언급 된 모든 subject가 TSource와 TResult 모두에 대해 동일한 유형이므로, 이전 인터페이스의 상위 집합 인이 인터페이스를 구현합니다.
``` csharp
//Represents an object that is both an observable sequence as well as an observer.
public interface ISubject<T> : ISubject<T, T>, IObserver<T>, IObservable<T>
{
}
```

이러한 인터페이스는 널리 사용되지 않지만 공통된 기본 클래스를 공유하지 않는 피험자에게 유용합니다. 나중에 우리가 [Hot and Cold observables]을 발견 할 때 사용되는 Subject 인터페이스를 보게 될 것입니다.

## Subject factory
마지막으로 팩토리 메서드를 통해 주제를 만들 수도 있음을 알리는 것이 중요합니다. Subject가 IObservable <T> 및 IObserver <T> 인터페이스를 결합한다는 것을 고려하면, 사용자가 직접 결합 할 수있는 factory가 있어야한다는 것이 합리적입니다. Subject.Create (IObserver <TSource>, IObservable <TResult>) 팩토리 메서드는 이것을 단지 제공합니다.
``` csharp
// 제목에 메시지를 게시하는 데 사용 된 지정된 관찰자로부터 subject를 만듭니다.
// 그리고 관찰 할 수 있는 제목에서 보낸 메시지를 구독하는 데 사용
public static ISubject<TSource, TResult> Create<TSource, TResult>(
IObserver<TSource> observer, 
IObservable<TResult> observable)
{...}
```

subject는 Rx를 돌릴 수있는 편리한 방법을 제공하지만 일상적인 사용에는 권장하지 않습니다. 설명은 부록의 사용 지침에 나와 있습니다. subject를 사용하는 대신 파트 2에서 살펴볼 팩토리 메소드를 사용하십시오.

IObserver <T>와 IObservable <T>의 기본 유형과 보조 주제 유형은 Rx 지식을 구축 할 기반을 만듭니다. 이러한 단순 유형과 암시 적 계약을 이해하는 것이 중요합니다. 프로덕션 코드에서는 IObserver <T> 인터페이스와 subject 타입을 거의 사용하지 않을 수도 있지만, 이를 이해하고 Rx 에코 시스템에 어떻게 적용되는지는 여전히 중요합니다. IObservable <T> 인터페이스는 움직이는 데이터의 순서를 나타낼 수있는 지배적인 유형이므로 Rx 및 대부분의 작업에 대한 핵심 관심사를 구성합니다.