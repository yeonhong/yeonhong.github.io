---
layout: single
title: "PART 2 - Sequence basics : Aggregation (집합)"
related: true
categories: 
  - 프로그래밍
tags:
  - intro to RX
  - Reactive Programming
link: "http://introtorx.com/Content/v1.0.10621.0/07_Aggregation.html#Aggregation"
---

# Aggregation
데이터는 원시 형식이 항상 귀중한 것은 아닙니다. 때때로 우리는 거대한 데이터를 통합하고, 조합하고, 결합하거나, 응축시켜 더 소비가 가능한 한 입 크기의 덩어리로 만들 필요가 있습니다. 계측, 재무, 신호 처리 및 운영 정보와 같은 영역에서 빠르게 이동하는 데이터를 고려하십시오. 이러한 종류의 데이터는 초당 10 개 이상의 값으로 변할 수 있습니다. 사람이 실제로 이것을 소비 할 수 있습니까? 아마도 사람이 소비하기 때문에 평균, 최소값 및 최대 값과 같은 집계 값을 더 많이 사용할 수 있습니다.

관찰 가능한 시퀀스를 줄이는 것을 계속하면서 Rx에서 사용할 수있는 집계 함수를 살펴 보겠습니다. 우리의 첫 번째 세트는 관찰 가능한 시퀀스를 취하여 하나의 값을 갖는 시퀀스로 줄이기 때문에 마지막 장에서 계속됩니다. 그런 다음 시퀀스를 다시 스칼라 값으로 전환 할 수있는 연산자를 찾습니다.

새로운 연산자를 도입하기 바로 전에 우리는 자체 확장 메서드를 신속하게 만들 것입니다. 샘플을 만드는 데 도움이되는 '덤프'확장 메서드를 사용합니다.
``` csharp
public static class SampleExtentions
{
  public static void Dump<T>(this IObservable<T> source, string name)
  {
    source.Subscribe(
      i=>Console.WriteLine("{0}-->{1}", name, i), 
      ex=>Console.WriteLine("{0} failed-->{1}", name, ex.Message),
      ()=>Console.WriteLine("{0} completed", name));
  }
}
```

## Count
