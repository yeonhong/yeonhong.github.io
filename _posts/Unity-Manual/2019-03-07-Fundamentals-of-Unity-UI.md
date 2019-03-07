---
layout: single
title: "Fundamentals of Unity UI"
categories: 
  - 프로그래밍
tags:
  - Unity
  - Optimaze
  - Unity UI
link: https://unity3d.com/learn/tutorials/topics/best-practices/fundamentals-unity-ui?playlist=30089
---
Unity UI 시스템을 구성하는 여러 부분을 이해하는 것이 중요합니다. 함께 시스템을 구성하는 몇 가지 기본 클래스와 component가 있습니다. 이 장에서는 먼저 이 연재 기사 전반에 걸쳐 사용 된 여러 용어를 정의한 다음 Unity UI의 주요 시스템에 대한 하위 수준 동작에 대해 설명합니다.

# Terminology (용어정리)
Canvas는 유니 코드의 렌더링 시스템이 게임의 월드 공간에서 그려지는 계층화 된 기하학을 제공하기 위해 사용되는 기본 코드 유니티 component입니다.

Canvas는 해당 구성 Geometry를 배치로 결합하고 적절한 렌더링 명령을 생성하여 유니티의 그래픽 시스템에 전송합니다. 이 모든 것은 원시 C++ 코드에서 수행되며 재 구현 또는 일괄 빌드라고합니다. Canvas가 재배치가 필요한 형상을 포함하는 것으로 표시되면 Canvas를 dirty로 생각합니다.

Geometry는 Canvas들에게 Canvas renderer component를 제공합니다.

하위 Canvas는 단순히 다른 Canvas component 안에 중첩 된 Canvas component입니다. 하위 Canvas는 자녀를 부모로부터 격리시킵니다. dirty child는 부모에게 그 geometry를 재구성하도록 강요하지 않으며 그 반대의 경우도 마찬가지입니다. 부모 Canvas를 변경하면 자식 Canvas의 크기가 조정되는 경우와 같이 특정 상황이 적용되지 않을 수 있습니다.

Graphic은 Unity UI C# 라이브러리에서 제공하는 기본 클래스입니다. Canvas 시스템에 drawable geometry를 제공하는 모든 Unity UI C# 클래스의 기본 클래스입니다. 내장 된 Unity UI Graphic은 대부분 MaskableGraphic 서브 클래스를 통해 구현되며, 이를 통해 IMaskable 인터페이스를 통해 마스크를 적용 할 수 있습니다. Drawable의 주요 하위 클래스는 이미지와 텍스트로, 동일한 이름의 component를 제공합니다.

layout component는 RectTransform의 크기와 위치를 제어하며 일반적으로 내용의 상대 크기 조정이나 상대 위치 지정이 필요한 복잡한 layout을 만드는 데 사용됩니다. layout component는 RectTransforms에만 의존하며 연관된 RectTransforms의 속성에만 영향을줍니다. 이들은 Graphic 클래스에 의존하지 않으며 Unity UI의 그래픽 component에 독립적으로 사용할 수 있습니다.

Graphic 및 layout component는 모두 Unity Editor 인터페이스에 표시되지 않는 CanvasUpdateRegistry 클래스를 사용합니다. 이 클래스는 업데이트해야하는 layout component 및 그래픽 component 집합을 추적하고 관련 Canvas가 willRenderCanvases 이벤트를 호출 할 때 필요에 따라 업데이트를 트리거합니다.

레이아웃 및 그래픽 component의 업데이트를 rebuild 라고합니다. rebuild 프로세스는이 문서의 뒷부분에서 자세히 설명합니다.

# Rendering details
Unity UI에서 사용자 인터페이스를 작성할 때 Canvas에 의해 그려지는 모든 지오메트리가 투명한 대기열에 그려지는 것을 명심하십시오. 즉, Unity UI에서 생성 된 지오메트리는 항상 알파 블렌딩으로 뒤에서 앞으로 그려집니다. **성능 관점에서 기억해야 할 중요한 점은 다각형에서 래스터 화 된 각 픽셀은 다른 불투명 한 다각형으로 완전히 덮여 있더라도 샘플링된다는 점입니다.** 모바일 장치에서이 높은 수준의 오버 드로는 GPU의 유효 용량을 빠르게 초과 할 수 있습니다.

# The Batch building process (Canvases)
일괄 처리 프로세스는 Canvas가 UI 요소를 나타내는 메시를 결합하고 Unity의 그래픽 파이프 라인에 보낼 적절한 렌더링 명령을 생성하는 프로세스입니다. 이 프로세스의 결과는 Canvas가 더티로 표시 될 때까지 캐싱되고 다시 사용됩니다. 이는 구성 메쉬 중 하나가 변경 될 때마다 발생합니다.

캔버스에서 사용하는 메쉬는 캔버스에 첨부되었지만 모든 하위 캔버스에 포함되지 않은 캔버스 렌더러 구성 요소 집합에서 가져옵니다.

배치를 계산하려면 깊이로 메쉬를 정렬하고 중첩, 공유 재료 등을 검사해야합니다. 이 작업은 멀티 스레드이므로 일반적으로 여러 CPU 아키텍처, 특히 모바일 SoC (일반적으로 CPU 코어가 거의 없음)와 최신 데스크톱 CPU (코어가 4 개 이상인 경우)의 성능이 매우 다릅니다.

# The rebuild process (Graphics)
rebuild 프로세스에서는 Unity UI의 C# 그래픽 구성 요소의 레이아웃과 메시가 다시 계산됩니다. 이 작업은 CanvasUpdateRegistry 클래스에서 수행됩니다. 이것은 C # 클래스이며 Unity의 Bitbucket에서 해당 소스를 찾을 수 있음을 기억하십시오.

CanvasUpdateRegistry 내에서 관심있는 메소드는 PerformUpdate입니다. 이 메서드는 Canvas 구성 요소가 [WillRenderCanvases](https://docs.unity3d.com/ScriptReference/Canvas-willRenderCanvases.html?_ga=2.209047762.1352180690.1551968663-1507229146.1546094146) 이벤트를 호출 할 때마다 호출됩니다. 이 이벤트는 프레임 당 한 번 호출됩니다.

PerformUpdate는 3단계 프로세스를 실행합니다.
1. dirty layout component는 [ICanvasElement.Rebuild](https://docs.unity3d.com/ScriptReference/UI.ICanvasElement.Rebuild.html?_ga=2.251072710.1352180690.1551968663-1507229146.1546094146) 메서드를 통해 레이아웃을 다시 작성해야합니다.
2. 등록 된 클리핑 구성 요소 (예 : 마스크)에는 클리핑 된 구성 요소를 제거해야합니다. 이 작업은 ClippingRegistry.Cull을 통해 수행됩니다.
3. dirty griphic component는 그래픽 요소를 rebuild를 요청합니다.

레이아웃 및 그래픽 재구성의 경우 프로세스가 여러 부분으로 분할됩니다. 레이아웃 재구성은 세 부분 (PreLayout, Layout 및 PostLayout)에서 실행되며 그래픽 재 작성은 두 단계 (PreRender 및 LatePreRender)에서 실행됩니다.

## Layout rebuilds
하나 이상의 레이아웃 구성 요소에 포함 된 구성 요소의 적절한 위치 (잠재적으로 크기)를 다시 계산하려면 적절한 계층 순서로 레이아웃을 적용해야합니다. GameObject 계층의 루트에 가까운 레이아웃은 중첩 될 수있는 레이아웃의 위치와 크기를 잠재적으로 변경할 수 있으므로 먼저 계산해야합니다.

이를 위해 Unity UI는 더티 레이아웃 구성 요소 목록을 계층 구조의 깊이별로 정렬합니다. 계층 구조의 상위 항목 (상위 변환이 적은 항목)이 목록의 맨 앞으로 이동됩니다.

그런 다음 레이아웃 구성 요소의 정렬 된 목록에 레이아웃을 rebuild 해야합니다. Layout 구성 요소에 의해 제어되는 UI 요소의 위치와 크기가 실제로 변경되는 곳입니다. 개별 요소의 위치가 레이아웃의 영향을받는 방식에 대한 자세한 내용은 Unity 설명서의 [UI auto layout 섹션](https://docs.unity3d.com/Manual/UIAutoLayout.html?_ga=2.85760217.1352180690.1551968663-1507229146.1546094146)을 참조하십시오.

## Graphic rebuilds
그래픽 구성 요소가 다시 작성되면 Unity UI는 컨트롤을 ICanvasElement 인터페이스의 [Rebuild 메서드](https://docs.unity3d.com/ScriptReference/UI.ICanvasElement.Rebuild.html?_ga=2.207540819.1352180690.1551968663-1507229146.1546094146)에 전달합니다. 그래픽은 이것을 구현하고 Rebuild 프로세스의 PreRender 단계에서 두 가지 다른 재구성 단계를 실행합니다.
* 버텍스 데이터가 dirty로 표시되면 (예 : 구성 요소의 RectTransform의 크기가 변경된 경우) 메시가 다시 작성됩니다.
* material 데이터가 더럽혀진 경우 (예 : 구성 요소의 material이나 texture가 변경된 경우), 부착 된 캔버스 렌더러의 material이 업데이트됩니다.

그래픽 재구성은 특정 순서로 그래픽 구성 요소 목록을 진행하지 않으므로 sort operation이 필요하지 않습니다.