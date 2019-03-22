---
layout: single
title: "PART 2 - Sequence basics : Creating a sequence - 2. Functional unfolds"
related: true
categories: 
  - 프로그래밍
tags:
  - intro to RX
  - Reactive Programming
link: "http://introtorx.com/Content/v1.0.10621.0/04_CreatingObservableSequences.html#Unfold"
---

## Functional unfolds
functional programmer로서 당신은 잠재적으로 무한 시퀀스를 전개 할 수있는 방법을 기대합니다. Observable.Create에서 직면 할 수있는 문제는 무한 시퀀스를 만드는 어색한 방법 일 수 있다는 것입니다. 위의 타이머 예제는 무한 시퀀스의 예제이며 간단한 구현이지만 모든 작업을 System.Timers.Timer 클래스에 효과적으로 위임하는 코드에 대한 성가신 양입니다. Observable.Create 메서드는 corecursion을 사용하여 전개 시퀀스에 대한 지원이 부족합니다.

### Corecursion
Corecursion은 다음 상태를 만들기 위해 현재 상태에 적용하는 함수입니다. 값을 취하여 핵심 회귀를 사용하여 그 값을 확장하는 함수를 적용하고 반복하여 시퀀스를 생성 할 수 있습니다. 간단한 예는 값 1을 시드로 취하고 주어진 값을 1 씩 증가시키는 함수 일 수 있습니다. 이것은 [1,2,3,4,5 ...]의 순서를 만드는데 사용될 수 있습니다.

corecursion을 사용하여 IEnumerable <int> 시퀀스를 만드는 것은 yield return 구문으로 간단 해집니다.
``` csharp
private static IEnumerable<T> Unfold<T>(T seed, Func<T, T> accumulator)
{
  var nextValue = seed;
  while (true)
  {
    yield return nextValue;
    nextValue = accumulator(nextValue);
  }
}
```
위의 코드는 이와 같은 자연 수의 시퀀스를 생성하는 데 사용될 수 있습니다.
``` csharp
var naturalNumbers = Unfold(1, i => i + 1);
Console.WriteLine("1st 10 Natural numbers");
foreach (var naturalNumber in naturalNumbers.Take(10))
{
  Console.WriteLine(naturalNumber);
}
/* outpue
1st 10 Natural numbers
1
2
3
4
5
6
7
8
9
10
*/
```
Take (10)는 무한 시퀀스를 종료하는 데 사용됩니다.

무한 및 임의의 길이 시퀀스는 매우 유용 할 수 있습니다. 먼저 Rx와 함께 제공되는 일부를 살펴보고 무한 관측 시퀀스의 생성을 일반화 할 수있는 방법을 고려해 보겠습니다.

### Observable.Range
Observable.Range (int, int)는 단순히 정수 범위를 반환합니다. 첫 번째 정수는 초기 값이고 두 번째 정수는 얻을 값의 수입니다. 이 예제에서는 값 '10'에서 '24'를 쓰고 완료합니다.
``` csharp
var range = Observable.Range(10, 15);
range.Subscribe(Console.WriteLine, ()=>Console.WriteLine("Completed"));
```

### Observable.Generate
Observable.Create를 사용하여 Range 팩토리 메서드를 에뮬레이트하는 것은 어렵습니다. 코드를 게으르게 평가해야한다는 원칙을 존중하고이를 선택하는 소비자는 구독 리소스를 처분 할 수 있어야합니다. 이것은 우리가 더 풍부한 전개를 제공하기 위해 핵심 회칙을 사용할 수있는 곳입니다. Rx에서 unfold 메소드는 Observable.Generate라고합니다.

Observable.Generate의 간단한 버전은 다음 매개 변수를 사용합니다.
* 초기 상태
* 시퀀스 종료시기를 정의하는 술어
* 현재 상태에 적용하여 다음 상태를 생성하는 함수
* 상태를 원하는 출력으로 변환하는 함수

``` csharp
public static IObservable<TResult> Generate<TState, TResult>(
  TState initialState, 
  Func<TState, bool> condition, 
  Func<TState, TState> iterate, 
  Func<TState, TResult> resultSelector)
```

연습으로 Observable.Generate를 사용하여 자신의 Range 팩토리 메소드를 작성하십시오.

조건부 술어에 대한 시드와 값을 제공하는 범위 서명 범위 (int start, int count)를 고려하십시오. 당신은 각각의 새로운 가치가 이전의 가치에서 파생 된 방법을 압니다. 이것은 반복 함수가됩니다. 마지막으로 상태를 변환 할 필요가 없으므로 결과 선택기 기능이 매우 간단 해집니다.

``` csharp
//Example code only
public static IObservable<int> Range(int start, int count)
{
  var max = start + count;
  return Observable.Generate(
  start, 
  value => value < max, 
  value => value + 1, 
  value => value);
}
```

### Observable.Interval
이 장의 앞부분에서 우리는 연속적인 알림 시퀀스를 생성하기 위해 관찰 할 수있는 System.Timers.Timer를 사용했습니다. 당시 예제에서 언급했듯이 이것은 Rx에서 타이머를 사용하는 기본 방법이 아닙니다. Rx가 우리에게이 기능을 제공하는 연산자를 제공하기 때문에이를 사용하지 않으면 바퀴를 다시 발명하는 것이라고 주장 할 수 있습니다. 더 중요한 것은 Rx 운영자가 기본 타이머를 쉽게 대체 할 수있는 스케줄러를 대신 할 수 있기 때문에 타이머 작업에 선호되는 방법입니다. 위의 예에서 선택할 수있는 타이머는 적어도 세 가지가 있습니다.
* System.Timers.Timer
* System.Threading.Timer
* System.Windows.Threading.DispatcherTimer

스케줄러를 통해 타이머를 추상화함으로써 우리는 여러 플랫폼에서 동일한 코드를 재사용 할 수 있습니다. 더 중요한 것은 플랫폼 독립적 인 코드를 작성하는 것보다 테스트 이중 스케줄러 / 타이머를 대체하여 테스트를 가능하게하는 것입니다. 스케줄러는이 장의 범위를 벗어난 복잡한 주제이지만 스케줄 및 스레딩에 대해서는 나중에 설명합니다.

일정 시간 사건을 다루는 세 가지 더 좋은 방법이 있으며, 각각은 이전의 사건을 더 일반화합니다. 첫 번째는 Observable.Interval (TimeSpan)으로, 선택한 빈도에 따라 0부터 시작하여 증분 값을 게시합니다. 이 예는 250 밀리 초마다 값을 게시합니다.
``` csharp
var interval = Observable.Interval(TimeSpan.FromMilliseconds(250));
  interval.Subscribe(
  Console.WriteLine, 
  () => Console.WriteLine("completed"));
/* output
0
1
2
3
4
5
*/
```
일단 구독하면 시퀀스를 중지하기 위해 구독을 처분해야합니다. 무한 시퀀스의 예입니다.

### Observable.Timer
일정 시간 기반 시퀀스를 생성하는 두 번째 팩토리 방법은 Observable.Timer입니다. 몇 가지 오버로드가 있습니다. 그 중 첫 번째는 매우 단순 해 보일 것입니다. Observable.Timer의 가장 기본적인 오버로드는 Observable처럼 TimeSpan을 사용합니다. 인터벌은 않습니다. Observable.Timer는 시간이 경과 한 후에 하나의 값 (0) 만 게시 한 다음 완료합니다.
``` csharp
var timer = Observable.Timer(TimeSpan.FromSeconds(1));
timer.Subscribe(
  Console.WriteLine, 
  () => Console.WriteLine("completed"));
/* output
0
completed
*/
```
또는 dueTime 매개 변수에 DateTimeOffset을 제공 할 수 있습니다. 그러면 값 0이 생성되고 기한 내에 완료됩니다.

추가 오버로드 집합은 이후 값을 생성하는 기간을 나타내는 TimeSpan을 추가합니다. 이제 우리는 무한한 시퀀스를 생성하고 Observable.Timer에서 Observable.Interval을 구성 할 수 있습니다.
``` csharp
public static IObservable<long> Interval(TimeSpan period)
{
  return Observable.Timer(period, period);
}
```
이제 int가 아닌 IObservable을 반환합니다. Observable.Interval은 항상 첫 번째 값을 생성하기 전에 주어진 기간을 기다리는 반면, Observable.Timer 과부하는 사용자가 선택할 때 시퀀스를 시작할 수있는 기능을 제공합니다. Observable.Timer를 사용하면 다음을 작성하여 즉시 시작되는 간격 순서를 가질 수 있습니다.
``` csharp
Observable.Timer(TimeSpan.Zero, period);
```

이것은 우리에게 세 번째 방법이자 타이머 관련 시퀀스를 생성하는 가장 일반적인 방법 인 Observable.Generate로 돌아갑니다. 이번에는 더 복잡한 오버로드를 살펴보고 다음 값의 만기 시간을 지정하는 함수를 제공 할 수 있습니다.
``` csharp
public static IObservable<TResult> Generate<TState, TResult>(
  TState initialState, 
  Func<TState, bool> condition, 
  Func<TState, TState> iterate, 
  Func<TState, TResult> resultSelector, 
  Func<TState, TimeSpan> timeSelector)
```

이 오버로드와 특별히 timeSelector 인수를 사용하여 Observable.Timer와 Observable.Interval의 자체 구현을 생성 할 수 있습니다.
``` csharp
public static IObservable<long> Timer(TimeSpan dueTime)
{
return Observable.Generate(
    0l,
    i => i < 1,
    i => i + 1,
    i => i,
    i => dueTime);
}
public static IObservable<long> Timer(TimeSpan dueTime, TimeSpan period)
{
return Observable.Generate(
    0l,
    i => true,
    i => i + 1,
    i => i,
    i => i == 0 ? dueTime : period);
}
public static IObservable<long> Interval(TimeSpan period)
{
  return Observable.Generate(
    0l,
    i => true,
    i => i + 1,
    i => i,
    i => period);
}
```

이것은 Observable.Generate를 사용하여 무한 시퀀스를 생성하는 방법을 보여줍니다. Observable.Generate를 사용하여 다양한 속도로 값을 산출하는 연습과 같이 독자에게 독자에게 맡깁니다. 나는이 방법들을 날마다 일하는 것뿐만 아니라 특히 더미 데이터를 생성하는 데 매우 중요하게 생각합니다.

## Transitioning into IObservable<T>
관찰 가능한 시퀀스의 생성은 함수적 프로그래밍, 즉 코어 커미션과 전개의 복잡한 측면을 다룹니다. 기존의 동기식 또는 비동기식 패러다임을 Rx 패러다임으로 전환함으로써 시퀀스를 시작할 수도 있습니다.

### From delegates
Observable.Start 메서드를 사용하면 **장시간 실행되는 Func <T> 또는 Action을 단일 값 관찰 가능 시퀀스로 바꿀 수 있습니다.** 디폴트에서는, 처리는 ThreadPool thread로 비동기 적으로 행해집니다. 
사용하는 오버로드가 Func <T>이면 반환 유형은 IObservable <T>입니다. 함수가 값을 반환하면 해당 값이 게시되고 시퀀스가 완료됩니다. 
Action을 취하는 오버로드를 사용하면 반환되는 시퀀스는 IObservable <Unit> 유형이됩니다. 
단위 유형은 함수 프로그래밍 구조이며 void와 유사합니다. 이 경우 Unit은 Action이 완료되었음을 알리기 위해 사용되지만, 
Sequence가 Unit의 직후에 곧바로 완성되기 때문에 이것은 중요하지 않습니다. 단위 유형 자체에는 값이 없습니다. OnNext 알림을 위한 빈 페이로드 역할을합니다. 
다음은 두 가지 오버로드를 사용하는 예입니다.
``` csharp
static void StartAction()
{
  var start = Observable.Start(() =>
  {
    Console.Write("Working away");
    for (int i = 0; i < 10; i++)
    {
      Thread.Sleep(100);
      Console.Write(".");
    }
  });
  start.Subscribe(
   unit => Console.WriteLine("Unit published"), 
    () => Console.WriteLine("Action completed"));
}
static void StartFunc()
{
  var start = Observable.Start(() =>
  {
    Console.Write("Working away");
    for (int i = 0; i < 10; i++)
    {
      Thread.Sleep(100);
      Console.Write(".");
    }
    return "Published value";
  });
  start.Subscribe(
    Console.WriteLine, 
    () => Console.WriteLine("Action completed"));
}
```
Observable.Start와 Observable의 차이점에 유의하십시오. lazily 함수에서 값을 평가하고, Return 값을 열렬하게 제공합니다. 이것은 Start를 Task과 매우 흡사하게 만듭니다. 또한 각 기능을 언제 사용할지 혼란 스러울 수 있습니다. 둘 다 유효한 도구이며 선택은 문제 공간의 맥락에 달려 있습니다. Task는 전산 작업을 병렬 처리하고 연산이 많은 작업을 위해 연속을 통해 워크 플로를 제공하는 데 적합합니다. Task는 단일 값 의미 체계를 문서화하고 시행하는 이점도 있습니다. Start를 사용하면 계산이 많은 작업을 주로 관찰 가능한 시퀀스로 구성된 기존 코드베이스에 통합하는 좋은 방법입니다.

### From events
이 책의 앞 부분에서 논의했듯이 .NET에는 이미 반응 형 이벤트 기반 프로그래밍 모델을 제공하는 이벤트 모델이 있습니다. Rx는 더 강력하고 유용한 프레임 워크이지만, 파티에 늦었고 기존 이벤트 모델과 통합해야합니다. Rx는 이벤트를 가져 와서 관찰 가능한 시퀀스로 변환하는 메소드를 제공합니다. 당신이 사용할 수있는 몇 가지 종류가 있습니다. 다음은 공통 이벤트 패턴의 선택입니다.
``` csharp
//Activated delegate is EventHandler
var appActivated = Observable.FromEventPattern(
  h => Application.Current.Activated += h,
  h => Application.Current.Activated -= h);
//PropertyChanged is PropertyChangedEventHandler
var propChanged = Observable.FromEventPattern
  <PropertyChangedEventHandler, PropertyChangedEventArgs>(
    handler => handler.Invoke,
    h => this.PropertyChanged += h,
    h => this.PropertyChanged -= h);
//FirstChanceException is EventHandler<FirstChanceExceptionEventArgs>
var firstChanceException = Observable.FromEventPattern<FirstChanceExceptionEventArgs>(
  h => AppDomain.CurrentDomain.FirstChanceException += h,
  h => AppDomain.CurrentDomain.FirstChanceException -= h);  
```
overload가 혼란 스러울 수 있지만 중요한 것은 이벤트의 서명이 무엇인지 알아내는 것입니다. 
서명이 기본 EventHandler 대리자 인 경우 첫 번째 예제를 사용할 수 있습니다. 
대리자가 EventHandler의 하위 클래스 인 경우 두 번째 예제를 사용하고 EventHandler 하위 클래스와 EventArg의 특정 유형을 제공해야합니다. 
대리자가 새로운 일반 EventHandler <TEventArgs> 인 경우에는 세 번째 예제를 사용하고 이벤트 인수의 제네릭 형식을 지정해야합니다.

프로퍼티가 변경된 이벤트를 관찰 가능한 시퀀스로 표시하고자하는 것은 매우 일반적입니다. 이러한 이벤트는 INotifyPropertyChanged 인터페이스, DependencyProperty 또는 표현하는 Property에 적절하게 지정된 이벤트를 통해 노출 될 수 있습니다. 이런 종류의 일을하기 위해 독자적인 래퍼를 작성하는 것을보고 있다면, 먼저 http://Rxx.codeplex.com의 Rxx 라이브러리를 살펴볼 것을 강력하게 제안 할 것입니다. 이들 중 많은 부분이 매우 우아한 방식으로 준비되어 있습니다.

### From Task
Rx는 다른 기존 패러다임에서 Observable 패러다임으로 변환하기 위해 유용하고 잘 명명 된 오버로드 집합을 제공합니다. ToObservable () 메서드 오버로드는 전환을 수행하는 간단한 경로를 제공합니다.

이전에 언급했듯이 AsyncSubject <T>는 Task <T>와 비슷합니다. 둘 다 비동기 소스에서 단일 값을 리턴합니다. 또한 값에 대한 반복 요청이나 지연 요청에 대한 결과를 캐시합니다. 첫 번째 ToObservable () 확장 메서드 오버로드는 Task <T>의 확장입니다. 구현은 간단합니다.
* 작업이 이미 RanToCompletion 상태에 있으면 값이 시퀀스에 추가 된 다음 시퀀스가 완료됩니다.
* 작업이 취소 된 경우 시퀀스에 TaskCanceledException 오류가 발생합니다.
* 작업이 Faulted이면 작업의 내부 예외와 함께 시퀀스에 오류가 발생합니다.
* 작업이 아직 완료되지 않은 경우 작업에 계속 작업을 추가하여 위의 작업을 적절하게 수행합니다

확장 메소드를 사용하는 데는 두 가지 이유가 있습니다.
1. Framework 4.5에서 거의 모든 I / O 바인딩 함수는 Task <T>를 반환합니다.
2. Task <T>가 잘 맞는 경우 형식 시스템에서 단일 값 결과를 전달하기 때문에 IObservable <T>을 통해 사용하는 것이 좋습니다. 즉, 미래에 단일 값을 반환하는 함수는 IObservable <T>가 아닌 Task <T>를 반환해야합니다. 그런 다음 다른 관찰 가능 항목과 결합해야하는 경우 ToObservable ()을 사용하십시오.

확장 메서드의 사용법도 간단합니다.
``` csharp
var t = Task.Factory.StartNew(()=>"Test");
var source = t.ToObservable();
source.Subscribe(
  Console.WriteLine,
  () => Console.WriteLine("completed"));
/* output
Test
completed
*/
```
Task(non-generic)를 IObservable <Unit>으로 변환하는 오버로드도 있습니다.

### From IEnumerable<T>
ToObservable의 마지막 오버로드는 IEnumerable <T>을 사용합니다. 이는 의미 상으로 Observable의 helper 메서드와 비슷합니다. foreach 루프를 사용하여 만듭니다.
``` csharp
//Example code only
public static IObservable<T> ToObservable<T>(this IEnumerable<T> source)
{
  return Observable.Create<T>(o =>
  {
    foreach (var item in source)
    {
     o.OnNext(item);
    }
  //Incorrect disposal pattern
  return Disposable.Empty;
  });
}
```
그러나이 조잡한 구현은 순진합니다. 올바른 처리를 허용하지 않으며, 예외를 올바르게 처리하지 못하며, 나중에 책에서 보게 될 것처럼 매우 훌륭한 동시성 모델을 가지고 있지 않습니다. 물론 Rx의 버전은 이러한 까다로운 세부 사항을 모두 충족하므로 걱정할 필요가 없습니다.

IEnumerable <T>에서 IObservable <T>로 전환 할 때 실제로 달성하고자하는 것을 신중히 고려해야합니다. 또한 의사 결정의 성능 영향을 신중하게 테스트하고 측정해야합니다. IEnumerable <T>의 동기식 동기 (풀) 특성이 때때로 IObservable <T>의 비동기 (푸시) 특성과 잘 섞이지 않는 경우를 고려하십시오. IEnumerable, IEnumerable <T>, 배열 또는 컬렉션을 관찰 가능한 시퀀스의 데이터 형식으로 전달하는 것이 완전히 유효하다는 것을 기억하십시오. 시퀀스를 한꺼번에 구체화 할 수 있다면 IEnumerable로 노출시키지 않아도됩니다. 이것이 적합하다고 생각되면 배열이나 ReadOnlyCollection <T>과 같은 변경 불가능한 타입을 전달하는 것도 고려해보십시오. 데이터 일괄 처리를 제공하는 연산자에 대해서는 IObservable <IList <T >>를 나중에 사용하는 것을 볼 수 있습니다.

### From APM
마지막으로 비동기 프로그래밍 모델 (APM)에서 관찰 가능한 시퀀스로 이동하는 오버로드 세트를 살펴 봅니다. 이것은 Begin ... 및 End ...와 접두어가 붙은 두 개의 메서드와 아이코닉 IAsyncResult 매개 변수 유형을 사용하여 식별 할 수있는 .NET에서 찾을 수있는 프로그래밍 스타일입니다. 이것은 일반적으로 I / O API에서 볼 수 있습니다.

``` csharp
class WebRequest
{    
  public WebResponse GetResponse() 
  {...}
  public IAsyncResult BeginGetResponse(
    AsyncCallback callback, 
    object state) 
  {...}
  public WebResponse EndGetResponse(IAsyncResult asyncResult) 
  {...}
  ...
  }
  class Stream
  {
    public int Read(
      byte[] buffer, 
      int offset, 
      int count) 
    {...}
  public IAsyncResult BeginRead(
    byte[] buffer, 
    int offset, 
    int count, 
    AsyncCallback callback, 
    object state) 
  {...}
  public int EndRead(IAsyncResult asyncResult) 
  {...}
  ...
}
APM 또는 비동기 패턴은 닷NET 프로그램이 장기 실행 IO 경계 작업을 수행하는 데있어 매우 강력하면서도 어색한 방식을 가능하게합니다. IO에 대한 동기식 액세스를 사용한다면 WebRequest.GetResponse () 또는 Stream.Read (...)를 사용하면 스레드를 차단하지만 IO를 기다리는 동안 작업을 수행하지 않습니다. I / O가 완료 될 때까지 대기하는 동안 스레드를 유휴 상태로 유지하기 위해 많은 동시 작업을 수행하는 바쁜 서버에서 이는 매우 낭비 일 수 있습니다. 구현에 따라 APM은 하드웨어 장치 드라이버 계층에서 작동 할 수 있으며 차단하는 동안 스레드가 필요하지 않습니다. APM 모델을 따르는 방법에 대한 정보는 거의 없습니다. 그러나 APM에 대한 자세한 내용은 Jeffery Richter의 뛰어난 책 CLR for C # 또는 Joe Duffy의 포괄적 인 Windows 동시 프로그래밍을 참조하십시오. 인터넷상의 대부분의 것들은 그의 책에서 리히터의 예를 뻔뻔스럽게 묘사 한 것이다. APM에 대한 심층적 인 검토는이 책의 범위에서 제외됩니다.

비동기 프로그래밍 모델을 사용하면서도 어색한 API는 피하려면 Observable.FromAsyncPattern 메서드를 사용할 수 있습니다. 제프리 반 고흐 (Jeffery van Gogh)는 Observable.PromonsyncPattern을 서버 블로그의 Rx 시리즈 1 부에서 훌륭하게 살펴 봅니다. 서버 시리즈에서 Rx를 뒷받침하는 이론이 건전하지만 2010 년 중반에 작성되었으며 Rx의 이전 버전을 대상으로합니다.

Observable의 30 오버로드가 있습니다 .FromAsyncPattern에서는 일반적인 개념을 살펴보고 적절한 과부하를 직접 선택할 수 있습니다. 먼저 APM의 일반적인 패턴을 살펴보면 BeginXXX 메서드는 0 개 이상의 데이터 인수와 AsyncCallback 및 Object를 취할 것입니다. BeginXXX 메서드는 IAsyncResult 토큰을 반환합니다.
``` csharp
//Standard Begin signature
IAsyncResult BeginXXX(AsyncCallback callback, Object state);
//Standard Begin signature with data
IAsyncResult BeginYYY(string someParam1, AsyncCallback callback, object state);
```

EndXXX 메서드는 BeginXXX 메서드에서 반환 된 토큰이어야하는 IAsyncResult를 허용합니다. 또한 EndXXX는 값을 반환 할 수 있습니다.
``` csharp
//Standard EndXXX Signature
void EndXXX(IAsyncResult asyncResult);
//Standard EndXXX Signature with data
int EndYYY(IAsyncResult asyncResult);
```

FromAsyncPattern 메서드의 일반적인 인수는 BeginXXX 데이터 인수 (있는 경우)와 EndXXX 반환 형식 (있는 경우)입니다. 우리가 Stream.Read (byte [], int, int, AsyncResult, object) 예제에 적용하면 BeginRead 메소드의 데이터 매개 변수로 byte [], int 및 또 다른 int가 있음을 알 수 있습니다.
``` csharp
//IAsyncResult BeginRead(
//  byte[] buffer, 
//  int offset, 
//  int count, 
//  AsyncCallback callback, object state) {...}
Observable.FromAsyncPattern<byte[], int, int ...
```

이제 EndXXX 메서드를 살펴보고 FromAsyncPattern 호출의 일반 서명을 완료하는 int를 반환합니다.
``` csharp
//int EndRead(
//  IAsyncResult asyncResult) {...}
Observable.FromAsyncPattern<byte[], int, int, int>
```

Observable.FromAsyncPattern에 대한 호출의 결과는 관찰 가능한 시퀀스를 반환하지 않습니다. 관찰 가능한 시퀀스를 반환하는 대리자를 반환합니다. 이 대리자의 서명은 반환 형식이 관찰 가능한 시퀀스로 래핑된다는 점을 제외하고는 FromAsyncPattern에 대한 호출의 일반적인 인수와 일치합니다.
``` csharp
var fileLength = (int) stream.Length;
//read is a Func<byte[], int, int, IObservable<int>>
var read = Observable.FromAsyncPattern<byte[], int, int, int>(
  stream.BeginRead, 
  stream.EndRead);
var buffer = new byte[fileLength];
var bytesReadStream = read(buffer, 0, fileLength);
bytesReadStream.Subscribe(
  byteCount =>
  {
    Console.WriteLine("Number of bytes read={0}, buffer should be populated with data now.",
    byteCount);
  });
```

이 구현은 단지 예일뿐입니다. 최신 버전의 Rx에 대해 빌드 된 매우 잘 설계된 구현을 위해서는 http://rxx.codeplex.com에있는 Rxx 프로젝트를 살펴 봐야합니다.

여기에는 쿼리 연산자의 첫 번째 분류 인 관찰 가능한 시퀀스 만들기가 포함됩니다. 우리는 시퀀스를 생성하는 다양한 열망하고 게으른 방법을 살펴 보았습니다. corecursion의 개념을 소개하고 Generate 메소드를 사용하여 잠재적 인 무한 시퀀스를 펼칠 수있는 방법을 보여줍니다. 이제 다양한 팩토리 메소드를 사용하여 타이머 기반 시퀀스를 생성 할 수 있습니다. 우리는 또한 다른 동기식 및 비동기식 패러다임에서 전환 할 수있는 방법에 익숙해야하며 그렇게 할 수있는시기와시기를 결정할 수 있어야합니다.

* Factory Methods
  * Observable.Return
  * Observable.Empty
  * Observable.Never
  * Observable.Throw
  * Observable.Create
* Unfold methods
  * Observable.Range
  * Observable.Interval
  * Observable.Timer
  * Observable.Generate
* Paradigm Transition
  * Observable.Start
  * Observable.FromEventPattern
  * Task.ToObservable
  * Task<T>.ToObservable
  * IEnumerable<T>.ToObservable
  * Observable.FromAsyncPattern

관찰 가능한 시퀀스를 생성하는 것은 Rx의 실제 적용에 대한 우리의 첫 번째 단계입니다 : 시퀀스를 생성 한 다음 소비를 위해 노출시킵니다. 이제 관찰 가능한 시퀀스를 만드는 방법에 대해 확고하게 파악 했으므로 관찰 가능한 시퀀스를 쿼리 할 수있는 연산자를 발견 할 수 있습니다.