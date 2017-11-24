# Provider Test Example (the other team)
The provider test has to be implemented by the people providing the weather API. We're consuming a public API provided by darksky.net. In theory the darksky team would implement the provider test on their end to check that they're not breaking the contract between their application and our service.

Obviously they don't care about our meager sample application and won't implement a CDC test for us. That's the big difference between a public-facing API and an organisation adopting microservices. Public-facing APIs can't consider every single consumer out there or they'd become unable to move forward. Within your own organisation, you can -- and should. Your app will most likely serve a handful, maybe a couple dozen of consumers max. You'll be fine writing provider tests for these interfaces in order to keep a stable system.

The providing team gets the pact file and runs it against their providing service. To do so they implement a provider test that reads the pact file, stubs out some test data and runs the expectations defined in the pact file against their service.

The pact folks have written several libraries for implementing provider tests. Their main [GitHub repo](https://github.com/DiUS/pact-jvm) gives you a nice overview which consumer and which provider libraries are available. Pick the one that best matches your tech stack.

For simplicity let's assume that the darksky API is implemented in Spring Boot as well. In this case they could use the [Spring pact provider](https://github.com/DiUS/pact-jvm/tree/master/pact-jvm-provider-spring) which hooks nicely into Spring's MockMVC mechanisms. A hypothetical provider test that the darksky.net team would implement could look like this:

```java
@RunWith(RestPactRunner.class)
@Provider("weather_provider") // same as in the "provider_name" part in our clientConsumerTest
@PactFolder("target/pacts") // tells pact where to load the pact files from
public class WeatherProviderTest {
    @InjectMocks
    private ForecastController forecastController = new ForecastController();

    @Mock
    private ForecastService forecastService;

    @TestTarget
    public final MockMvcTarget target = new MockMvcTarget();

    @Before
    public void before() {
        initMocks(this);
        target.setControllers(forecastController);
    }

    @State("weather forecast data") // same as the "given()" part in our clientConsumerTest
    public void weatherForecastData() {
        when(forecastService.fetchForecastFor(any(String.class), any(String.class)))
                .thenReturn(weatherForecast("Rain"));
    }
}
```

You see that all the provider test has to do is to load a pact file (e.g. by using the `@PactFolder` annotation to load previously downloaded pact files) and then define how test data for pre-defined states should be provided (e.g. using Mockito mocks). There's no custom test to be implemented. These are all derived from the pact file. It's important that the provider test has matching counterparts to the _provider name_ and _state_ declared in the consumer test.

I know that this whole CDC thing can be confusing as hell when you get started. Believe me when I say it's worth taking your time to understand it. If you need a more thorough example, go and check out the [fantastic example](https://github.com/lplotni/pact-example) my friend [Lukasz](https://twitter.com/lplotni) has written. This repo demonstrates how to write consumer and provider tests using pact. It even features both Java and JavaScript services so that you can see how easy it is to use this approach with different programming languages.
