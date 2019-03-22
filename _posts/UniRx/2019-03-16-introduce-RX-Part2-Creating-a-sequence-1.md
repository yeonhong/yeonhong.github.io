---
layout: single
title: "PART 2 - Sequence basics : Creating a sequence - 1. Simple factory methods"
related: true
permalink: /docs/intro to RX/
categories: 
  - 프로그래밍
tags:
  - intro to RX
  - Reactive Programming
link: "http://introtorx.com/Content/v1.0.10621.0/04_CreatingObservableSequences.html#CreationOfObservables"
---

# PART 2 - Sequence basics
따라서 참여하고 Rx 코드를 작성하려고하지만 시작하려면 어떻게해야합니까? 우리는 주요 유형을 살펴 보았지만 IObserver <T> 또는 IObservable <T>의 자체 구현을 작성해서는 안되며 subject를 사용하는 것보다 [factory methods](https://jdm.kr/blog/180)을 선호해야한다는 것을 알고 있습니다. 관찰 할 수있는 시퀀스가 있더라도 원하는 데이터를 선택하는 방법은 무엇입니까? 우리는 관측 가능한 시퀀스를 생성하고, 값을 얻고, 우리가 원하는 값을 추출하는 기초를 이해해야합니다.

Part 2에서는 관찰 가능한 시퀀스를 구성하고 쿼리하기위한 기본 사항을 살펴 봅니다. 우리는 LINQ가 Rx를 사용하고 이해하는데 있어 기본적이라고 주장합니다. 자세히 살펴보면 함수 프로그래밍 개념이 LINQ를 심층적으로 이해하고 Rx를 마스터 할 수 있다는 점에서 핵심 요소라는 것을 알 수 있습니다. 이러한 이해를 돕기 위해 쿼리 연산자를 세 가지 주요 그룹으로 분류합니다. 이 그룹들 각각은 다른 연산자가 구축 될 수있는 루트 연산자를 가지고 있음을 증명합니다. 이 해체 운동은 Rx, 함수 프로그래밍 및 쿼리 작성에 대한 더 깊은 통찰력을 제공 할뿐만 아니라, 일반 RX 연산자가 사용자의 요구에 부합하지 않는 사용자 정의 연산자를 만들 수 있는 기능을 갖추어야 합니다.

# Creating a sequence
이전 장에서 우리는 첫 번째 Rx 확장 메서드 인 Subscribe 메서드와 오버로드를 사용했습니다. Subject.Create()에서도 첫 번째 factory methods를 보았습니다. 우리는 Rx를 만들기 위해 IObservable <T>을 풍부하게 만드는 방대한 다른 방법들을 살펴볼 것입니다. Rx 라이브러리에 공개 인스턴스 메소드가 비교적 적다는 사실을 알면 놀랄 수 있습니다. 그러나 많은 수의 public static 메서드와 더 많은 수의 확장 메서드가 있습니다. 수많은 메소드와 overloads로 인해 카테고리로 분류 할 것입니다.
> 일부 독자는 다음 몇 장의 부분을 건너 뛸 수 있다고 느낄 수도 있습니다. LINQ 및 기능 구성에 대해 매우 확신하는 경우에만 그렇게하는 것이 좋습니다. 이 책의 목적은 Rx에 대한 단계별 소개를 제공하는 것입니다. 독자 여러분, 독자의 목표는 소프트웨어에 Rx를 적용하는 것입니다. Rx의 적절한 응용 프로그램은 Rx의 기본 사항에 대한 올바른 이해를 통해 제공됩니다. 사람들이 Rx로 만들게 될 가장 일반적인 실수는 Rx가 구축 된 원리에 대한 오해 때문입니다. 이를 염두에두고 계속 읽어 보시기 바랍니다.

새로운 주제의 사례를 단순히 구축한 주요 유형에 대한 우리의 조사를 계속하는 것이 현명합니다. 우리의 첫 번째 범주의 방법은 생성적인 방법입니다 : 간단한 방법으로 IObservable <T> 시퀀스의 인스턴스를 만들 수 있습니다. 이러한 메서드는 일반적으로 시퀀스를 생성하기 위해 하나의 형식 값을 사용하거나 형식 자체를 사용합니다. 함수형 프로그래밍에서 이것은 아나모피즘 또는 '전개'라고 합니다.

## Simple factory methods

### Observable.Return
가장 기본적인 예제로 Observable.Return <T> (T value)을 소개합니다. 이 메서드는 T값을 사용하고 단일 값으로 IObservable <T>를 반환 한 다음 완료합니다. 그것은 관찰 가능한 순서로 T의 값을 펼쳤습니다.
``` csharp
var singleValue = Observable.Return<string>("Value");
//which could have also been simulated with a replay subject
var subject = new ReplaySubject<string>();
subject.OnNext("Value");
subject.OnCompleted();
```
위 예제에서 우리는 팩토리 메서드를 사용하거나 replay subject를 사용하여 동일한 효과를 얻을 수 있습니다. 명백한 차이점은 팩토리 메서드는 단 한 줄이고 명령형 프로그래밍 스타일보다 선언적이라는 점입니다. 위의 예제에서 type 매개 변수를 string으로 지정 했으므로 제공된 인수에서 유추 할 수 있으므로 필요하지 않습니다.
``` csharp
singleValue = Observable.Return<string>("Value");
//Can be reduced to the following
singleValue = Observable.Return("Value");
```

### Observable.Empty
다음 두 예제는 관찰 가능한 시퀀스로 펼치기 위해 type 매개 변수 만 필요합니다. 첫 번째는 Observable.Empty <T> ()입니다. 이렇게하면 빈 IObservable <T> 즉, OnCompleted 알림 만 게시됩니다.
``` csharp
var empty = Observable.Empty<string>();
//Behaviorally equivalent to
var subject = new ReplaySubject<string>();
subject.OnCompleted();
```

### Observable.Never
Observable.Never <T> () 메서드는 알림없이 무한 시퀀스를 반환합니다.
``` csharp
var never = Observable.Never<string>();
//similar to a subject without notifications
var subject = new Subject<string>();
```

### Observable.Throw
Observable. <T> (Exception) 메서드가 형식 매개 변수 정보를 필요로하는 경우 OnError와 함께 Exception이 필요합니다. 이 메서드는 팩토리에 전달 된 예외를 포함하는 OnError 알림을 하나만 사용하여 시퀀스를 만듭니다.
``` csharp
var throws = Observable.Throw<string>(new Exception()); 
//Behaviorally equivalent to
var subject = new ReplaySubject<string>(); 
subject.OnError(new Exception());
```

### Observable.Create
Create factory method는 위의 작성 메소드와 약간 다릅니다. create라는 시그니처 자체는 처음에는 다소 압도적 일 수 있지만 일단 사용하면 매우 자연스러워집니다.
``` csharp
//Creates an observable sequence from a specified Subscribe method implementation.
public static IObservable<TSource> Create<TSource>(
Func<IObserver<TSource>, IDisposable> subscribe)
{...}
public static IObservable<TSource> Create<TSource>(
Func<IObserver<TSource>, Action> subscribe)
{...}
```
본질적으로이 메서드를 사용하면 구독 할 때마다 실행될 대리자를 지정할 수 있습니다. 구독을 만든 IObserver <T>는 대리인에게 전달되어 필요에 따라 OnNext / OnError / OnCompleted 메서드를 호출 할 수 있습니다. 이것은 IObserver <T> 인터페이스에 관심을 가져야하는 몇 가지 시나리오 중 하나입니다. 대리자는 IDisposable을 반환하는 Func입니다. 이 IDisposable 구독자가 구독을 처분 할 때 Dispose () 메서드가 호출됩니다.

Create Factory 메서드는 사용자 정의 관찰 가능 시퀀스를 구현하는 기본 방법입니다. subject의 사용은 주로 샘플과 테스트의 영역에 남아 있어야합니다. subject는 Rx를 시작하기에 좋은 방법입니다. 새로운 개발자의 학습 곡선은 줄어들지 만 Create 메서드를 사용하면 여러 가지 문제가 발생할 수 있습니다. Rx는 사실 함수 프로그래밍 패러다임입니다. subject를 사용한다는 것은 잠재적으로 돌연변이가 될 수있는 상태를 관리하고 있다는 것을 의미합니다. Mutating 상태와 비동기 프로그래밍은 매우 어렵습니다. 또한 많은 연산자 (확장 메소드)가 정확하고 일관된 가입 기간 및 시퀀스가 유지되도록 신중하게 작성되었습니다. 당신이 subject를 소개 할 때 이것을 깨뜨릴 수 있습니다. 향후 릴리스에서는 명시적으로 subject를 사용하면 성능이 크게 저하 될 수도 있습니다.
> subject를 사용하지 말라는 소리..

Create 메서드는 IObservable 인터페이스를 구현하는 사용자 지정 형식을 만드는 것보다 더 좋습니다. 관찰자 / 관찰 가능한 인터페이스를 직접 구현할 필요가 없습니다. Rx는 알림 및 구독의 스레드 안전성과 같이 생각할 수없는 복잡함을 해결합니다.

Create 메서드가 subject보다 갖는 중요한 이점은 시퀀스가 느슨하게 평가된다는 것입니다. 게으른 평가는 Rx의 매우 중요한 부분입니다. 그것은 우리가 나중에 보게 될 시퀀스의 스케줄링 및 조합과 같은 다른 강력한 기능에 대한 문을 열어줍니다. 대리인은 가입이 이루어질 때만 호출됩니다.

이 예에서 어떻게 첫 return을 하는지 보여준다. (standard blocking eagerly evaluated call 를 통해) , 우리는 게으른 평가에 의해 차단하지 않고 관찰 순서를 반환하는 올바른 방법을 보여줍니다.
``` csharp
private IObservable<string> BlockingMethod()
{
  //구독해도 1초후에 받음.
  var subject = new ReplaySubject<string>();
    subject.OnNext("a");
    subject.OnNext("b");
    subject.OnCompleted();
    Thread.Sleep(1000);
    return subject; 
}
private IObservable<string> NonBlocking()
{
  //구독해도 1초를 안기다림.
  return Observable.Create<string>(
  (IObserver<string> observer) =>
  {
    observer.OnNext("a"); 
    observer.OnNext("b");
    observer.OnCompleted();
    Thread.Sleep(1000);
    return Disposable.Create(() => Console.WriteLine("Observer has unsubscribed"));
    //or can return an Action like 
    //return () => Console.WriteLine("Observer has unsubscribed"); 
  });
}
```
예제가 다소 의도되었지만 소비자가 blocking eagerly evaluated call하면 실제 구독 여부와 관계없이 IObservable <string>을 받기까지 최소 1초 동안 차단됩니다. 비 차단 방법은 느슨하게 평가되므로 소비자는 IObservable <string>을 즉시받으며 구독하는 경우 스레드 수면 비용 만 발생합니다.

연습으로 Create 메서드를 사용하여 Empty, Return, Never & Throw 확장 메서드를 직접 만들어보십시오. 지금 Visual Studio 또는 LINQPad를 사용할 수있는 경우 가능한 빨리 코드를 작성해보세요.
---

Observable.Create를 사용하여 Empty, Return, Never 및 Throw를 다시 작성한 예 :
``` csharp
public static IObservable<T> Empty<T>()
{
  return Observable.Create<T>(o =>
  {
    o.OnCompleted();
    return Disposable.Empty;
  });
}
public static IObservable<T> Return<T>(T value)
{
  return Observable.Create<T>(o =>
  {
    o.OnNext(value);
    o.OnCompleted();
    return Disposable.Empty;
  });
}
public static IObservable<T> Never<T>()
{
  return Observable.Create<T>(o =>
  {
    return Disposable.Empty;
  });
}
public static IObservable<T> Throws<T>(Exception exception)
{
  return Observable.Create<T>(o =>
  {
    o.OnError(exception);
    return Disposable.Empty;
  });
}
```
Observable.Create는 원한다면 우리 자신의 factory method를 구축 할 수있는 힘을 제공한다는 것을 알 수 있습니다. 우리가 모든 OnNext 알림을 생성 한 후 에는 각 예제에서 구독 토큰 (IDisposable 구현) 만 반환 할 수 있습니다. 이는 우리가 제공하는 delegate 내부에서 완전히 순차적이기 때문입니다. 또한 토큰을 무의미하게 만듭니다. 이제보다 유용한 방법으로 반환 값을 사용할 수있는 방법을 살펴 보겠습니다. 첫 번째 예제는 우리 delegate 내부에서 타이머를 만들 때마다 관찰자의 OnNext를 호출하는 Timer를 만드는 예제입니다.
``` csharp
//Example code only
public void NonBlocking_event_driven()
{
  var ob = Observable.Create<string>(
  observer =>
  {
    var timer = new System.Timers.Timer();
    timer.Interval = 1000;
    timer.Elapsed += (s, e) => observer.OnNext("tick");
    timer.Elapsed += OnTimerElapsed;
    timer.Start();
    return Disposable.Empty;
  });
  var subscription = ob.Subscribe(Console.WriteLine);
  Console.ReadLine();
  subscription.Dispose();
}
private void OnTimerElapsed(object sender, ElapsedEventArgs e)
{
  Console.WriteLine(e.SignalTime);
}

/* output
tick
01/01/2012 12:00:00
tick
01/01/2012 12:00:01
tick
01/01/2012 12:00:02
01/01/2012 12:00:03
01/01/2012 12:00:04
01/01/2012 12:00:05
*/
```
위의 예제는 망가졌습니다. 구독을 dispose하면 화면에 "tick"가 표시되는 것을 멈춥니다. 그러나 우리는 두 번째 이벤트 처리기 "OnTimerElasped"를 릴리스하지 않았으며 타이머의 인스턴스를 처리하지 않았으므로 처리 후에도 ElapsedEventArgs.SignalTime을 콘솔에 기록합니다. 매우 간단한 해결 방법은 timer를 IDisposable 토큰으로 반환하는 것입니다.
```csharp
//Example code only
var ob = Observable.Create<string>(
observer =>
{
  var timer = new System.Timers.Timer();
  timer.Interval = 1000;
  timer.Elapsed += (s, e) => observer.OnNext("tick");
  timer.Elapsed += OnTimerElapsed;
  timer.Start();
  return timer; //여기가 다르다. timer 자체를 IDisposable 토큰으로 반환한다고 하는것.
});
```
이제 소비자가 구독을 처분하면 기본 타이머도 처분됩니다.

Observable.Create에는 또한 Func가 IDisposable 대신 Action을 반환해야하는 오버로드가 있습니다. 위의 예와 비슷한 예제에서이 함수는 이벤트 처리기의 등록을 취소하는 액션을 사용하여 타이머에 대한 참조를 유지함으로써 메모리 누수를 방지하는 방법을 보여줍니다.
```csharp
//Example code only
var ob = Observable.Create<string>(
  observer =>
  {
    var timer = new System.Timers.Timer();
    timer.Enabled = true;
    timer.Interval = 100;
    timer.Elapsed += OnTimerElapsed;
    timer.Start();
    return ()=>{
      timer.Elapsed -= OnTimerElapsed;
      timer.Dispose();
    };
  }
);
```
이 마지막 몇 가지 예제에서는 Observable.Create 메서드를 사용하는 방법을 보여 줬습니다. 이것들은 단지 예일뿐입니다. 타이머에서 값을 생성하는 더 좋은 방법이 실제로 있습니다. Observable.Create가 관찰 가능한 시퀀스를 생성하는 느슨하게 평가 된 방법을 제공한다는 것을 보여줍니다. 우리는 동시성과 스케줄링을 커버 할 때, 특히 책 전체에 걸쳐 Create factory 메소드의 게으른 평가와 적용에 더 깊이 파고 들어갈 것입니다.
