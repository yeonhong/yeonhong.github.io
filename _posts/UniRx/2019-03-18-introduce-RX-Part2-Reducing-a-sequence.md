---
layout: single
title: "PART 2 - Sequence basics : Reducing a sequence"
related: false
categories: 
  - 프로그래밍
tags:
  - intro to RX
  - Reactive Programming
link: "http://introtorx.com/Content/v1.0.10621.0/05_Filtering.html#Reduction"
---

# Reducing a sequence
우리는 정보화 시대에 살고 있습니다. 놀라운 속도로 데이터가 생성, 저장 및 배포되고 있습니다. 소화 호스에서 직접 마시려고하는 것처럼이 데이터를 소비하는 것이 압도적 일 수 있습니다. 우리는 필요한 데이터를 골라 내고 관련성이없는 데이터를 선택하고 관련성 높은 데이터를 롤업 할 수있는 능력이 필요합니다. 사용자, 고객 및 관리자는 더 높은 성능과 엄격한 마감 시간을 제공하면서 이전보다 많은 데이터로이 작업을 수행해야합니다.

관찰 가능한 시퀀스를 생성하는 방법을 알면, 관찰 가능한 시퀀스를 줄일 수있는 다양한 방법을 살펴 보겠습니다. 시퀀스를 줄이는 연산자를 다음과 같이 분류 할 수 있습니다.

Filter and partition operators
* 같은 수의 요소가있는 시퀀스로 소스 시퀀스 줄이기
Aggregation operators
* 단일 요소를 사용하여 소스 시퀀스를 시퀀스로 줄입니다.
Fold operators
* 소스 시퀀스를 스칼라 값으로 단일 요소로 줄입니다.

스칼라 값으로부터 관찰 가능한 시퀀스를 생성하는 것은 anamorphism으로 정의되거나 'unfold'되는것으로 설명됩니다. 우리는 T에서 I Observable <T>의 anamorphism을 'unfold'된것으로 생각할 수 있습니다. 이것은 "entering the monad"이라고도 할 수 있습니다.이 경우 (이 책의 대부분의 경우) 모나드는 IObservable <T>입니다. 우리가 이제부터 살펴보기 시작하는 것은 궁극적으로 catamorphism 또는 fold로 정의 된 역행렬을 얻는 방법입니다. 다른 인기있는 접기 이름은 'reduce', 'accumulate'및 'inject'입니다.
> 함수형 프로그래밍의 개념으로 생각됨.

