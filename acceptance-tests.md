# Acceptance Tests -- Do Your Features Work Correctly?
The higher you move up in your test pyramid the more likely you enter the realms of testing whether the features you're building work correctly from a user's perspective. You can treat your application as a black box and shift  the focus in your tests from

> when I enter the values `x` and `y`, the return value should be `z`

towards

> _given_ there's a logged in user  
> _and_ there's an article "bicycle"  
> _when_ the user navigates to the "bicycle" article's detail page  
> _and_ clicks the "add to basket" button  
> _then_ the article "bicycle" should be in their shopping basket

Sometimes you'll hear the terms [**functional test**](https://en.wikipedia.org/wiki/Functional_testing) or [**acceptance test**](https://en.wikipedia.org/wiki/Acceptance_testing#Acceptance_testing_in_extreme_programming) for these kinds of tests. Sometimes people will tell you that functional and acceptance tests are different things. Sometimes the terms are conflated. Sometimes people will argue endlessly about wording and definitions. Often this discussion is a pretty big source of confusion.

Here's the thing: At one point you should make sure to test that your software works correctly from a _user's_ perspective, not just from a technical perspective. What you call these tests is really not that important. Having these tests, however, is. Pick a term, stick to it, and write those tests.

This is also the moment where people talk about <abbr title="Behaviour-Driven Development">BDD</abbr> and tools that allow you to implement tests in a BDD fashion. BDD or a BDD-style way of wrtiting tests can be a nice trick to shift your mindset from implementation details towards the users' needs. Go ahead and give it a try.

You don't even need to adopt full-blown BDD tools like [Cucumber](https://cucumber.io/) (though you can). Some assertion libraries (like [chai.js](http://chaijs.com/guide/styles/#should) allow you to write assertions with `should`-style keywords that can make your tests read more BDD-like. And even if you don't use a library that provides this notation, clever and well-factored code will allow you to write user behaviour focused tests. Some helper methods/functions can get you a very long way:

```java
def test_add_to_basket():
    # given
    user = a_user_with_empty_basket()
    user.login()
    bicycle = article(name="bicycle", price=100)

    # when
    article_page.add_to_.basket(bicycle)

    # then
    assert user.basket.contains(bicycle)
```

Acceptance tests can come in different levels of granularity. Most of the time they will be rather high-level and test your service through the user interface. However, it's good to understand that there's technically no need to write acceptance tests at the highest level of your test pyramid. If your application design and your scenario at hand permits that you write an acceptance test at a lower level, go for it. Having a low-level test is better than having a high-level test. The concept of acceptance tests -- proving that your features work correctly for the user -- is completely orthogonal to your test pyramid.
