# The Test Pyramid
If you want to get serious about automated tests for your software there is one key concept you should know about: the **test pyramid**. Mike Cohn came up with this concept in his book [Succeeding with Agile](https://www.amazon.com/dp/0321579364/ref=cm_sw_r_cp_dp_T2_bbyqzbMSHAG05). It's a great visual metaphor telling you to think about different layers of testing. It also tells you how much testing to do on each layer.

![Test Pyramid](img/testPyramid.png)
_The test pyramid_

Mike Cohn's original test pyramid consists of three layers that your test suite should consist of (bottom to top):

  1. Unit Tests
  2. Service Tests
  3. User Interface Tests

Unfortunately the concept of the test pyramid falls a little short if you take a closer look. [Some argue](https://watirmelon.blog/2011/06/10/yet-another-software-testing-pyramid/) that either the naming or some conceptual aspects of Mike Cohn's test pyramid are not optimal, and I have to agree. From a modern point of view the test pyramid seems overly simplistic and can therefore be a bit misleading.

Still, due to it's simplicity the essence of the test pyramid serves as a good rule of thumb when it comes to establishing your own test suite. Your best bet is to remember two things from Cohn's original test pyramid:

  1. Write tests with different granularity
  2. The more high-level you get the fewer tests you should have

Stick to the pyramid shape to come up with a healthy, fast and maintainable test suite: Write _lots_ of small and fast _unit tests_. Write _some_ more coarse-grained tests and _very few_ high-level tests that test your application from end to end. Watch out that you don't end up with a [test ice-cream cone](https://watirmelon.blog/2012/01/31/introducing-the-software-testing-ice-cream-cone/) that will be a nightmare to maintain and takes way too long to run.

Don't become too attached to the names of the individual layers in Cohn's test pyramid. In fact they can be quite misleading: _service test_ is a term that is hard to grasp (Cohn himself talks about the observation that [a lot of developers completely ignore this layer](https://www.mountaingoatsoftware.com/blog/the-forgotten-layer-of-the-test-automation-pyramid)). In the days of modern single page application frameworks like react, angular, ember.js and others it becomes apparent that _UI tests_ don't have to be on the highest level of your pyramid -- you're perfectly able to unit test your UI in all of these frameworks.

Given the shortcomings of the original names it's totally okay to come up with other names for your test layers, as long as you keep it consistent within your codebase and your team's discussions.

## Moving On
While the test pyramid suggests that you'll have three different types of tests (_unit tests_, _service tests_ and _UI tests_) I need to disappoint you. Your reality will look a little more diverse. Lets keep Cohn's test pyramid in mind for its good things (use test layers with different granularity, make sure they're differently sized) and find out what types of tests we need for an effective test suite.
