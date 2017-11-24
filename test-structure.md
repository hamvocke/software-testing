# Test Structure
A good structure for all your tests (this is not limited to unit tests) is this one:

  1. Set up the test data
  2. Call your method under test
  3. Assert that the expected results are returned

There's a nice mnemonic to remember this structure: [_"Arrange, Act, Assert"_](http://wiki.c2.com/?ArrangeActAssert). Another one that you can use takes inspiration from <abbr title="Behavior-Driven Development">BDD</abbr>. It's the _"given"_, _"when"_, _"then"_ triad, where _given_ reflects the setup, _when_ the method call and _then_ the assertion part.

This pattern can be applied to other, more high-level tests as well. In every case they ensure that your tests remain easy and consistent to read. On top of that tests written with this structure in mind tend to be shorter and more expressive.
