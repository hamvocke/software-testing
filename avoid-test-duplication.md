# Avoid Test Duplication
Now that you know that you should write different types of tests there's one more pitfall to avoid: test duplication. While your gut feeling might say that there's no such thing as too many tests let me assure you, there is. Every single test in your test suite is additional baggage and doesn't come for free. Writing and maintaining tests takes time. Reading and understanding other people's test takes time. And of course, running tests takes time.

As with production code you should strive for simplicity and avoid duplication. If you managed to test all of your code's edge cases on a unit level there's no need to test these edge cases again on a higher-level. Keep this as a rule of thumb.

If your high-level test adds additional value (e.g. testing the integration with a real database) than it's a good idea to have this higher level test even though you might have tested the same database access function in a unit test. Just make sure to focus on the integration part in that test and avoid going through all possible edge-cases again.

Duplicating tests can be quite tempting, especially when you're new to test automation. Be aware of the additional cost and don't be afraid to delete tests if you were able to replace them with lower level tests or if they no longer provide any value.
