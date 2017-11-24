# REST API Integration [Java]
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
