# Integration Tests
All non-trivial applications will integrate with some other parts (databases, filesystems, network calls to other applications). When writing unit tests these are usually the parts you leave out in order to come up with better isolation and fast tests. Still, your application will interact with other parts and this needs to be tested. _Integration tests_ are there to help. They test the integration of your application with all the parts that live outside of your application.

For your automated tests this means you don't just need to run your own application but also the component you're integrating with. If you're testing the integration with a database you need to run a database when running your tests. For testing that you can read files from a disk you need to save a file to your disk and use it as load it in your integration test.

Integration tests live at the boundary of your service. Conceptually they're always about triggering an action that leads to integrating with the outside part (filesystem, database, etc). A database integration test would probably look like this:

![a database integration test](img/dbIntegrationTest.png)
*A database integration test integrates your code with a real database*

    1. start a database
    2. connect your application to the database
    3. trigger a function within your code that writes data to the database
    4. check that the expected data has been written to the database by reading the data from the database


Another example, an integration test for your REST API could look like this:

![an HTTP integration test](img/httpIntegrationTest.png)
*An HTTP integration test checks that real HTTP calls hit your code correctly*

    1. start your application
    2. fire an HTTP request against one of your REST endpoints
    3. check that the desired interaction has been triggered within your application

Your integration tests -- like unit tests -- can be fairly whitebox. Some frameworks allow you to start your application while still being able to mock some other parts of your application so that you can check that the correct interactions have happened.

Write integration tests for all pieces of code where you either _serialize_ or _deserialize_ data. This happens more often than you might think. Think about:

  * Calls to your services' REST API
  * Reading from and writing to databases
  * Calling other application's APIs
  * Reading from and writing to queues
  * Writing to the filesystem

Writing integration tests around these boundaries ensures that writing data to and reading data from these external collaborators works fine.

If possible you should prefer to run your external dependencies locally: spin up a local MySQL database, test against a local ext4 filesystem. In some cases this won't be easy. If you're integrating with third-party systems from another vendor you might not have the option to run an instance of that service locally (though you should try; talk to your vendor and try to find a way).

If there's no way to run a third-party service locally you should opt for running a dedicated test instance somewhere and point at this test instance when running your integration tests. Avoid integrating with the real production system in your automated tests. Blasting thousands of test requests against a production system is a surefire way to get people angry because you're cluttering their logs (in the best case) or even <abbr title="Denial of Service">DoS</abbr>'ing their service (in the worst case).

With regards to the test pyramid, integration tests are on a higher level than your unit tests. Integrating slow parts like filesystems and databases tends to be much slower than running unit tests with these parts stubbed out. They can also be harder to write than small and isolated unit tests, after all you have to take care of spinning up an external part as part of your tests. Still, they have the advantage of giving you the confidence that your application can correctly work with all the external parts it needs to talk to. Unit tests can't help you with that.
