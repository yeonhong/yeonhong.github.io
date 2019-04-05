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