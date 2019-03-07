---
layout: single
title: "The Resources folder"
categories: 
  - 프로그래밍
tags:
  - Unity
  - Asset 관리
link: https://unity3d.com/learn/tutorials/topics/best-practices/resources-folder?playlist=30089
---
이 장에서는 Resources 시스템에 대해 설명합니다. 이것은 개발자가 Resources라는 하나 이상의 폴더에 Assets을 저장하고 Resources API를 사용하여 런타임에 해당 Assets에서 Object를 로드/언로드 할 수 있게 해주는 시스템입니다.

## 2.1. Best Practices for the Resources System
**Don't use it.**
여러 가지 이유로 사용하지 않아야 합니다.
* Resources 폴더를 사용하면 세분화 된 메모리 관리가 더욱 어려워집니다.
* 리소스 폴더를 잘못 사용하면 응용 프로그램 시작 시간과 빌드 길이가 늘어납니다.
  * 리소스 폴더의 수가 증가하면 해당 폴더 내의 자산 관리가 매우 어려워집니다
* 리소스 시스템은 특정 플랫폼에 사용자 지정 콘텐츠를 제공하는 프로젝트의 기능을 저하시키고 점진적인 콘텐츠 업그레이드의 가능성을 제거합니다.
  * AssetBundle Variant는 장치별로 콘텐츠를 조정하는 Unity의 기본 도구입니다.

## 2.2. Proper uses of the Resources system
자원 시스템이 도움이 될 수 있는 두 가지 구체적인 사용 사례가 있습니다.

1. Resources 폴더의 용이성은 신속하게 프로토 타입을 작성하는 탁월한 시스템입니다. 그러나 프로젝트를 완전히 제작하려면 Resources 폴더를 사용하지 않아야합니다.
2. Resources 폴더는 내용이 다음과 같은 사소한 경우에 유용 할 수 있습니다.
  * 프로젝트의 전체 lifetime 동안 필요할때.
  * 메모리를 많이 사용하지 않을때.
  * 패치가 발생하지 않거나 플랫폼이나 장치에 따라 다르지 않을때.
  * 최소 부트 스트랩에 사용할때.

다른예는 Prefab을 호스팅하는 데 사용되는 MonoBehaviour 싱글톤 또는 Facebook App ID와 같은 타사 구성 데이터가 포함 된 ScriptableObject가 포함됩니다.

## 2.3. Serialization of Resources
"Resources"이라는 이름의 모든 폴더에있는 자산 및 개체는 프로젝트가 빌드 될 때 하나의 직렬화 된 파일로 결합됩니다. 이 파일에는 AssetBundle과 비슷한 메타 데이터 및 색인 정보도 들어 있습니다. AssetBundle 문서에서 설명한 것처럼이 인덱스에는 주어진 Object의 이름을 해당 File GUID와 Local ID로 변환하는 데 사용되는 일련화 된 검색 트리가 포함됩니다. 또한 직렬화 된 파일 본문의 특정 바이트 오프셋에서 Object를 찾는 데 사용됩니다.

대부분의 플랫폼에서 조회 데이터 구조는 O(n log (n)) 속도로 증가하는 구축 시간을 갖는 균형 검색 트리입니다. 또한이 증가로 인해 리소스 폴더의 개체 수가 증가함에 따라 인덱스의 로딩 시간이 선형적으로 증가합니다.

이 작업은 건너 뛸 수 없으며 초기 비 대화식 시작 화면이 표시되는 동안 응용 프로그램 시작시 발생합니다. 리소스 폴더에 포함 된 대부분의 개체가 실제로 응용 프로그램의 첫 번째 장면으로로드하는 데 거의 필요하지 않더라도 10,000 개의 자산을 포함하는 리소스 시스템을 초기화하면 로우 엔드 모바일 장치에서 여러 초를 소비하는 것으로 나타났습니다.