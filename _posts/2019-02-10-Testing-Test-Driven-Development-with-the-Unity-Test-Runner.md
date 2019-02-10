---
layout: single
title: "Testing Test-Driven Development with the Unity Test Runner"
categories: 
  - TDD
tags:
  - TDD
  - Unity
---

[출처 : blogs.unity3d](https://blogs.unity3d.com/kr/2018/11/02/testing-test-driven-development-with-the-unity-test-runner/)

> 테스트 주도 개발 (TDD)은 코드 자체를 작성하기 전에 코드 조각에 대한 자동화 된 테스트를 작성하는 관행입니다. 이 블로그 게시물에서는 동료와 내가 TDD를 (코드 스니펫을 사용하여) 게임을 만드는 데 어떻게 사용했는지, 그리고 잘된 점과 그렇지 않은 점을 설명 할 것입니다. TDD는 완전히 수정 된 것은 아니지만 우리는 많은 것을 배웠고 더 나은 개발자가되었습니다. Unity에서 NUnit 테스트를 작성하고 실행하는 시스템 인 Unity Test Runner를 프로젝트에 사용했습니다.

TDD의 일반적인 작업 과정은 다음과 같습니다. :

1. 먼저 코드가 수행 할 작업을 결정합니다. 코드가 codeHasRun의 값을 false에서 true로 설정한다고 가정 해 보겠습니다.
2. 그런 다음 코드가 작업을 완료했는지 확인하는 테스트를 작성합니다. 이 경우 테스트는 codeHasRun이 true인지 확인합니다.
3. 테스트를 실행하십시오. 코드가 아직 작성되지 않았기 때문에 테스트가 실패해야합니다. 테스트가 실패하지 않으면 테스트에 문제가 있거나 코드를 이해 한 것입니다.
4. 코드를 작성하십시오.
5. 테스트를 실행하십시오. 이제 테스트가 통과해야합니다.

이 워크 플로를 따르면 코드를 리팩토링하고 변경 작업을 신속하게 처리 할 수 있습니다. 왜 깨 졌는지, 왜 그런지 즉시 볼 수 있기 때문입니다.

코드 자체를 작성하기 전에 왜 테스트를 작성해야하는지 궁금 할 것입니다. 이는 코드를 작성한 후에 테스트를 작성하면 개발자가 테스트를 작성하여 테스트를 통과 할 수 있기 때문입니다. 실패한 테스트를 먼저 작성하면 제대로 된 이유 (예 : 필수 기능을 올바르게 구현하지 못하는 것)와 긍정오류 (false positive)를 배제하는것을 실패했는지 확인할 수 있습니다.

TDD는 일반적으로 소프트웨어 개발에 사용되지만 게임 개발에서는 매우 드뭅니다.

지난 달 유니티 릴리스 엔지니어링 그룹의 다른 팀원 5 명이 함께 모여 TDD를 사용하여 게임을 만들었습니다. 이전 일부 개발자는 자신의 게임 코드로 자동화 된 게임을 가질 수 없다고 생각한다고 들었습니다. 그들은 TDD를 사용할 수 없다고 생각하기에 우리는 직접 해보았습니다.

우리는 Pong, Snake, Asteroids, Flappy Bird와 같은 고전 게임을 만들기로 결정했습니다. 이것의 이점은 게임 플레이를 디자인 할 때 시간을 낭비 할 필요가 없었기 때문입니다. 우리는 이미 모든 것이 어떻게 결합 될지 (또는 그렇게 생각했는지) 대략적인 아이디어가 있었기 때문입니다.

우리는 각 게임이 어떻게 작동하는지 알았지 만 개념을 자세히 조사해야 테스트를 구성하는 방법을 알 수있었습니다. Pong의 예를 보면 나는 패들이 움직여야한다는 것을 알았습니다 ... 하지만 패들은 무엇입니까? 모든 아이디어는 세분화되어야했습니다.

우리는 퐁 (Pong)의 패들을 아래의 속성들로 쪼갰습습니다 :

* (0.5, 2, 0)의 눈금을 가진 사각형 입니다.
* XY 평면에서 움직일 수 있습니다.
* Z에서 움직일 수 없습니다.
* 왼쪽 및 오른쪽으로 이동할 수 없습니다.
* 화면 경계 밖으로 이동할 수 없습니다.
* collision이 있습니다.
* 키네마틱 리지드 바디 입니다.

이렇게하면 테스트를 작성하는 데 좋은 출발점이되었습니다. ex)
```
       [Test]
       public void AtLeastOnePaddleIsSuccesfullyCreated()
       {
           GameObject[] paddles = CreatePaddles();

           // Assert that the paddles object exists
           Assert.IsNotNull(paddles);
       }
       [Test]
       public void TwoPaddlesAreSuccesfullyCreated()
       {
           GameObject[] paddles = CreatePaddles();

           // Assert that the number of paddles equals 2
           Assert.AreEqual(2, paddles.Length);
       }
```

이 테스트에서 두 개의 GameObject 배열을 만드는 CreatePaddles라는 메서드를 작성해야한다는 것을 알 수 있습니다.

Unity Test Runner는 UnityTest와 같은 기능을 포함합니다. UnityTest는 IEnumerator를 반환하고 코루틴 (conoutine)으로 재생 모드에서 실행됩니다. 이 기능을 사용하면 프레임을 완료해야하는 작업을 테스트 할 수 있습니다.

이 예제에서는 UnitTest를 사용하여 패들의 경계를 벗어날 수 없는지 확인했습니다.

```
  [UnityTest]
    public IEnumerator Paddle1StaysInUpperCameraBounds()
    {
        // Increase the timeScale so the test executes quickly
        Time.timeScale = 20.0f;

        // _setup is a member of the class TestSetup where I store the code for
        //setting up the test scene (so that I don’t have a lot of copy-pasted code)
        Camera cam = _setup.CreateCameraForTest();

        GameObject[] paddles = _setup.CreatePaddlesForTest();

        float time = 0;
        while (time < 5)
        {
            paddles[0].GetComponent<Paddle>().RenderPaddle();
            paddles[0].GetComponent<Paddle>().MoveUpY("Paddle1");
            time += Time.fixedDeltaTime;
            yield return new WaitForFixedUpdate();
        }

        // Reset timeScale
        Time.timeScale = 1.0f;

        // Edge of paddle should not leave edge of screen
        // (Camera.main.orthographicSize - paddle.transform.localScale.y /2) is where the edge
        //of the paddle touches the edge of the screen, and 0.15 is the margin of error I gave it
        //to wait for the next frame
        Assert.LessOrEqual(paddles[0].transform.position.y, (Camera.main.orthographicSize - paddles[1].transform.localScale.y /2)+0.15f);
    }
```

단위 테스트는 가능한 가장 작은 기능을 테스트해야하므로 보드의 상단과 하단 모두에 대해 각 패들을 테스트했습니다.

여기서 deltaTime을 사용하면 테스트가 달라질 수 있으므로 불안정합니다. 이것을 예측 가능하게 만들기 위해 Time.captureFramerate 또는 fixedDeltaTime을 설정했습니다. 처음에 Time.timeScale을 설정하지 않으면 테스트가 완료 될 때까지 5 초가 걸릴 것입니다. 이는 한 테스트에서 용납 될 수 없습니다! timeScale을 20으로 설정하면 시간이 정상보다 20 배 빠르게 전달됩니다. 즉, 5 초 테스트는 대략 0.25 초 내에 실행될 수 있습니다.

위의 테스트에서 나는 패들을 5 초 동안 위쪽으로 움직이는 것을 시뮬레이션합니다. 이 테스트는 MoveUpY 방법으로 패들이 보드에서 떨어지지 않도록 제한되었는지 확인합니다. 처음 테스트를 작성했을 때, MoveUpY에는 패들이 보드 밖으로 이동하는 것을 막을 수있는 기능이 없었습니다. 이것은 테스트가 실패했음을 의미합니다.

테스트를 실제로 작성한 후에 실제로 실패했는지 확인한 후 실제 기능을 추가하거나 오 탐지 (false positive)로 끝날 수 있습니다. 초기 프로젝트에서 모든 걸 기다리는 동안 테스트를 작성하고 실패했는지 확인하는 것을 잊어 버렸습니다. 그런 다음 게임에서 기능이 손상되었지만 테스트가 진행 중임을 알았습니다. 내가 내 테스트를 확인하기 위해 되돌아 갔을 때 나는 그것을 잘못 쓰고 있음을 깨달았다. (나는 실제로 테스트하고있는 메소드를 호출하는 것을 잊었다.)! 당황스럽지만 필자는 배웠고 이후엔 기능을 작성하기 전에 테스트 실패를 확인했습니다. Unity Test Runner를 사용하면 실패한 테스트를 다시 실행할 수 있으므로 프로젝트에 많은 테스트가있는 경우 반복 시간을 단축 할 수 있습니다!

TDD를 사용하는 것은 약간 느린 시작일 수 있지만 일단 시작하면 매우 보람있는 작업 방식입니다. 또한 나중에 프로젝트의 변경 사항이 더 빠르고 안전하다는 것을 의미합니다!

이 작업 방식은 게임 내에서 서로 다른 시스템이 어떻게 작동 할 것인지를 결정하는 데 도움이되었습니다. 우리가 이미 알고있는 게임을 재창조하는 동안 우리는 디자인 내의 개별 시스템이 어떻게 결합 될지 확신하지 못했습니다. 테스트를 작성한다는 것은 우리가 일을하고 싶었던 방법을 계획 할 수 있다는 것을 의미했습니다. 그런 다음 실제로 실행 가능한지 확인하기 위해 실습에 투입했습니다. 우리가 필요하다면 거기에서 반복 할 수 있습니다. TDD를 사용하면 필자가 작성한 것과 그 이유에 대해 더 많은 생각을 가졌기 때문에 더 좋고 깔끔한 코드를 작성할 수 있다는 것을 알게되었습니다.

TDD는 픽스가 아니지만 좋은 안전망이며 모든 테스트가 완료되면 프로젝트에서 더 빠른 반복이 가능하다는 것을 명심하는 것이 매우 중요합니다. Unity Test Runner 창에 나타나는 모든 녹색 눈금을 보는 것은 정말 만족스러운 느낌이었습니다. 그러나 TDD를 사용한다고해서 다른 종류의 테스트가 필요하지않다는건 아닙니다. TDD는 QA 전략보다는 품질 중심 개발 방식입니다.
