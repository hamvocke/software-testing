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

## Database Integration (Java)
The `PersonRepository` is the only repository class in the codebase. It relies on _Spring Data_ and has no actual implementation. It just extends the `CrudRepository` interface and provides a single method header. The rest is Spring magic.

```java
public interface PersonRepository extends CrudRepository<Person, String> {
    Optional<Person> findByLastName(String lastName);
}
```

With the `CrudRepository` interface Spring Boot offers a fully functional CRUD repository with `findOne`, `findAll`, `save`, `update` and `delete` methods. Our custom method definition (`findByLastName()`) extends this basic functionality and gives us a way to fetch `Person`s by their last name. Spring Data analyses the return type of the method and its method name and checks the method name against a naming convention to figure out what it should do.

Although Spring Data does the heavy lifting of implementing database repositories I still wrote a database integration test. You might argue that this is _testing the framework_ and something that I should avoid as it's not our code that we're testing. Still, I believe having at least one integration test here is crucial. First it tests that our custom `findByLastName` method actually behaves as expected. Secondly it proves that our repository used Spring's magic correctly and can connect to the database.

To make it easier for you to run the tests on your machine (without having to install a PostgreSQL database) our test connects to an in-memory _H2_ database.

I've defined H2 as a test dependency in the `build.gradle` file. The `application.properties` in the test directory doesn't define any `spring.datasource` properties. This tells Spring Data to use an in-memory database. As it finds H2 on the classpath it simply uses H2 when running our tests.

When running the real application with the `int` profile (e.g. by setting `SPRING_PROFILES_ACTIVE=int` as environment variable) it connects to a PostgreSQL database as defined in the `application-int.properties`.

I know, that's an awful lot of Spring magic to know and understand. To get there, you'll have to sift through [a lot of documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-sql.html#boot-features-embedded-database-support). The resulting code is easy on the eye but hard to understand if you don't know the fine details of Spring.

On top of that going with an in-memory database is risky business. After all, our integration tests run against a different type of database than they would in production. Go ahead and decide for yourself if you prefer Spring magic and simple code over an explicit yet more verbose implementation.

Enough explanation already, here's a simple integration test that saves a Person to the database and finds it by its last name:

```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class PersonRepositoryIntegrationTest {
    @Autowired
    private PersonRepository subject;

    @After
    public void tearDown() throws Exception {
        subject.deleteAll();
    }

    @Test
    public void shouldSaveAndFetchPerson() throws Exception {
        Person peter = new Person("Peter", "Pan");
        subject.save(peter);

        Optional<Person> maybePeter = subject.findByLastName("Pan");

        assertThat(maybePeter, is(Optional.of(peter)));
    }
}
```

You can see that our integration test follows the same _arrange, act, assert_ structure as the unit tests. Told you that this was a universal concept!

## REST API Integration [Java]
Testing our microservice's REST API is quite simple. Of course we can write simple unit tests for all `Controller` classes and call the controller methods directly as a first measure. `Controller` classes should generally be quite straightforward and focus on request and response handling. Avoid putting business logic into controllers, that's none of their business (_best pun ever..._). This makes our unit tests straightforward (or even unnecessary, if it's too trivial).

As Controllers make heavy use of [Spring MVC's](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/mvc.html) annotations for defining endpoints, query parameters and so on we won't get very far with unit tests. We want to see if our API works as expected: Does it have the correct endpoints, interpret input parameters and answer with correct HTTP status codes and response bodies? To do so, we have to go beyond unit tests.

One way to test our API were to start up the entire Spring Boot service and fire real HTTP requests against our API. With this approach we were on the very top of our test pyramid. Luckily there's another, a little less end-to-end way.

Spring MVC comes with a nice testing utility we can use: With [MockMVC](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-testing-spring-boot-applications-testing-autoconfigured-mvc-tests)we can spin up a small slice of our spring application, use a <abbr title="Domain-Specific Language">DSL</abbr> to fire test requests at our API and check that the returned data is as expected.

Let's see how this works for the `/hello/<lastname>` endpoint `ExampleController`:

```java
@RestController
public class ExampleController {
    private final PersonRepository personRepository;

    // shortened for clarity

    @GetMapping("/hello/{lastName}")
    public String hello(@PathVariable final String lastName) {
        Optional<Person> foundPerson = personRepository.findByLastName(lastName);

        return foundPerson
             .map(person -> String.format("Hello %s %s!", person.getFirstName(), person.getLastName()))
             .orElse(String.format("Who is this '%s' you're talking about?", lastName));
    }
}
```

Our controller calls the `PersonRepository` in the `/hello/<lastname>` endpoint. For our tests we need to replace this repository class with a mock to avoid hitting a real database. Even though this is an integration test, we're testing the REST API integration, not the database integration. That's why we stub the database in this case. The controller integration test looks as follows:

```java
@RunWith(SpringRunner.class)
@WebMvcTest(controllers = ExampleController.class)
public class ExampleControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private PersonRepository personRepository;

    // shortened for clarity

    @Test
    public void shouldReturnFullName() throws Exception {
        Person peter = new Person("Peter", "Pan");
        given(personRepository.findByLastName("Pan")).willReturn(Optional.of(peter));

        mockMvc.perform(get("/hello/Pan"))
                .andExpect(content().string("Hello Peter Pan!"))
                .andExpect(status().is2xxSuccessful());
    }
}
```

I annotated the test class with `@WebMvcTest` to tell Spring which controller we're testing. This mechanism instructs Spring to only start the Rest API slice of our application. We won't hit any repositories so spinning them up and requiring a database to connect to would simply be wasteful.

Instead of relying on the real `PersonRepository` we replace it with a mock in our Spring context using the `@MockBean` annotation. This annotation replaces the annotated class with a Mockito mock globally, all classes that are `@Autowired` will only find the `@MockBean` in the Spring context and wire that one instead of a real one. In our test methods we can set the behaviour of these mocks exactly as we would in a unit test, it's a Mockito mock after all.

To use `MockMvc` we can simply `@Autowire` a MockMvc instance. In combination with the `@WebMvcTest` annotation this is all Spring needs to fire test requests against our controller and expect return values and HTTP status codes. The `MockMVC` DSL is quite powerful and gets you a long way. Fiddle around with it to see what else you can do.

## Integration With Third-Party Services [Java]
Our microservice talks to [darksky.net](https://darksky.net), a weather REST API. Of course we want to ensure that our service sends requests and parses the responses correctly.

We want to avoid hitting the real _darksky_ servers when running automated tests. Quota limits of our free plan is only part of the reason. The real reason is _decoupling_. Our tests should run independently of whatever the lovely people at darksky.net are doing. Even when your machine can't access the _darksky_ servers (e.g. when you're coding on the airplane again instead of enjoying being crammed into a tiny airplane seat) or the darksky servers are down for some reason.

We can avoid hitting the real _darksky_ servers by running our own, fake _darksky_ server while running our integration tests. This might sound like a huge task. Thanks to tools like [Wiremock](http://wiremock.org/) it's easy peasy. Watch this:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class WeatherClientIntegrationTest {

    @Autowired
    private WeatherClient subject;

    @Rule
    public WireMockRule wireMockRule = new WireMockRule(8089);

    @Test
    public void shouldCallWeatherService() throws Exception {
        wireMockRule.stubFor(get(urlPathEqualTo("/some-test-api-key/53.5511,9.9937"))
                .willReturn(aResponse()
                        .withBody(FileLoader.read("classpath:weatherApiResponse.json"))
                        .withHeader(CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
                        .withStatus(200)));

        Optional<WeatherResponse> weatherResponse = subject.fetchWeather();

        Optional<WeatherResponse> expectedResponse = Optional.of(new WeatherResponse("Rain"));
        assertThat(weatherResponse, is(expectedResponse));
    }
}
```

To use Wiremock we instanciate a `WireMockRule` on a fixed port (`8089`). Using the DSL we can set up the Wiremock server, define the endpoints it should listen on and set canned responses it should respond with.

Next we call the method we want to test, the one that calls the third-party service and check if the result is parsed correctly.

It's important to understand how the test knows that it should call the fake Wiremock server instead of the real _darksky_ API. The secret is in our `application.properties` file contained in `src/test/resources`. This is the properties file Spring loads when running tests. In this file we override configuration like API keys and URLs with values that are suitable for our testing purposes, e.g. calling the the fake Wiremock server instead of the real one:

    weather.url = http://localhost:8089

Note that the port defined here has to be the same we define when instanciating the `WireMockRule` in our test. Replacing the real weather API's URL with a fake one in our tests is made possible by injecting the URL in our `WeatherClient` class' constructor:

```java
@Autowired
public WeatherClient(final RestTemplate restTemplate,
                     @Value("${weather.url}") final String weatherServiceUrl,
                     @Value("${weather.api_key}") final String weatherServiceApiKey) {
    this.restTemplate = restTemplate;
    this.weatherServiceUrl = weatherServiceUrl;
    this.weatherServiceApiKey = weatherServiceApiKey;
}
```

This way we tell our `WeatherClient` to read the `weatherUrl` parameter's value from the `weather.url` property we define in our application properties.

## Parsing and Writing JSON
Writing a REST API these days you often pick JSON when it comes to sending your data over the wire. Using Spring there's no need to writing JSON by hand nor to write logic that transforms your objects into JSON (although you can do both if you feel like reinventing the wheel). Defining <abbr title="Plain Old Java Object">POJOs</abbr> that represent the JSON structure you want to parse from a request or send with a response is enough.

Spring and [Jackson](https://github.com/FasterXML/jackson) take care of everything else. With the help of Jackson, Spring automagically parses JSON into Java objects and vice versa. If you have good reasons you can use any other JSON mapper out there in your codebase. The advantage of Jackson is that it comes bundled with Spring Boot.

Spring often hides the parsing and converting to JSON part from you as a developer. If you define a method in a `RestController` that returns a POJO, Spring MVC will automatically convert that POJO to a JSON string and put it in the response body. With Spring's `RestTemplate` you get the same magic. Sending a request using `RestTemplate` you can provide a POJO class that should be used to parse the response. Again it's Jackson being used under the hood.

When we talk to the weather API we receive a JSON response. The `WeatherResponse` class is a POJO representation of that JSON structure including all the fields we care about (which is only `response.currently.summary`). Using the `@JsonIgnoreProperties` annotation with the `ignoreUnknown` parameter set to `true` on our POJO objects gives us a [tolerant reader](https://www.martinfowler.com/bliki/TolerantReader.html), an interface that is liberal in what data it accepts (following [Postel's Law](https://en.wikipedia.org/wiki/Robustness_principle)). This way there can be all kinds of silly stuff in the JSON response we receive from the weather API. As long as `response.currently.summary` is there, we're happy.

If you want to test-drive your Jackson Mapping take a look at the `WeatherResponseTest`. This one tests the conversion of JSON into a `WeatherResponse` object. Since this deserialization is the only conversion we do in the application there's no need to test if a `WeatherResponse` can be converted to JSON correctly. Using the approach outlined below it's very simple to test serialization as well, though.

```java
@Test
public void shouldDeserializeJson() throws Exception {
   String jsonResponse = FileLoader.read("classpath:weatherApiResponse.json");
   WeatherResponse expectedResponse = new WeatherResponse("Rain");

   WeatherResponse parsedResponse = new ObjectMapper().readValue(jsonResponse, WeatherResponse.class);

   assertThat(parsedResponse, is(expectedResponse));
}
```

In this test case I read a sample JSON response from a file and let Jackson parse this JSON response using `ObjectMapper.readValue()`. Then I compare the result of the conversion with an expected `WeatherResponse` to see if the conversion works as expected.

You can argue that this kind of test is rather a unit than an integration test. Nevertheless, this kind of test can be pretty valuable to make sure that your JSON serialization and deserialization works as expected. Having these tests in place allows you to keep the integration tests around your REST API and your client classes smaller as you don't need to check the entire JSON conversion again.
