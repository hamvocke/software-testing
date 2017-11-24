---
page.next: integration-tests.md
---

# Unit tests
The foundation of your test suite will be made up of unit tests. Your unit tests make sure that a certain unit (your _subject under test_) of your codebase works as intended. Unit tests have the narrowest scope of all the tests in your test suite. The number of unit tests in your test suite will largely outnumber any other type of test.

![unit tests](img/unitTest.png)
*A unit test typically replaces external collaborators with mocks or stubs*

## What's a Unit?
If you ask three different people what _"unit"_ means in the context of unit tests, you'll probably receive four different, slightly nuanced answers. To a certain extend it's a matter of your own definition and it's okay to have no canonical answer.

If you're working in a functional language a _unit_ will most likely be a single function. Your unit tests will call a function with different parameters and ensure that it returns the expected values. In an object-oriented language a unit can range from a single method to an entire class.

## Sociable and Solitary
Some argue that all collaborators (e.g. other classes that are called by your class under test) of your subject under test should be substituted with _mocks_ or _stubs_ to come up with perfect isolation and to avoid side-effects and complicated test setup. Others argue that only collaborators that are slow or have bigger side effects (e.g. classes that access databases or make network calls) should be stubbed or mocked.

[Occasionally](https://www.martinfowler.com/bliki/UnitTest.html) people label these two sorts of tests as **solitary unit tests** for tests that stub all collaborators and **sociable unit tests** for tests that allow talking to real collaborators (Jay Fields' [Working Effectively with Unit Tests](https://leanpub.com/wewut) coined these terms). If you have some spare time you can go down the rabbit hole and [read more about the pros and cons](https://martinfowler.com/articles/mocksArentStubs.html) of the different schools of thought.

At the end of the day it's not important to decide if you go for solitary or sociable unit tests. Writing automated tests is what's important. Personally, I find myself using both approaches all the time. If it becomes awkward to use real collaborators I will use mocks and stubs generously. If I feel like involving the real collaborator gives me more confidence in a test I'll only stub the outermost parts of my service.

## Mocking and Stubbing
[**Mocking** and **stubbing**](https://martinfowler.com/articles/mocksArentStubs.html) should be heavily used instruments in your unit tests.

In plain words it means that you replace a real thing (e.g. a class, module or function) with a fake version of that thing. The fake version looks and acts like the real thing (answers to the same method calls) but answers with canned responses that you define yourself at the beginning of your unit test.

Regardless of your technology choice, there's a good chance that either your language's standard library or some popular third-party library will provide you with elegant ways to set up mocks. And even writing your own mocks from scratch is only a matter of writing a fake class/module/function with the same signature as the real one and setting up the fake in your test.

Your unit tests will run very fast. On a decent machine you can expect to run thousands of unit tests within a few minutes. Test small pieces of your codebase in isolation and avoid hitting databases, the filesystem or firing HTTP queries (by using mocks and stubs for these parts) to keep your tests fast.

Once you got a hang of writing unit tests you will become more and more fluent in writing them. Stub out external collaborators, set up some input data, call your subject under test and check that the returned value is what you expected. Look into [Test-Driven Development](https://en.wikipedia.org/wiki/Test-driven_development) and let your unit tests guide your development; if applied correctly it can help you get into a great flow and come up with a good and maintainable design while automatically producing a comprehensive and fully automated test suite. Still, it's no silver bullet. Go ahead, give it a real chance and see if it feels right for you.

## What to Test?
The good thing about unit tests is that you can write them for all your production code classes, regardless of their functionality or which layer in your internal structure they belong to. You can unit tests controllers just like you can unit test repositories, domain classes or file readers. Simply stick to the **one test class per production class** rule of thumb and you're off to a good start.

A unit test class should at least **test the _public_ interface of the class**. Private methods can't be tested anyways since you simply can't call them from a different test class. _Protected_ or _package-private_ are accessible from a test class (given the package structure of your test class is the same as with the production class) but testing these methods could already go too far.

There's a fine line when it comes to writing unit tests: They should ensure that all your non-trivial code paths are tested (including happy path and edge cases). At the same time they shouldn't be tied to your implementation too closely.

Why's that?

Tests that are too close to the production code quickly become annoying. As soon as you refactor your production code (quick recap: refactoring means changing the internal structure of your code without changing the externally visible behavior) your unit tests will break.

This way you lose one big benefit of unit tests: acting as a safety net for code changes. You rather become fed up with those stupid tests failing every time you refactor, causing more work than being helpful and whose idea was this stupid testing stuff anyways?

What do you do instead? Don't reflect your internal code structure within your unit tests. Test for observable behavior instead. Think about

> _"if I enter values `x` and `y`, will the result be `z`?"_

instead of

> _"if I enter `x` and `y`, will the method call class A first, then call class B and then return the result of class A plus the result of class B?"_

Private methods should generally be considered an implementation detail that's why you shouldn't even have the urge to test them.

I often hear opponents of unit testing (or <abbr title="Test-Driven Development">TDD</abbr>) arguing that writing unit tests becomes pointless work where you have to test all your methods in order to come up with a high test coverage. They often cite scenarios where an overly eager team lead forced them to write unit tests for getters and setters and all other sorts of trivial code in order to come up with 100% test coverage.

There's so much wrong with that.

Yes, you should _test the public interface_. More importantly, however, you **don't test trivial code**. You won't gain anything from testing simple _getters_ or _setters_ or other trivial implementations (e.g. without any conditional logic). Save the time, that's one more meeting you can attend, hooray! Don't worry, [Kent Beck said it's ok](https://stackoverflow.com/questions/153234/how-deep-are-your-unit-tests/).

## But I _Really_ Need to Test This Private Method
If you ever find yourself in a situation where you _really really_ need to test a private method you should take a step back and ask yourself why.

I'm pretty sure this is more of a design problem than a scoping problem. Most likely you feel the need to test a private method because it's complex and testing this method through the public interface of the class requires a lot of awkward setup.

Whenever I find myself in this situation I usually come to the conclusion that the class I'm testing is already too complex. It's doing too much and violates the _single responsibility_ principle -- the _S_ of the five [_SOLID_](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)) principles.

The solution that often works for me is to split the original class into two classes. It often only takes one or two minutes of thinking to find a good way to cut the one big class into two smaller classes with individual responsibility. I move the private method (that I urgently want to test) to the new class and let the old class call the new method. Voilà, my awkward-to-test private method is now public and can be tested easily. On top of that I have improved the structure of my code by adhering to the single responsibility principle.

## Unit Testing is Not Enough
A good unit test suite will be immensely helpful during development: You know that all the small units you tested are working correctly in isolation. Your small-scoped unit tests help you narrowing down and reproducing errors in your code. On top they give you fast feedback while working with the codebase and will tell you whether you broke something unintendedly. Consider them as a tool _for developers_ as they are written from the developer's point of view and make their job easier.

Unfortunately writing unit alone won't get you very far. With unit tests you don't know whether your application as a whole works as intended. You don't know whether the features your customers love actually work. You don't know if you did a proper job plumbing and wiring all those components, classes and modules together.

Maybe there's something funky happening once all your small units join forces and work together as a bigger system. Maybe your code works perfectly fine when running against a mocked database but fails when it's supposed to write data to a real database. And maybe you wrote perfectly elegant and well-crafted code that totally fails to solve your users problem. Seems like we need more in order to spot these problems.
