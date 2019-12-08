---
layout: single
title: "PART 3 - Taming the sequence : Advanced error handling"
related: true
classes: wide
categories: 
  - 프로그래밍
tags:
  - intro to RX
  - Reactive Programming
link: "http://introtorx.com/Content/v1.0.10621.0/11_AdvancedErrorHandling.html"
---

## Advanced error handling

예외가 발생합니다. 예외 자체는 나쁘지도 좋지도 않지만 우리가 키우거나 잡을 수있는 방법이 있습니다. 일부 예외는 예측할 수 있으며 DivideByZeroException과 같이 부적절한 코드로 인해 발생합니다. FileNotFoundException 또는 TimeoutException과 같은 I / O 예외와 같은 방어적인 코딩을 사용하면 다른 예외를 방지 할 수 없습니다. 이러한 경우 예외를 적절히 처리해야합니다. 사용자에게 일종의 오류 메시지를 제공하거나 오류를 기록하거나 재 시도하는 것은 이러한 예외를 처리 할 수있는 모든 잠재적 인 방법입니다.

Observer<T> 인터페이스 및 Subscribe 확장 메서드는 오류로 종료되는 시퀀스를 처리 할 수있는 기능을 제공하지만 시퀀스가 종료 된 상태로 둡니다. 또한 다른 예외 유형을 처리 할 수있는 방법을 제공하지 않습니다. 오류 처리기를 구성하여 모나드에 남아있게하는 기능적 접근 방식이 더 유용 할 것입니다.

## Control flow constructs

marble 다이어그램을 사용하여 다양한 제어 흐름을 처리하는 다양한 방법을 살펴 봅니다. 일반적인 .NET 코드와 마찬가지로 try / catch / finally와 같은 흐름 제어 구조가 있습니다. 이 장에서는 관찰 가능한 시퀀스에 어떻게 적용 할 수 있는지 살펴 봅니다.

### Catch

SEH의 catch(Structured Exception Handling)를 사용하면, 예외를 삼키는 다른 예외를 포장 또는 다른 로직을 수행 할 수있는 옵션이 있습니다.

우리는 이미 관찰 가능한 시퀀스가 OnError 구조로 잘못된 상황을 처리 할 수 있다는 것을 이미 알고 있습니다. OnError 알림을 처리하는 Rx의 유용한 메소드는 Catch 확장 메소드입니다. Catch를 사용하면 특정 Exception 유형을 가로 채고 다른 시퀀스로 계속할 수 있습니다.

다음은 catch의 간단한 오버로드에 대한 서명입니다.

``` csharp
public static IObservable<TSource> Catch<TSource>(
  this IObservable<TSource> first,
  IObservable<TSource> second)
{
  ...
}
```

### Swallowing exceptions

Rx를 사용하면 SEH와 비슷한 방식으로 예외를 포착하고 삼킬 수 있습니다. 그것은 아주 간단합니다. 우리는 Catch 확장 메서드를 사용하고 빈 시퀀스를 두 번째 값으로 제공합니다.

우리는 marble 다이어그램으로 이런 예외를 삼킨 예외를 나타낼 수 있습니다.

``` csharp
S1--1--2--3--X
S2            -|
R --1--2--3----|
```

여기서 S1은 오류 (X)로 끝나는 첫 번째 시퀀스를 나타냅니다. S2는 연속 시퀀스, 빈 시퀀스입니다. R은 S1으로 시작하는 결과 시퀀스이고 S1이 종료되면 S2로 계속됩니다.

``` csharp
var source = new Subject<int>();
var result = source.Catch(Observable.Empty<int>());
result.Dump("Catch"););
source.OnNext(1);
source.OnNext(2);
source.OnError(new Exception("Fail!"));
/*
Output:

Catch-->1
Catch-->2
Catch completed
*/
```

위의 예는 모든 유형의 예외를 잡아서 삼켜 버릴 것입니다. 이것은 SEH에서 다음과 다소 동일합니다.

``` csharp
try
{
  DoSomeWork();
}
catch
{
}
```

SEH에서 일반적으로 피해지는 것과 마찬가지로, Rx에서 삼키는 오류 사용을 제한하려고합니다. 그러나 처리하려는 특정 예외가있을 수 있습니다. Catch에는 예외 유형을 지정할 수있는 오버로드가 있습니다. 다음 코드가 TimeoutException을 잡을 수있게 해줍니다.

``` csharp
try
{
//
}
catch (TimeoutException tx)
{
//
}
```

Rx는 또한이 문제를 해결하기 위해 Catch의 오버로드를 제공합니다.

``` csharp
public static IObservable<TSource> Catch<TSource, TException>(
  this IObservable<TSource> source,
  Func<TException, IObservable<TSource>> handler)
  where TException : Exception
{
  ...
}
```

다음 Rx 코드를 사용하면 TimeoutException을 catch 할 수 있습니다. 두 번째 시퀀스를 제공하는 대신 예외를 취하여 시퀀스를 반환하는 함수를 제공합니다. 이렇게하면 factory를 사용하여 계속 작업을 만들 수 있습니다. 이 예에서는 오류 시퀀스에 값 -1을 더한 다음 오류 시퀀스를 완료합니다.

``` csharp
var source = new Subject<int>();
var result = source.Catch<int, TimeoutException>(tx=>Observable.Return(-1));
result.Dump("Catch");
source.OnNext(1);
source.OnNext(2);
source.OnError(new TimeoutException());
/*
Output:

Catch-->1
Catch-->2
Catch-->-1
Catch completed
*/
```

시퀀스가 TimeoutException에 캐스트 할 수없는 Exception으로 종료되는 경우 오류가 발견되지 않고 구독자로 전달됩니다.

``` csharp
var source = new Subject<int>();
var result = source.Catch<int, TimeoutException>(tx=>Observable.Return(-1));
result.Dump("Catch");
source.OnNext(1);
source.OnNext(2);
source.OnError(new ArgumentException("Fail!"));
/*
Output:

Catch-->1
Catch-->2
Catch failed-->Fail!
*/
```

### Finally

