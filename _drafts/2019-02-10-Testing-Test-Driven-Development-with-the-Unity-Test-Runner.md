---
layout: single
title: "Testing Test-Driven Development with the Unity Test Runner"
categories: 
  - TDD
tags:
  - TDD
  - Unity
---

> Test-driven development (TDD) is the practice of writing automated tests for a piece of code before writing the code itself. In this blog post, I’m going to explain how my colleagues and I used TDD for making games (with code snippets), as well as what went well and what didn’t. TDD is not a fix-all, but we definitely learned a lot and became better developers as a result. We used Unity Test Runner in our project, which is a system for writing and executing NUnit tests in Unity.

The usual workflow for TDD is as follows:

1. First, you determine what the code will do. Let’s say that the code sets the value of codeHasRun from false to true.
2. Next, you write a test that checks that the code has done its job. In this case, the test would check that codeHasRun equals true.
3. Run the test. The test should fail because the code hasn’t been written yet. If the test doesn’t fail, there’s something wrong with the test, or with your understanding of the code.
4. Write the code.
5. Run the test. The test should now pass.

Following this workflow speeds up the process of refactoring code and making changes, because you can see straight away what has broken and why.

You may wonder why we write the test before writing the code itself. This is because writing tests after writing the code can often lead to developers writing tests to make them pass. When you write a failing test first, you’re making sure that it fails for a good reason (such as not implementing the required functionality correctly), as well as ruling out false positives.

TDD is commonly used in software development, but it’s quite rare in game development.

Last month, five people from different teams in Unity’s Release Engineering Group got together to look into making games using TDD. I’d heard before that some developers think they can’t have automated games with their game code, and they certainly couldn’t use TDD, so I wanted to see for myself.

We decided to make a few classic games – Pong, Snake, Asteroids, and Flappy Bird. The benefit of this was that we didn’t need to spend any time designing gameplay, because we already had a rough idea of how everything would come together (or so we thought).

While we knew how each game would work, we still had to really drill down into the concepts so we could know how to structure our tests. With the example of Pong, I knew that a paddle should move… But what even is a paddle? Every idea had to be broken down.

We broke a paddle in Pong down to the following attributes:

* A rectangle with a scale of (0.5, 2, 0)
* Can move in in the XY plane
* Cannot move in Z
* Cannot move left and right
* Cannot move outside the bounds of the screen
* Has collision
* Is a Kinematic Rigidbody

Doing this gave us a good starting point for writing tests. For example:
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

From these tests, you can see that I need to write a method called CreatePaddles that creates an array of 2 GameObjects.

The Unity Test Runner includes functionality such as a UnityTest. A UnityTest returns an IEnumerator and runs in Play mode as a coroutine, which allows you to test actions that may need a frame or more to complete.

In this example, we used a UnityTest to check that the paddles could not leave the bounds of the board.

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

A unit test should test the smallest piece of functionality possible, so I tested each paddle for both the top and bottom of the board.

If I was to use deltaTime here, the test would be unstable because it can vary. I set up Time.captureFramerate, or fixedDeltaTime, to make this predictable. If I didn’t set Time.timeScale at the start, the test would take over 5 seconds to complete – which is unacceptable for one test! Setting timeScale to 20 means time passes 20 times quicker than normal. This means a 5-second test can execute in roughly 0.25 seconds.

In the above test, I simulate moving the paddle upwards for 5 seconds. The test checks that the paddle is restricted from moving off the board by the MoveUpY method. When I first wrote the test, MoveUpY didn’t have the functionality to stop the paddle from moving off the board. This meant that the test failed.

It’s really important to check that your test actually fails after you write it but before you add the actual functionality, or you could end up with false positives. Early on in the project, while I was still getting the hang of everything, I wrote a test, forgot to check that it was failing, then noticed that the tests were passing even though the functionality was broken in the game. When I went back to check my test, I realized I’d written it wrong (I’d actually forgotten to call the method I was testing…)! It was embarrassing, but I learned my lesson and made sure in future to check for test failures before writing the functionality. Unity Test Runner allows you to rerun failed tests, which can help speed up iteration time if you have lots of tests in a project!

Using TDD can be a bit of a slow start, but once you get going it’s a very rewarding way of working. It also means that changes later in the project are quicker and safer!

This way of working also helped us to shape how different systems would work within the game. While we were recreating games that we were already familiar with, we weren’t sure how individual systems within the design would come together. Writing the tests first meant that we could plan out how we wanted things to work, and then put them into practice to see if they were actually feasible. We could iterate from there if we needed to. I also found that using TDD caused me to write better and neater code because I was putting more thought into what I was writing and why.

It’s very important to bear in mind that TDD is not a fix-all, but it’s a nice safety net, and it allows for faster iteration in the project once all the tests are in place. Seeing all the green ticks appear in the Unity Test Runner window was a really satisfying feeling, too. However, using TDD doesn’t mean you don’t need any other kind of testing. TDD is a quality-driven development approach, rather than a QA strategy.

Thanks to my coworkers Marc Di Luzio, Ugnius Dovidaukas, Linas Ratkevičius, and Andy Selby for working with me on this project!
