# Some Advice Before You Leave

There we go, you made it through the entire testing pyramid. Congratulations! Before you go, there are some more general pieces of advice that I think will be helpful on your journey. Keep these in mind and you'll soon write automated tests that truly kick ass:

1. Test code is as important as production code. Give it the same level of care and attention. Never allow sloppy code to be justified with the _"this is only test code"_ claim
2. Test one condition per test. This helps you to keep your tests short and easy to reason about
3. _"arrange, act, assert"_ or _"given, when, then"_ are good mnemonics to keep your tests well-structured
4. Readability matters. Don't try to be overly <abbr title="Don't Repeat Yourself">DRY</abbr>. Duplication is okay, if it improves readability. Try to find a balance between [DRY and <abbr title="Descriptive and Meaningful Phrases">DAMP</abbr>](https://stackoverflow.com/questions/6453235/what-does-damp-not-dry-mean-when-talking-about-unit-tests) code
5. When in doubt use the [Rule of Three](https://blog.codinghorror.com/rule-of-three/) to decide when to refactor. _Use before reuse_.
