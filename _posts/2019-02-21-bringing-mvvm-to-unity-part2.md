---
layout: single
title: "bringing-mvvm-to-unity-part2(번역)"
categories: 
  - 프로그래밍
tags:
  - Unity
  - MVVM
  - Design Pattern
link: http://www.what-could-possibly-go-wrong.com/bringing-mvvm-to-unity-part-2-property-and-event-bindings/
---

## Bringing MVVM to Unity - Part 2: Property and event bindings

Part 2에서는 유니티 - 웰드 (Unity-Weld)의 기본을 보여주는 예제를 제공합니다. ViewModel을 Unity UI에 바인딩하는 것이 가장 핵심적인 측면입니다.

### UI Setup

다음 예제에서는 Unity-Weld의 가장 기본적인 측면을 보여줍니다 : 뷰 모델을 뷰에 바인딩하는 방법. OneWayPropertyBinding 및 TwoWayPropertyBinding을 사용하여 UI 속성을 뷰 모델에 바인딩하는 방법을 배우게됩니다. 그런 다음 EventBinding이 UI 이벤트를 뷰 모델 기능에 바인딩하는 방법을 살펴 보겠습니다. Unity-Weld에는 훨씬 더 많은 것이 있습니다 ... 그러나 이것들은 당신이 그것을 사용하기 시작할 때 필요한 기초입니다.

Unity-Weld를 사용하여 UI를 만드는 단계 :
1. Unity UI를 만듭니다. 텍스트, 입력 필드, 버튼 등 모든 일반적인 것들이 필요합니다.
2. 뷰 모델 클래스를 작성하십시오. 이는 사용자 지정 MonoBehaviour처럼 간단 할 수 있습니다.
3. 뷰 모델 클래스와 UI 요소에 바인딩 할 속성 및 메서드에 Binding 특성을 추가합니다.
4. 뷰 모델을 UI 계층에 연결하십시오.
5. UI 요소에 바인딩 구성 요소 (예 : OneWayPropertyBinding, TwoWayPropertyBinding 및 EventBinding)를 첨부하여 뷰 모델에 연결합니다

다음 예제에서는 Unity-Weld로 간단한 UI를 만드는 구체적인 예를 보여줍니다.

### Example

[예제 프로젝트](https://github.com/Real-Serious-Games/Unity-Weld-Examples)

#### 1. "One way property" binding to a text field

Unity-Weld의 첫 번째 핵심 구성 요소는 OneWayPropertyBinding으로이 예제에서 보여줍니다. 보기의 특성과보기 모델간에 단방향 데이터 바인딩을 작성하는 데 사용됩니다.

이 UI는 가능한 한 간단합니다. 주기적으로 임의의 값으로 업데이트하는 텍스트 상자입니다.

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/03/1_One_way_binding_1.png)

계층 구조를보고 UI의 구조를 확인하십시오. 비대화 형 텍스트 필드 [B]가 포함 된 Canvas [A]가 있습니다.

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/03/1_One_way_binding_2.png)

이 예제의 UI 요소는 MyViewModel1.cs의 뷰 모델에 의해 조정됩니다. 캔버스 [A]를 선택하고 속성에서 MyViewModel1 [B]가 단순히 계층에 연결된 MonoBehaviour인지 확인하십시오.

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/03/1_One_way_binding_3-1.png)

다음은 View Model 과 View 간의 연결에 대한 개념적 개요입니다.

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/03/1_One_way_binding_4-1.png)

계층 구조에서 텍스트 필드 [A]를 선택하고 One Way Property Binding [B]가 첨부되어 있는지 확인하십시오.

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/03/1_One_way_binding_5.png)

OneWayPropertyBinding은 뷰의 text 속성을 view-model의 Text 속성에 데이터 바인딩하는 풀입니다. 속성의 값은 단일 방향으로 만 전파되므로 뷰 모델에 대한 변경 사항은 뷰에 전파되지만 다른 방식으로는 전파되지 않습니다. 이것은 Unity의 텍스트 필드가 비대화 적이기 때문에 의미가 있습니다. UI에서 값을 변경할 수 없으므로 뷰 모델로 다시 전달해야하는 것이 없습니다.

Unity Inspector를 통해 데이터 바인딩을 구성 할 수 있습니다. OneWayPropertyBinding을 사용하면 뷰 [A]의 속성을 뷰 모델 [B]의 속성에 연결할 수 있습니다. 이 예제에서는 텍스트 속성 [C]가 뷰 모델의 텍스트 속성 [D]에 붙었습니다.

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/03/1_One_way_binding_6.png)

Unity Editor에서 드롭 다운 메뉴를 사용하여 사용 가능한 속성 중 하나를 선택할 수 있습니다 :

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/03/1_One_way_binding_7.png)

우리는 계층 구조를 살펴본 후 뷰 모델이 뷰 모델과 어떻게 융합되는지 살펴 보았습니다. 뷰 모델 자체가 어떻게 생성되는지 살펴 보겠습니다. 다음은 개요입니다 (일부 코드 섹션은 접혀 있음).

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/03/1_One_way_binding_8.png)

INotifyPropertyChanged 인터페이스는 뷰 모델과 뷰 간의 속성 동기화를 지원합니다. 뷰 모델은 PropertyChanged 이벤트를 발생시켜 Unity-Weld에 값이 변경되고 UI가 새로 고쳐 져야 함을 알립니다. 이것은 WPF에서 사용되는 것과 동일한 데이터 바인딩 메커니즘입니다.

Binding Attribute는 UI에 데이터 바인딩 될 속성에 주석을 추가합니다.

​```
[Binding]
public string Text
{
    get
    {
        return text;
    }
    set
    {
        if (text == value)
        {
            return; // No change.
        }

        text = value;

        OnPropertyChanged("Text");
    }
}
​```

이러한 속성에 대한 설정자는 PropertyChanged 이벤트를 발생시킵니다. 여기에 사용 된 OnPropertyChanged는 이벤트를 발생시키는 헬퍼 메서드입니다.

```
private void OnPropertyChanged(string propertyName)
{
    if (PropertyChanged != null)
    {
        PropertyChanged(this, new PropertyChangedEventArgs(propertyName));
    }
}
```
#### 2. "Two way binding" to input field

Unity-Weld의 두 번째 핵심 구성 요소는 TwoWayPropertyBinding입니다. 이 예제에서는 뷰 모델 속성과 UI 속성 사이에 양방향 데이터 바인딩을 만드는 방법을 보여줍니다.

여기 예제 UI는 이전 예제보다 약간 더 복잡합니다. 우리는 읽기 전용 텍스트 필드와 편집 가능한 입력 필드를 가지고 있습니다. 입력 필드에 입력하면 업데이트 된 값이 자동으로 텍스트 필드에 복제됩니다.

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/03/2_Two_way_binding_1.png)

두 번째 예제의 계층 구조는 첫 번째 예제를 확장합니다. 캔버스 [A]와 텍스트 필드 [B]가 있습니다. 또한 TwoWayPropertyBinding 구성 요소 [D]가 첨부 된 InputField [C]가 있습니다.

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/03/2_Two_way_binding_2-1.png)

이렇게하면 뷰의 text 속성과 view-model의 Text 속성 사이에 양방향 관계가 설정됩니다. UI에서 입력 필드가 변경되면 새 값이 자동으로 뷰 모델에 푸시됩니다. 반면에 뷰 모델에서 입력이 변경 될 때마다 값이 자동으로 UI로 푸시됩니다. 양방향 관계는 아래 다이어그램에 설명되어 있습니다.

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/03/2_Two_way_binding_3.png)

#### 3. Event binding to a button

유니티 - 웰드 (Unity-Weld)의 세 번째 핵심 구성 요소이자이 글에서 마지막으로 살펴본 것은 EventBinding입니다. 이렇게하면 버튼 클릭과 같은 이벤트를 뷰 모델의 함수에 바인딩 할 수 있으므로 해당 이벤트가 발생하면 뷰 모델 함수가 호출됩니다.

이 예제에서 UI는 다음과 같습니다.

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/03/3_Event_binding_1.png)

단추를 누를 때마다 큐브가 점차적으로 회전합니다.

계층 구조는 다음과 같습니다. 우리는 EventBinding 컴포넌트 [B]가 첨부 된 UI에서 Button 요소 [A]를 가지고 있습니다.

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/03/3_Event_binding_2.png)

EventBinding에는 Unity 이벤트[A]와 ViewModel메소드[B]를 선택할 수있는 메뉴가 있습니다.

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/03/3_Event_binding_3.png)

이벤트가 발생하면 다음 다이어그램과 같이 뷰 모델 함수가 자동으로 호출됩니다.

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/03/3_Event_binding_4.png)

이 예제의보기 모델은 매우 간단합니다. Binding 속성을 사용하여 특정 클래스 및 메소드를 마크 업합니다.

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/03/3_Event_binding_5.png)

이 예제에서 MyViewModel3은 INotifyPropertyChanged를 구현하지 않은 이유는, 우리가 속성 바인딩을 사용하고 있지 않기 때문입니다. 여기서는 EventBinding 만 사용했습니다.

#### 4. Putting it all together

예제 4는 앞의 3 가지 예제에서 설명한 모든 구성 요소를 결합하여 함께 작동하는 방법을 보여줍니다.

이 UI에는 3D 큐브 [D]의 회전을 표시하는 텍스트 필드 [A]가 포함됩니다. 입력 필드 [B]는 회전에 대한 새 값을 입력하는 데 사용됩니다. 큐브를 점진적으로 회전시키는 [C] 버튼이 있습니다.

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/03/4_Combined_example_1.png)

계층 구조 및 뷰 모델의 설정은 이전 예제의 조합이므로 구체적으로 다루지는 않습니다.

그러나 다음 다이어그램에서 계층 구조와 뷰 모델 간의 완전한 관계를 살펴보면 모든 것이 어떻게 연결되어 있는지 확인할 수 있습니다.

![no-alignment](http://www.what-could-possibly-go-wrong.com/content/images/2017/03/4_Combined_example_2.png)

예제의 유니티 씬을 보시고 그것이 모두 어울리는 지 확인하십시오.

