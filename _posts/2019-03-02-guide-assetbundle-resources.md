---
layout: single
title: "A Guide to AssetBundles and Resources"
categories: 
  - 프로그래밍
tags:
  - Unity
  - AssetBundle
  - Asset 관리
---

이것은 Unity 엔진에서 Assets 및 리소스 관리에 대한 심층적인 토론을 제공하는 일련의 기사입니다. Unity는 전문 개발자에게 Unity의 자산 및 serialization 시스템에 대한 깊이있는 소스 수준의 지식을 제공하고자합니다. 그것은 Unity의 AssetBundle 시스템의 기술적인 토대와 이를 사용하는 현재의 모범 사례를 모두 검토합니다.

가이드는 네 장으로 나뉩니다.

1. Assest, 객체 및 serialization는 Unity가 Assets을 직렬화하고 자산 간의 참조를 처리하는 방법에 대한 하위 수준의 세부 사항을 설명합니다. 독자들이 본 안내서에서 사용되는 용어를 정의 할 때이 장에서 시작하는 것이 좋습니다.
2. 리소스 폴더는 기본 제공 리소스 API에 대해 설명합니다.
3. AssetBundle의 기본 요소는 1장의 정보를 바탕으로 AssetBundle의 작동 방식을 설명하고 AssetBundles로드 및 AssetBundle에서의 Asset로드에 대해 설명합니다.
4. AssetBundle 사용 패턴은 AssetBundles의 실제 사용을 둘러싼 많은 주제를 다루는 긴 기사입니다. AssetBundles에 자산 할당 및로드 된 자산 관리에 대한 섹션이 포함되어 있으며 AssetBundles를 사용하는 개발자가 자주 접하는 함정에 대해 설명합니다.

> 참고 : 이 안내서의 객체 및 자산 용어는 Unity의 공용 API 명명 규칙과 다릅니다.

이 가이드가 호출하는 데이터 객체는 [AssetBundle.LoadAsset](https://docs.unity3d.com/ScriptReference/AssetBundle.LoadAsset.html?_ga=2.31491769.60866194.1551365663-678518112.1480121168) 및 [Resources.UnloadUnusedAssets](https://docs.unity3d.com/ScriptReference/Resources.UnloadUnusedAssets.html?_ga=2.31491769.60866194.1551365663-678518112.1480121168)와 같은 많은 공용 Unity API에서 에셋이라고합니다. 
이 가이드에서 자산이라고하는 파일은 공개 API에 거의 노출되지 않습니다. 노출되면 일반적으로 [AssetDatabase](https://docs.unity3d.com/ScriptReference/AssetDatabase.html?_ga=2.259565069.60866194.1551365663-678518112.1480121168) 및 [BuildPipeline](https://docs.unity3d.com/ScriptReference/BuildPipeline.html?_ga=2.259565069.60866194.1551365663-678518112.1480121168)과 같은 빌드 관련 코드에서만 사용됩니다. 이러한 경우 Public APIs 파일이라고합니다.