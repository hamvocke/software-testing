# End-to-End Test via a REST API Using RestAssured
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
