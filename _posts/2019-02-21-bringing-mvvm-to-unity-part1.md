---
layout: single
title: "bringing-mvvm-to-unity-part1(번역)"
categories: 
  - 프로그래밍
tags:
  - Unity
  - MVVM
  - Design Pattern
link: http://www.what-could-possibly-go-wrong.com/bringing-mvvm-to-unity-part-1-about-mvvm-and-unity-weld/
---
## Bringing MVVM to Unity - part 1: About MVVM and Unity-Weld

Unity-Weld는 UI 계층을 구현하는 코드와 Unity 계층 구조 사이의 접착제입니다. 그것은 당신의 UI의 부분들을 묶어줍니다. 그것들은 그것들을 조정하고 그들이 의사 소통하도록 허락합니다. Unity all-Weld의 장점은 간단합니다. 배우기 쉽고 사용하기 쉽습니다. 그것은 Unity와 잘 작동하고 Unity의 UI 시스템에 맞는 것처럼 느껴집니다.

이 연재 기사에서는 Unity-Weld를 사용하여 Unity UI에서 MVVM을 활용하는 방법을 설명합니다.

1 부에서는 MVVM이 무엇인지 설명하고 왜 사용해야 하는지를 설명하는 몇 가지 배경을 다룹니다.

### MVVM은 무엇인가?

MVVM 또는 ViewModel - View - Model은 깨끗한 조직을 촉진하고 단위 테스트를 수행하는 데 도움이되는 UI를 구성하는 디자인 패턴 및 방법입니다. MVVM은 Unity 애플리케이션의 맥락에서 실제로 우리가 서로 묶어서 일반적으로 계층 전체에 흩어져있는 서로 다른 모든 스크립트를 구성하는 데 도움이된다고 주장합니다. 이것은 구성 요소간에 발생하는 통신을 형식화하고 구조화하는 방법입니다. 그렇지 않으면 무분별하고 일관성없는 방식으로 쉽게 진화 할 수 있습니다.

MVVM은 UI 렌더링(View)과 UI 로직(ViewModel)을 훌륭하게 구분합니다.

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/02/MVVM.png)

UI 로직의 자동화 된 테스팅에 필요한 코드분리를 깔끔하게 해줍니다.

MVVM을 사용하면 단일 뷰 모델 인스턴스에 여러 뷰를 연결하기 만하면 동일한 UI 논리에서 별도의 뷰를 사용할 수 있습니다.

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/02/MVVM_4.png)

MVVM에는 두 가지 주요 측면이 있습니다. 첫 번째는 뷰 모델의 속성과 뷰 사이의 연결을 형성하는 데이터 바인딩입니다. 연결의 양쪽에서의 변경 사항은 자동으로 다른쪽에 전파됩니다.

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/02/MVVM_2.png)

MVVM의 또 다른 측면은 이벤트 기반 프로그래밍으로, View는 ViewModel에 의해 처리되는 이벤트 (사용자 작업에 의해 트리거 됨)를 발생시킵니다.

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/02/MVVM_3.png)

MVVM은 일반적으로 Windows Presentation Foundation (또는 WPF)과 관련되어 있으며이를 통해 들어봤을 수도 있습니다. 그러나 디자인 패턴 자체는 프레젠테이션 모델 패턴 및 MVC 패턴으로 되돌아갑니다.

MVC 패턴에는 많은 변형이 있습니다. MVVM의 구현으로 Unity-Weld를 설명하는 것이 맞는지에 대해,
WPF의 근간을 이루는 MVVM 패턴에 영감을 얻고 계속해서 영감을 얻었습니다. 새 패턴은 새로운 상황과 목적을 처리하기 위해 진화하면서 항상 만들어집니다. 필자의 견해로 볼 때, Unity-Weld는 구현 방식이 이전과 다른 점이 있더라도 MVVM의 직접적인 후손이라고 생각합니다.

### 왜 MVVM을 써야하나?

MVVM의 주 목적은 UI 로직에 대한 테스트 주도 개발 및 유닛 테스트를 가능하게하는 것입니다. MVVM의 올바른 구현은 확실히 이것을 가능하게하고 UI 로직에 대한 자동화 된 테스트를 직접 작성한다고해도, 과거에는 불가능하지는 않더라도 어렵다고 여겨졌습니다. 그래서 TDD는 MVVM의 주요 판매 포인트였습니다.

그러나 나는 특히 Unity와 함께 일할 때 다른 이점을 주장 할 것입니다. 그래서 TDD가 당신의 일이 아니라도, MVVM을 사용함으로써 실질적인 이익을 얻을 수 있다고 생각합니다. 처음에는 응용 프로그램 디자인에 긍정적 인 변화가있었습니다. UI 렌더링 / 컨트롤과 UI 로직을 깨끗하게 분리하여 우아한 구조를 만듭니다.

일반적으로 Unity 애플리케이션은 여러 계층의 스크립트로 구성되어 있으며 수많은 스크립트가 첨부되어 있습니다. 이 스크립트는 기능적 UI를 형성하기 위해 서로 통신하고 협력해야합니다. MVVM을 사용하면 특정 스크립트 모음을 통합하고 서로 묶어 응집력있게 통합 할 수 있습니다.

**이미 언급했듯이 MVVM의 유용한 부작용은 단일보기 모델에 여러보기를 연결하고 MVVM을 통해 자동으로 동기화 할 수 있다는 것입니다. 뷰 모델을 업데이트하면 연결된 모든 뷰가 자동으로 업데이트 된 상태로 유지됩니다.**

### MVVM for Unity

Unity-Weld를 사용하면 Unity 계층을 통해 뷰 모델을 뷰에 연결할 수 있습니다. 우리의 뷰 모델은 MonoBehaviours 일 수도 있고 순수한 C# 클래스 일 수도 있습니다.

우리의 견해는 계층에 추가 된 InputField 및 Button과 같은 Unity 구성 요소입니다. 그런 다음 뷰와 뷰 모델 간의 연결을 처리하는 OneWayPropertyBinding, TwoWayPropertyBinding 및 EventBinding과 같은 추가 구성 요소를 추가해야합니다. 

Unity-Weld는 뷰 모델과 뷰 사이의 동기화를 자동으로 유지합니다. 특성 간의 접착제는 일반적으로 데이터 바인딩으로 알려져 있습니다. 우리는 UI 이벤트와 뷰 모델 함수 사이의 연결에 이벤트 바인딩이라는 이름을 부여했습니다.

Unity를 사용하여 응용 프로그램을 작성할 때 우리는 응용 프로그램을 구조화 할 수있는 하나의 주요 메커니즘을 가지고 있습니다. Unity 구성 요소 시스템 패턴의 구현입니다. 그것이 유용한 패턴이지만 우리는 복잡한 애플리케이션을 구조화하기 위해 더 많은 것을 필요로합니다. 

이전에 의존성 주입에 대해 논의했는데 이 퍼즐의 한 부분입니다. MVVM은 또 다른 부분입니다. 이 패턴들은 함께 게임을 구성하는 스크립트와 게임 객체의 엉망을 이해하고 연관시키는 데 도움이됩니다. **종속성 삽입 및 MVVM은 다른 모든 기능을 중단 할 수있는 응용 프로그램의 구조적 토대를 만듭니다.**
