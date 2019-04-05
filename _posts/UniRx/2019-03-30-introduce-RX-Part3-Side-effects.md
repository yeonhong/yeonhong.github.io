---
layout: single
title: "PART 3 - Taming the sequence : Side effects"
related: true
classes: wide
categories: 
  - 프로그래밍
tags:
  - intro to RX
  - Reactive Programming
link: "http://introtorx.com/Content/v1.0.10621.0/09_SideEffects.html"
---

# PART 3 - Taming the sequence

이 책의 세 번째 파트에서는 Rx를 샘플 코드 이상에 적용 할 수있는 기능을 살펴볼 것입니다. 생산 품질 코드를 작성할 때 종종 오류 시나리오를 처리하고 워크 플로를 기록하고 특정 상황에서 다시 시도하고 사례 및 데모에서 정기적으로 제외되는 리소스 및 기타 실제 문제를 처리 할 수 있어야합니다.

이 책의 제 3 부는 Rx를 단순한 장난감 이상으로 사용하는 데 필요한 도구를 제공하는 것을 목표로합니다. Rx를 제대로 사용하면 코드 기반에서 Rx가 보급 될 것입니다. IEnumerable 형식을 사용하여 foreach 구문을 사용하지 않거나 IDisposable 형식을 사용하여 구문을 사용하는 것을 부끄럽게 생각하지 않아도되는 것처럼 위장 해서는 안됩니다. Rx를 이해하고 수용하면 경쟁 조건을 식별하고 제거하여 코드를 더 선언적으로 만들 수 있으므로 코드베이스를 줄이고 유지 보수가 쉬워집니다.

Rx 코드의 유지 관리에는 분명히 Rx 지식이 필요하지만 "닭이 먼저인가 계란이 먼저인가"문제가 발생합니다. 나는 Rx가 여기 머무르고 있다고 믿기로 선택한다. 나는 그것이 목표의 문제를 아주 잘 해결하기 때문에 이것을 믿는다. 또한 TPL (Task Parallel Library) 및 .NET 4.5의 향후 비동기 / 대기 기능과 같은 다른 라이브러리 및 기능에 무료로 제공됩니다. 이것을 고려하면, Rx가 코드베이스를 개선한다면 우리는 이를 받아 들여야합니다!

## Side effects

생산 시스템의 비 기능 요구 사항은 종종 고 가용성, 품질 모니터링 기능 및 결함 해결을위한 리드 타임을 요구합니다. 로깅, 디버깅, 계측 및 저널링은 개발자가 생산 준비 시스템을 고려해야 할 일반적인 비 기능 요구 사항입니다. 이러한 아티팩트는 주요 비즈니스 워크 플로우의 부작용으로 간주 될 수 있습니다. 부작용은 코드 샘플 및 사용 방법 안내가 종종 무시하는 실제 문제이지만 Rx는 도움이되는 도구를 제공합니다.

이 장에서는 observable 시퀀스로 작업 할 때 부작용을 가져 오는 결과에 대해 설명합니다. 함수는 반환 값 이외에 관찰 가능한 다른 효과가있는 경우 부작용이있는 것으로 간주됩니다. 일반적으로 '관찰 가능한 효과'는 state를 수정 한 것입니다. 이 관찰 가능한 효과는

* 함수 (즉, 전역 변수, 정적 변수 또는 인수 변수)보다 넓은 범위의 변수 수정 (함수밖에 있는 변수의 수정)
* 파일 또는 네트워크에서의 읽기 / 쓰기와 같은 I / O
* 디스플레이 업데이트

### Issues with side effects

함수형 프로그래밍은 일반적으로 부작용을 피하려고합니다. 부작용이있는 함수, 특히 상태를 수정하는 함수는 프로그래머가 함수의 입력과 출력 이상의 것을 이해해야합니다. 표면적인 상태의 히스토리와 맥락으로 확대하여 이해 할 필요가 있다. 이렇게하면 함수의 복잡성이 크게 증가 할 수 있으므로 정확하게 이해하고 유지 관리하기가 더 어려워집니다.

부작용이 항상 우연한 것은 아니며 항상 부작용이 아닙니다. 우연한 부작용을 줄이는 쉬운 방법은 변화를위한 표면적을 줄이는 것입니다. 코더가 취할 수있는 간단한 행동은 국가의 가시성이나 범위를 줄이고 불변의 것을 만들 수있게하는 것입니다. 변수와 같은 코드 블록으로 범위를 지정하여 변수의 가시성을 줄일 수 있습니다. 클래스 멤버를 비공개 또는 보호함으로써 클래스 멤버의 가시성을 줄일 수 있습니다. 정의에 따라 불변의 데이터는 수정할 수 없으므로 부작용을 나타낼 수 없습니다. 이는 Rx 코드의 유지 보수성을 획기적으로 향상시키는 현명한 캡슐화 규칙입니다.

부작용이있는 쿼리의 간단한 예제를 제공하기 위해 변수 (클로저)를 업데이트하여받은 요소의 인덱스와 값을 출력하려고 시도합니다.

```csharp
var letters = Observable.Range(0, 3)
  .Select(i => (char)(i + 65));
var index = -1;
var result = letters.Select(
  c =>
  {
    index++;
    return c;
  });
result.Subscribe(
  c => Console.WriteLine("Received {0} at index {1}", c, index),
  () => Console.WriteLine("completed"));

/* output
Received A at index 0
Received B at index 1
Received C at index 2
completed
*/
```

이것은 충분히 해가 없지만 다른 사람이이 코드를보고 팀이 사용하는 패턴으로 이해하는 경우를 상상해보십시오. 그들은 차례로이 스타일을 스스로 채택합니다. 예를 들어 이전 예제에 중복 구독을 추가합니다.

``` csharp
var letters = Observable.Range(0, 3)
  .Select(i => (char)(i + 65));
var index = -1;
var result = letters.Select(
  c =>
  {
    index++;
    return c;
  });
result.Subscribe(
  c => Console.WriteLine("Received {0} at index {1}", c, index),
  () => Console.WriteLine("completed"));
result.Subscribe(
  c => Console.WriteLine("Also received {0} at index {1}", c, index),
  () => Console.WriteLine("2nd completed"));
/*
Output

Received A at index 0
Received B at index 1
Received C at index 2
completed
Also received A at index 3
Also received B at index 4
Also received C at index 5
2nd completed

즉, index변수를 두개의 subscribe가 공유하여 의도하지 않은 결과를 얻게됨.
*/
```

이제 두 번째 사람의 결과는 분명히 난센스입니다. 그들은 인덱스 값이 0, 1 및 2가 될 것으로 기대하지만 대신 3, 4 및 5를 얻습니다. 나는 코드베이스에서 부작용의 훨씬 더 불길한 버전을 보았다. 지저분한 것들은 종종 부울 값인 상태를 수정합니다. hasValues, isStreaming 등이 있습니다. 우리는 이후 장에서 공유 상태를 사용하는 것보다 관찰 가능한 순서로 워크 플로우를 제어하는 훨씬 더 나은 방법을 살펴볼 것입니다.

기존 소프트웨어에 예측할 수없는 결과를 생성하는 것 외에도 부작용이있는 프로그램은 테스트 및 유지 관리가 훨씬 어렵습니다. 부작용을 나타내는 프로그램에 대한 향후 리팩토링, 개선 또는 기타 유지 관리가 취약해질 수 있습니다. 이것은 특히 비동기 또는 동시 소프트웨어에서 그러합니다.

### Composing data in a pipeline

상태를 캡처하는 가장 좋은 방법은이를 파이프 라인에 도입하는 것입니다. 이상적으로, 우리는 파이프 라인의 각 부분을 독립적이고 결정 론적으로 취급하기를 원합니다. 즉, 파이프 라인을 구성하는 각 함수는 유일한 상태로 입력과 출력을 가져야합니다. 예제를 수정하기 위해 파이프 라인의 데이터를 풍부하게하여 공유 상태가 없도록 할 수 있습니다. 인덱스를 노출하는 Select 오버로드를 사용할 수있는 좋은 예가 될 것입니다.

``` csharp
var source = Observable.Range(0, 3);
var result = source.Select(
  (idx, value) => new
  {
    Index = idx,
    Letter = (char) (value + 65)
  });
result.Subscribe(
  x => Console.WriteLine("Received {0} at index {1}", x.Letter, x.Index),
  () => Console.WriteLine("completed"));
result.Subscribe(
  x => Console.WriteLine("Also received {0} at index {1}", x.Letter, x.Index),
  () => Console.WriteLine("2nd completed"));
/*
Output:

Received A at index 0
Received B at index 1
Received C at index 2
completed
Also received A at index 0
Also received B at index 1
Also received C at index 2
2nd completed
*/
```

상자 밖에서 생각해 보면 Scan과 같은 다른 기능을 사용하여 비슷한 결과를 얻을 수도 있습니다. 여기에 예제가있다.

``` csharp
var result = source.Scan(
  new
  {
    Index = -1,
    Letter = new char()
  },
  (acc, value) => new
  {
    Index = acc.Index + 1,
    Letter = (char)(value + 65)
  });
```

여기서 핵심은 상태를 분리하고 상태를 변경하는 것과 같은 부작용을 줄이거나 제거하는 것입니다.

### Do

부작용을 피하기 위해 노력해야하지만 경우에 따라 피할 수없는 경우도 있습니다. Do 확장 방법을 사용하면 부작용을 주입 할 수 있습니다. Do 확장 메서드의 서명은 Select 메서드와 매우 비슷합니다.

* OnNext, OnError 및 OnCompleted 처리기의 조합을 처리하기 위해 다양한 오버로드가 있습니다.
* 그들은 모두 돌아가서 관찰 가능한 순서를 취한다.

``` csharp
// Invokes an action with side effecting behavior for each element in the observable 
//  sequence.
public static IObservable<TSource> Do<TSource>(
  this IObservable<TSource> source, 
  Action<TSource> onNext)
{...}
// Invokes an action with side effecting behavior for each element in the observable 
//  sequence and invokes an action with side effecting behavior upon graceful termination
//  of the observable sequence.
public static IObservable<TSource> Do<TSource>(
  this IObservable<TSource> source, 
  Action<TSource> onNext, 
  Action onCompleted)
{...}
// Invokes an action with side effecting behavior for each element in the observable
//  sequence and invokes an action with side effecting behavior upon exceptional 
//  termination of the observable sequence.
public static IObservable<TSource> Do<TSource>(
  this IObservable<TSource> source, 
  Action<TSource> onNext, 
  Action<Exception> onError)
{...}
// Invokes an action with side effecting behavior for each element in the observable
//  sequence and invokes an action with side effecting behavior upon graceful or
//  exceptional termination of the observable sequence.
public static IObservable<TSource> Do<TSource>(
  this IObservable<TSource> source, 
  Action<TSource> onNext, 
  Action<Exception> onError, 
  Action onCompleted)
{...}
// Invokes the observer's methods for their side effects.
public static IObservable<TSource> Do<TSource>(
  this IObservable<TSource> source, 
  IObserver<TSource> observer)
{...}
```

Select overload는 OnNext 처리기에 대한 Func 인수를 취하고 소스와 다른 유형 인 관찰 가능한 시퀀스를 반환하는 기능을 제공합니다. 반대로 **Do 메서드는 OnNext 처리기에 대한 작업 <T> 만 사용하므로 원본과 동일한 형식의 시퀀스 만 반환 할 수 있습니다.** Do를 overload로 전달할 수 있는 각 인수는 action이므로 암시적으로 부작용을 일으 킵니다.

다음 예제에서는 먼저 로깅을위한 다음 메소드를 정의합니다.

``` csharp
private static void Log(object onNextValue)
{
  Console.WriteLine("Logging OnNext({0}) @ {1}", onNextValue, DateTime.Now);
}
private static void Log(Exception onErrorValue)
{
  Console.WriteLine("Logging OnError({0}) @ {1}", onErrorValue, DateTime.Now);
}
private static void Log()
{
  Console.WriteLine("Logging OnCompleted()@ {0}", DateTime.Now);
}
```

이 코드는 위의 메소드를 사용하여 일부 로깅을 소개하기 위해 Do를 사용할 수 있습니다.

``` csharp
var source = Observable
  .Interval(TimeSpan.FromSeconds(1))
  .Take(3);
var result = source.Do(
  i => Log(i),
  ex => Log(ex),
  () => Log());

result.Subscribe(
  Console.WriteLine,
  () => Console.WriteLine("completed"));
/*
Output:

Logging OnNext(0) @ 01/01/2012 12:00:00
0
Logging OnNext(1) @ 01/01/2012 12:00:01
1
Logging OnNext(2) @ 01/01/2012 12:00:02
2
Logging OnCompleted() @ 01/01/2012 12:00:02
completed
*/
```

Do는 쿼리 체인의 구독보다 먼저 있기 때문에 먼저 값을 받으면 먼저 콘솔에 씁니다. Do 메소드를 시퀀스의 와이어 탭으로 생각하고 싶습니다. 시퀀스를 수 정할 수있는 기능이 없기 때문에 시퀀스를 청취 할 수 있습니다.

Rx에서 볼 수있는 가장 일반적인 허용 가능한 부작용은 기록 할 필요가 있다는 것입니다. Do의 서명을 통해 쿼리 체인에 삽입 할 수 있습니다. 이를 통해 시퀀스에 로깅을 추가하고 캡슐화를 유지할 수 있습니다. 저장소, 서비스 에이전트 또는 제공자가 관찰 가능한 시퀀스를 노출하면 공개적으로 공개하기 전에 시퀀스에 부작용 (예 : 로깅)을 추가 할 수 있습니다. 그러면 소비자는 쿼리에 연산자를 추가 할 수 있습니다 (예 : Where, SelectMany). 이로 인해 공급자 로깅에 영향을주지 않습니다.

아래의 방법을 고려하십시오. 그것은 숫자를 생산하지만 그것이 생산하는 것을 기록합니다 (간결함을 위해 콘솔에). 소비 코드에는 로깅이 투명합니다.

``` csharp
private static IObservable<long> GetNumbers()
{
  return Observable.Interval(TimeSpan.FromMilliseconds(250))
  .Do(i => Console.WriteLine("pushing {0} from GetNumbers", i));
}
```

그런 다음이 코드로 호출합니다.

``` csharp
var source = GetNumbers();
var result = source.Where(i => i%3 == 0)
  .Take(3)
  .Select(i => (char) (i + 65));
result.Subscribe(
  Console.WriteLine,
  () => Console.WriteLine("completed"));
/*
Output:

pushing 0 from GetNumbers
A
pushing 1 from GetNumbers
pushing 2 from GetNumbers
pushing 3 from GetNumbers
D
pushing 4 from GetNumbers
pushing 5 from GetNumbers
pushing 6 from GetNumbers
G
completed
*/
```

이 예는 생산자 또는 중개자가 최종 소비자가 수행하는 것과 관계없이 시퀀스에 로깅을 적용하는 방법을 보여줍니다.

Do overload는 IObserver <T>를 전달할 수있게합니다. 이 오버로드에서 OnNext, OnError 및 OnCompleted 메서드 각각은 수행 할 각 작업으로 다른 Do overload로 전달됩니다.

부작용을 적용하면 쿼리가 복잡해집니다. 부작용이 필요한 악이라면 명백한 것은 동료 코더가 의도를 이해하는 데 도움이됩니다. Do 메서드를 사용하는 것은 선호하는 방법입니다. 이것은 사소한 것처럼 보일 수 있지만 비동기 및 동시성이 혼합 된 비즈니스 도메인의 고유 한 복잡성을 감안할 때 개발자는 Subscribe 또는 Select 연산자에 숨겨진 부작용의 추가 된 복잡성을 필요로하지 않습니다.

### Encapsulating with AsObservable

불충분 한 캡슐화는 의도하지 않은 부작용으로 개발자가 문을 열어 둘 수있는 방법입니다. 부주의로 인해 새는 추상화가 발생하는 몇 가지 시나리오가 있습니다. 첫 번째 예는 한 눈에 무해한 것처럼 보일 수 있지만 여러 가지 문제가 있습니다.

``` csharp
public class UltraLeakyLetterRepo
{
  public ReplaySubject<string> Letters { get; set; }
  public UltraLeakyLetterRepo()
  {
    Letters = new ReplaySubject<string>();
    Letters.OnNext("A");
    Letters.OnNext("B");
    Letters.OnNext("C");
  }
}
```

이 예제에서는 관찰 가능한 시퀀스를 속성으로 표시합니다. 첫 번째 문제는 설정 가능한 속성이라는 점입니다. 원하는 경우 소비자는 전체 주제를 변경할 수 있습니다. 이것은 이 class의 다른 소비자들에게는 매우 좋지 않은 경험이 될 것입니다. 간단한 수정을하면 충분히 안전하다고 생각되는 class를 만들 수 있습니다.

``` csharp
public class LeakyLetterRepo
{
  private readonly ReplaySubject<string> _letters;
  
  public LeakyLetterRepo()
  {
    _letters = new ReplaySubject<string>();
    _letters.OnNext("A");
    _letters.OnNext("B");
    _letters.OnNext("C");
  }

  public ReplaySubject<string> Letters
  {
    get { return _letters; }
  }
}
```

이제 Letters 속성에는 getter 만 있고 읽기 전용 필드가 지원됩니다. 이것은 훨씬 낫다. Keen 독자는 Letters 속성이 ReplaySubject <string>을 반환한다는 것을 알 수 있습니다. OnNext / OnError / OnCompleted를 호출 할 수 있기 때문에 캡슐화가 잘 이루어지지 않습니다. 그 허점을 없애기 위해서 우리는 단순히 리턴 타입을 IObservable <string>으로 만들 수 있습니다.

``` csharp
public IObservable<string> Letters
{
  get { return _letters; }
}
```

class은 이제 훨씬 나아 보인다. 그러나 개선은 단지 겉치례에 불과합니다. 소비자가 결과를 ISubject <string>로 캐스팅하지 못하게하고 원하는 메서드를 호출 할 수있는 방법은 없습니다. 이 예제에서는 외부 코드가 값을 시퀀스로 밀어 넣는 것을 볼 수 있습니다.

``` csharp
var repo = new ObscuredLeakinessLetterRepo();
var good = repo.GetLetters();
good.Subscribe(Console.WriteLine);

//Be naughty
var evil = repo.GetLetters();
var asSubject = evil as ISubject<string>;
if (asSubject != null)
{
  //So naughty, 1 is not a letter!
  asSubject.OnNext("1");
}
else
{
  Console.WriteLine("could not sabotage");
}
/*
Output:

A
B
C
1
*/
```

이 문제의 수정은 매우 간단합니다. AsObservable 확장 메서드를 적용하면 _letters 필드가 IObservable <T> 만 구현하는 형식으로 래핑됩니다.

``` csharp
public IObservable<string> GetLetters()
{
  return _letters.AsObservable();
}
/*
Output:

A
B
C
could not sabotage
*/
```

이 예에서는 '악'과 '방해'와 같은 단어를 사용했지만, 문제를 유발하는 악의적 인 의도보다는 오히려 oversight쪽이 더 자주 발생합니다. 실패한 것은 먼저 누출 된 클래스를 설계 한 프로그래머에게 돌아갑니다. 인터페이스 설계는 어렵지만, 우리는 코드의 소비자가 발견 가능하고 일관성있는 유형을 제공함으로써 성공의 구덩이에 빠지도록 최선을 다해야합니다. 우리가 소비자에게 사용하려는 기능 만 노출하도록 표면적을 줄이면 유형을 더 쉽게 발견 할 수 있습니다. 이 예에서는 유형의 표면적을 줄였습니다. 우리는 프로퍼티를 제거하고 AsObservable 메서드를 통해보다 간단한 형식을 반환했습니다.

### Mutable elements cannot be protected

AsObservable 메서드는 시퀀스를 캡슐화 할 수 있지만 변경 가능한 요소에 대한 보호 기능을 제공하지 않는다는 점을 알고 있어야합니다. 이 클래스의 시퀀스의 소비자가 할 수있는 일을 고려하십시오.

``` csharp
public class Account
{
  public int Id { get; set; }
  public string Name { get; set; }
}
```

시퀀스의 요소를 수정하기로 결정할 때 우리가 할 수있는 혼란의 빠른 예가 있습니다.

``` csharp
var source = new Subject<Account>();
//Evil code. It modifies the Account object.
source.Subscribe(account => account.Name = "Garbage");
//unassuming well behaved code
source.Subscribe(
  account=>Console.WriteLine("{0} {1}", account.Id, account.Name),
  ()=>Console.WriteLine("completed"));

source.OnNext(new Account {Id = 1, Name = "Microsoft"});
source.OnNext(new Account {Id = 2, Name = "Google"});
source.OnNext(new Account {Id = 3, Name = "IBM"});
source.OnCompleted();
/*
Output:

1 Garbage
2 Garbage
3 Garbage
completed
*/
```

두 번째 소비자는 '마이크로 소프트', '구글', 'IBM'을 얻으려고했지만 '쓰레기'만 받았다.

관찰 가능한 순서는 일련의 해결 된 사건으로 인식 될 것입니다 : 사실은 statement로서 일어났습니다. 이는 두 가지를 의미합니다. 첫째, 각 요소는 게시시점의 상태 스냅 샷을 나타내며 두 번째로 정보는 신뢰할 수 있는 출처에서 나옵니다. 우리는 위조의 가능성을 제거하고자합니다. 이상적으로 T 타입은 불변 일 것이므로 두 문제를 모두 해결할 수 있습니다. 이 방법으로 시퀀스의 소비자는 그들이 얻은 데이터가 소스가 생성 한 데이터라는 것을 확신 할 수 있습니다. 요소를 변이시킬 수 없다는 것은 소비자로서의 한계로 보일 수 있지만 이러한 요구는 더 나은 캡슐화를 제공하는 변환 연산자를 통해 가장 잘 충족됩니다.

가능한 경우 부작용을 피해야합니다. 동시성과 공유 상태의 모든 조합은 일반적으로 복잡한 잠금, CPU 아키텍처에 대한 깊은 이해, 사용하는 언어의 잠금 및 최적화 기능을 사용하는 방법에 대한 필요성을 요구합니다. 간단하고 선호되는 방법은 공유 상태를 피하고 불변의 데이터 유형을 선호하며 쿼리 작성 및 변환을 이용하는 것입니다. Where 나 Select 절에 부작용을 숨기면 매우 혼란스러운 코드를 만들 수 있습니다. 부작용이 필요한 경우 Do 메서드는 명시 적으로 부작용을 작성한다는 의도를 나타냅니다.
