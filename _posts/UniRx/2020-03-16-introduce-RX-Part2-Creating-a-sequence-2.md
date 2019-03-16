---
layout: single
title: "PART 2 - Sequence basics : Creating a sequence - 2. Functional unfolds"
related: false
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

