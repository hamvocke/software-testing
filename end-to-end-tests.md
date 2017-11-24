# End-to-End Tests
Testing your deployed application via its user interface is the most end-to-end way you could test your application. The previously described, webdriver driven UI tests are a good example of end-to-end tests.

![an end-to-end test](img/e2etests.png)
_End-to-end tests test your entire, completely integrated system_

End-to-end tests give you the biggest confidence when you need to decide if your software is working or not. [Selenium](http://docs.seleniumhq.org/) and the [WebDriver Protocol](https://www.w3.org/TR/webdriver/) allow you to automate your tests by automatically driving a (headless) browser against your deployed services, performing clicks, entering data and checking the state of your user interface. You can use Selenium directly or use tools that are build on top of it, [Nightwatch](http://nightwatchjs.org/) being one of them.

End-to-End tests come with their own kind of problems. They are notoriously flaky and often fail for unexpected and unforseeable reasons. Quite often their failure is a false positive. The more sophisticated your user interface, the more flaky the tests tend to become. Browser quirks, timing issues, animations and unexpected popup dialogs are only some of the reasons that got me spending more of my time with debugging than I'd like to admit.

In a microservices world there's also the big question of who's in charge of writing these tests. Since they span multiple services (your entire system) there's no single team responsible for writing end-to-end tests.

If you have a centralised _quality assurance_ team they look like a good fit. Then again having a centralised QA team is a big anti-pattern and shouldn't have a place in a DevOps world where your teams are meant to be truly cross-functional. There's no easy answer who should own end-to-end tests. Maybe your organisation has a community of practice or a _quality guild_ that can take care of these. Finding the correct answer highly depends on your organisation.

Furthermore, end-to-end tests require a lot of maintenance and run pretty slowly. Thinking about a landscape with more than a couple of microservices in place you won't even be able to run your end-to-end tests locally -- as this would require to start all your microservices locally as well. Good luck spinning up hundreds of applications on your development machine without frying your RAM.

Due to their high maintenance cost you should aim to reduce the number of end-to-end tests to a bare minimum.

Think about the high-value interactions users will have with your application. Try to come up with user journeys that define the core value of your product and translate the most important steps of these user journeys into automated end-to-end tests.

If you're building an e-commerce site your most valuable customer journey could be a user searching for a product, putting it in the shopping basket and doing a checkout. That's it. As long as this journey still works you shouldn't be in too much trouble. Maybe you'll find one or two more crucial user journeys that you can translate into end-to-end tests. Everything more than that will likely be more painful than helpful.

Remember: you have lots of lower levels in your test pyramid where you already tested all sorts of edge cases and integrations with other parts of the system. There's no need to repeat these tests on a higher level. High maintenance effort and lots of false positives will slow you don't and make sure you'll lose trust in your tests rather sooner than later.


## End-to-End Test with Selenium Example (testing via the UI) [Java]
For end-to-end tests [Selenium](http://docs.seleniumhq.org/) and the [WebDriver](https://www.w3.org/TR/webdriver/) protocol are the tool of choice for many developers. With Selenium you can pick a browser you like and let it automatically call your website, click here and there, enter data and check that stuff changes in the user interface.

Selenium needs a browser that it can start and use for running its tests. There are multiple so-called _'drivers'_ for different browsers that you could use. [Pick one](https://www.mvnrepository.com/search?q=selenium+driver) (or multiple) and add it to your `build.gradle`. Whatever browser you choose, you need to make sure that all devs in your team and your CI server have installed the correct version of the browser locally. This can be pretty painful to keep in sync. For Java, there's a nice little library called [webdrivermanager](https://github.com/bonigarcia/webdrivermanager) that can automate downloading and setting up the correct version of the browser you want to use. Add these two dependencies to your `build.gradle` and you're good to go:

    testCompile('org.seleniumhq.selenium:selenium-chrome-driver:2.53.1')
    testCompile('io.github.bonigarcia:webdrivermanager:1.7.2')

Running a fully-fledged browser in your test suite can be a hassle. Especially when using continuous delivery the server running your pipeline might not be able to spin up a browser including a user interface (e.g. because there's no X-Server available). You can take a workaround for this problem by starting a virtual X-Server like [xvfb](https://en.wikipedia.org/wiki/Xvfb).

A more recent approach is to use a _headless_ browser (i.e. a browser that doesn't have a user interface) to run your webdriver tests. Until recently [PhantomJS](http://phantomjs.org/) was the leading headless browser used for browser automation. Ever since both [Chromium](https://developers.google.com/web/updates/2017/04/headless-chrome) and [Firefox](https://developer.mozilla.org/en-US/Firefox/Headless_mode) announced that they've implemented a headless mode in their browsers PhantomJS all of a sudden became obsolete. After all it's better to test your website with a browser that your users actually use (like Firefox and Chrome) instead of using an artificial browser just because it's convenient for you as a developer.

Both, headless Firefox and Chrome, are brand new and yet to be widely adopted for implementing webdriver tests. We want to keep things simple. Instead of fiddling around to use the bleeding edge headless modes let's stick to the classic way using Selenium and a regular browser. A simple end-to-end test that fires up Chrome, navigates to our service and checks the content of the website looks like this:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class HelloE2ESeleniumTest {

    private WebDriver driver;

    @LocalServerPort
    private int port;

    @BeforeClass
    public static void setUpClass() throws Exception {
        ChromeDriverManager.getInstance().setup();
    }

    @Before
    public void setUp() throws Exception {
        driver = new ChromeDriver();
    }

    @After
    public void tearDown() {
        driver.close();
    }

    @Test
    public void helloPageHasTextHelloWorld() {
        driver.get(String.format("http://127.0.0.1:%s/hello", port));

        assertThat(driver.findElement(By.tagName("body")).getText(), containsString("Hello World!"));
    }
}
```

Note that this test will only run on your system if you have Chrome installed on the system you run this test on (your local machine, your CI server).

The test is straightforward. It spins up the entire Spring application on a random port using `@SpringBootTest`. We then instanciate a new Chrome webdriver, tell it to go navigate to the `/hello` endpoint of our microservice and check that it prints "Hello World!" on the browser window. Cool stuff!

## End-to-End Test Example Using RestAssured (Testing via the REST API)
I know, we already have tests in place that fire some sort of request against our REST API and check that the results are correct. Still, none of them is truly end to end. The MockMVC tests are "only" integration tests and don't send real HTTP requests against a fully running service.

Let me show you one last tool that can come in handy when you write a service that provides a REST API. [REST-assured](https://github.com/rest-assured/rest-assured) is a library that gives you a nice DSL for firing real HTTP requests against an API and checks the responses. It looks similar to MockMVC but is truly end-to-end (fun fact: there's even a REST-Assured MockMVC dialect). If you think Selenium is overkill for your application as you don't really have a user interface that needs testing, REST-Assured is the way to go.

First things first: Add the dependency to your `build.gradle`.

    testCompile('io.rest-assured:rest-assured:3.0.3')

With this library at our hands we can implement a end-to-end test for our REST API:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class HelloE2ERestTest {

    @Autowired
    private PersonRepository personRepository;

    @LocalServerPort
    private int port;

    @After
    public void tearDown() throws Exception {
        personRepository.deleteAll();
    }

    @Test
    public void shouldReturnGreeting() throws Exception {
        Person peter = new Person("Peter", "Pan");
        personRepository.save(peter);

        when()
                .get(String.format("http://localhost:%s/hello/Pan", port))
        .then()
                .statusCode(is(200))
                .body(containsString("Hello Peter Pan!"));
    }
}
```

Again, we start the entire Spring application using `@SpringBootTest`. In this case we `@Autowire` the `PersonRepository` so that we can write test data into our database easily. When we now ask the REST API to say "hello" to our friend "Mr Pan" we're being presented with a nice greeting. Amazing! And more than enough of an end-to-end test if you don't even sport a web interface.
