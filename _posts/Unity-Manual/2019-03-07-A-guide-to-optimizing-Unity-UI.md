---
layout: single
title: "A guide to optimizing Unity UI"
categories: 
  - 프로그래밍
tags:
  - Unity
  - Optimaze
  - UI
link: https://unity3d.com/learn/tutorials/topics/best-practices/guide-optimizing-unity-ui?playlist=30089
---
Unity UI로 구동되는 사용자 인터페이스를 최적화하는 것은 하나의 기술입니다. 엄격하고 엄격한 규칙은 거의 없습니다. 대신 시스템의 동작을 염두에두고 각 상황을 신중하게 평가해야합니다. Unity UI를 최적화 할 때 핵심적인 것은 그리기 호출과 배치 비용의 균형을 맞추는 것입니다. 일부 상식적인 기술을 사용하여 하나 또는 다른 것을 줄일 수 있지만 복잡한 UI는 절충점을 만들어야합니다.

그러나 다른 곳에서는 모범 사례처럼 유니티 UI를 최적화하려는 시도는 프로파일 링으로 시작해야합니다. Unity UI 시스템을 최적화하기 전에 가장 중요한 작업은 관찰 된 성능 문제의 정확한 이유를 찾는 것입니다. Unity UI 사용자가 겪는 네 가지 공통적 인 문제는 다음과 같습니다.

* 과도한 GPU 프래그먼트 셰이더 활용 (즉, fill-rate 과다 사용)
* 캔버스 배치를 재 작성하는 데 소요되는 과도한 CPU 시간
* Canvas 배치의 과도한 재 구축 (지나치게 지저분한)
* 버텍스 생성에 소모되는 CPU 과도한 시간 (일반적으로 텍스트)

원칙적으로 성능이 GPU로 전송되는 무수한 그리기 호출에 의해 제약되는 Unity UI를 만드는 것이 가능합니다. 그러나 실제로는 그리기 호출로 GPU에 과부하가 걸리는 프로젝트는 채우기 비율 과다 사용으로 인해 발생할 가능성이 큽니다.

이 가이드에서는 Unity UI를 구성하는 기본 개념, 알고리즘 및 코드는 물론 일반적인 문제 및 솔루션에 대해 논의합니다. 그것은 5 개의 장으로 나뉩니다.
1. 유니티 UI의 기본 장에서는 유니티 UI에 관련된 용어를 정의하고 일괄 형상 작성과 같은 UI 렌더링을 위해 수행되는 많은 기본 프로세스의 세부 사항에 대해 설명합니다. 독자가이 장부터 시작하도록 강력히 권장합니다.
2. Unity UI 프로파일 링 도구 장에서는 개발자가 사용할 수있는 다양한 도구를 사용하여 프로파일 링 데이터를 수집하는 방법에 대해 설명합니다.
3. fill-rate, 캔버스 및 입력 장에서는 Unity UI의 캔버스 및 입력 구성 요소의 성능을 향상시키는 방법에 대해 설명합니다.
4. UI 컨트롤 장에서는 UI 텍스트, 스크롤 뷰 및 기타 구성 요소 별 최적화와 다른 곳에서 잘 맞지 않는 몇 가지 기술에 대해 설명합니다.
5. 기타 기술 및 팁 장에서는 UI 시스템의 "gotchas"에 대한 몇 가지 기본 팁과 해결 방법을 포함하여 다른 곳에 적합하지 않은 몇 가지 문제에 대해 설명합니다.

# UI Source Code
Unity UI의 그래픽 및 레이아웃 구성 요소는 전적으로 오픈 소스임을 기억하십시오. 그들의 소스 코드는 [Unity의 Bitbucket 저장소에서 UI](https://bitbucket.org/Unity-Technologies/ui/)로 찾을 수 있습니다.