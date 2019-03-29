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