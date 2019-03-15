---
layout: single
title: "PART 1 - Getting started : Lifetime management"
related: false
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
