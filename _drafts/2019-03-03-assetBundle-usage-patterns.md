---
layout: single
title: "AssetBundle usage patterns"
categories: 
  - 프로그래밍
tags:
  - Unity
  - Assetbundle
---
[출처](https://unity3d.com/learn/tutorials/topics/best-practices/assetbundle-usage-patterns?playlist=30089)

이 시리즈의 이전 장에서는 다양한 로딩 API의 저수준 동작을 포함하는 AssetBundles의 기본 사항에 대해 설명했습니다. 이 장에서는 실제로 AssetBundles를 사용하는 다양한 측면에 대한 문제와 잠재적 인 솔루션에 대해 설명합니다.

## Managing loaded Assets
메모리에 민감한 환경에서는, 로드 된 객체의 크기와 수를 신중하게 제어하는 것이 중요합니다. 유니티는 활성 장면에서 객체를 제거 할 때 객체를 자동으로 언로드하지 않습니다. 자산 cleanup은 특정 시간에 트리거되며 수동으로 트리거 될 수도 있습니다.

AssetBundles 자체는 신중하게 관리해야합니다. AssetBundle은 로컬 스토리지에있는 파일 (Unity 캐시 또는 AssetBundle.LoadFromFile을 통해로드 된 파일)에 의해 지원되므로 최소한의 메모리 오버 헤드를 가지므로 수십 킬로바이트 이상을 소비하지 않습니다. 그러나 많은 수의 AssetBundle이 있으면이 오버 헤드가 여전히 문제가 될 수 있습니다.

대부분의 프로젝트에서 사용자는 콘텐츠 재생 (예: 레벨 재생)을 할 수 있으므로 AssetBundle을 로드하거나 언로드 할시기를 알아야합니다. **AssetBundle이 부적절하게 언로드되면 메모리에 Object 복제가 발생할 수 있습니다.** 애셋 번들을 부적절하게 언로드하면 텍스처가 누락되는 등의 특정 상황에서 바람직하지 않은 동작이 발생할 수도 있습니다.

**자산 및 AssetBundle을 관리 할 때 이해해야 할 가장 중요한 점은 unloadAllLoadedObjects 매개 변수에 대해 AssetBundle.Unload를 true 또는 false로 호출 할 때 동작의 차이입니다.**

이 API는 호출중인 AssetBundle의 헤더 정보를 언로드합니다. unloadAllLoadedObjects 매개 변수는이 AssetBundle에서 인스턴스화 된 모든 객체를 언로드할지 여부를 결정합니다. true로 설정하면 AssetBundle에서 비롯된 모든 객체가 현재 활성 장면에서 사용되고있는 경우에도 즉시 언로드됩니다.

예를 들어, M이 AssetBundle AB에서로드되었고 M이 현재 활성 장면에 있다고 가정합니다.

!(https://unity3d.com/sites/default/files/ab2a.jpg)

AssetBundle.Unload (true)가 호출되면 M은 장면에서 제거되고 파괴되고 언로드됩니다. 그러나 AssetBundle.Unload (false)가 호출되면 AB의 헤더 정보가 언로드되지만 M은 장면에 남아 있고 계속 작동합니다. AssetBundle.Unload (false)를 호출하면 M과 AB 사이의 링크가 끊어집니다. AB가 나중에 다시로드되면 AB에 포함 된 객체의 새로운 사본이 메모리에로드됩니다.

!(https://unity3d.com/sites/default/files/ab2b.jpg)

AB가 나중에 다시로드되면 AssetBundle의 헤더 정보의 새로운 사본이 다시로드됩니다. 그러나 M은 AB의 새로운 사본에서로드되지 않았습니다. 유니티는 AB와 M.의 새 복사본 사이의 링크를 설정하지 않습니다

!(https://unity3d.com/sites/default/files/ab2c.jpg)

AssetBundle.LoadAsset ()가 M을 다시로드하기 위해 호출 된 경우 Unity는 M의 이전 복사본을 AB의 데이터 인스턴스로 해석하지 않습니다. 그러므로 Unity는 M의 새로운 복사본을로드 할 것이고 M의 동일한 복사본 두 개가 장면에있게 될 것입니다.

!(https://unity3d.com/sites/default/files/ab2d.jpg)

대부분의 프로젝트에서이 동작은 바람직하지 않습니다. 대부분의 프로젝트는 AssetBundle.Unload (true)를 사용하고 객체가 중복되지 않도록 보장하는 메소드를 채택해야합니다. 일반적인 두 가지 방법은 다음과 같습니다.
1. 레벨 간 또는 로딩 화면과 같이 일시적인 AssetBundles이 언로드되는 애플리케이션의 수명 동안 잘 정의 된 포인트가 있습니다. 이것은 간단하고 가장 일반적인 옵션입니다.
2. 개별 Object에 대한 참조 카운트 유지 및 모든 구성 객체가 사용되지 않은 경우에만 AssetBundle 언로드. 이렇게하면 응용 프로그램이 메모리를 복제하지 않고 개별 객체를 언로드하고 다시로드 할 수 있습니다.

응용 프로그램에서 AssetBundle.Unload (false)를 사용해야만 하는 경우, 개별 객체는 다음 두 가지 방법으로만 언로드 할 수 있습니다.
1. 장면과 코드에서 원하지 않는 객체에 대한 모든 참조를 제거하십시오. 이 작업이 완료되면 Resources.UnloadUnusedAssets를 호출하십시오.
2. 장면을 비가 상적으로로드하십시오. 그러면 현재 장면의 모든 객체가 삭제되고 Resources.UnloadUnusedAssets가 자동으로 호출됩니다.

프로젝트가 게임 모드 나 레벨 사이에서와 같이 객체로드 및 언로드를 기다릴 수있는 잘 정의 된 지점이있는 경우 이러한 점을 사용하여 필요한 만큼의 객체를 언로드하고 새 객체를로드해야합니다.

가장 간단한 방법은 프로젝트의 각 덩어리를 씬에 패키지화 한 다음 해당 씬을 모든 종속성과 함께 AssetBundles로 빌드하는 것입니다. 그런 다음 응용 프로그램은 "로드 중" 씬을 입력하고 이전 장면이 포함 된 AssetBundle을 완전히 언로드 한 다음 새 씬에 포함 된 AssetBundle을 로드 할 수 있습니다.

이것이 가장 간단한 흐름이지만 일부 프로젝트는보다 복잡한 AssetBundle 관리가 필요합니다. 모든 프로젝트가 다르기 때문에 보편적 인 AssetBundle 디자인 패턴이 없습니다.

객체를 AssetBundle로 그룹화하는 방법을 결정할 때, 객체를 동시에로드하거나 업데이트해야하는 경우 객체를 AssetBundle에 번들링하는 것이 일반적으로 가장 좋습니다. 예를 들어, 롤 플레잉 게임을 생각해보십시오. 개별 맵과 컷씬은 장면별로 AssetBundles로 그룹화 할 수 있지만 대부분의 장면에서는 일부 오브젝트가 필요할 것입니다. AssetBundles는 인물 게임, 인게임 UI 및 다양한 캐릭터 모델과 텍스처를 제공하도록 제작할 수 있습니다. 이 후자의 개체와 자산은 시작시로드되고 앱의 수명 동안로드 된 상태로 유지되는 두 번째 AssetBundle 집합으로 그룹화 할 수 있습니다.

AssetBundle이 언로드 된 후에 Unity가 AssetBundle에서 객체를 다시로드 해야하는 경우 또 다른 문제가 발생할 수 있습니다. 이 경우 리로드가 실패하고 Object가 Unity Editor의 계층 구조에 (Missing) 객체로 나타납니다.

**이는 주로 Unity가 모바일 앱이 일시 중지되거나 사용자가 자신의 PC를 잠그는 등 그래픽 컨텍스트에 대한 제어권을 잃어 버리고 다시 회복 할 때 발생합니다. 이 경우 Unity는 텍스처와 쉐이더를 GPU에 다시 업로드해야합니다.** 이러한 애셋의 소스 AssetBundle을 사용할 수없는 경우 응용 프로그램은 장면의 객체를 자홍색으로 렌더링합니다.

## 4.2. Distribution (배포)
프로젝트의 AssetBundles를 클라이언트에 배포하는 두 가지 기본 방법은 프로젝트와 동시에 설치하거나 설치 후 다운로드하는 것입니다.

AssetBundle을 설치 중에 또는 이후에 제공할지 여부는 프로젝트가 실행될 플랫폼의 기능 및 제한 사항에 따라 결정됩니다. 모바일 프로젝트는 일반적으로 초기 설치 크기를 줄이고 무선 다운로드 크기 제한을 유지하기 위해 설치 후 다운로드를 선택합니다. 콘솔 및 PC 프로젝트는 일반적으로 초기 설치시 AssetBundles를 제공합니다.

적절한 아키텍처를 사용하면 처음에 AssetBundle을 전달하는 방법과 상관없이 프로젝트의 사후 설치에 새롭거나 수정 된 내용을 패치 할 수 있습니다. 이에 대한 더 자세한 정보는 Unity 매뉴얼의 [AssetBundles로 패치](https://docs.unity3d.com/Manual/AssetBundles-Patching.html?_ga=2.19443519.60866194.1551365663-678518112.1480121168)하기 섹션을 참조하십시오.

### 4.2.1. Shipped with project


