## Consumer Test (our team)
Our microservice consumes the weather API. So it's our responsibility to write a **consumer test** that defines our expectations for the contract (the API) between our microservice and the weather service.

First we include a library for writing pact consumer tests in our `build.gradle`:

    testCompile('au.com.dius:pact-jvm-consumer-junit_2.11:3.5.5')

Thanks to this library we can implement a consumer test and use pact's mock services:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class WeatherClientConsumerTest {

    @Autowired
    private WeatherClient weatherClient;

    @Rule
    public PactProviderRuleMk2 weatherProvider = new PactProviderRuleMk2("weather_provider", "localhost", 8089, this);

    @Pact(consumer="test_consumer")
    public RequestResponsePact createPact(PactDslWithProvider builder) throws IOException {
        return builder
                .given("weather forecast data")
                .uponReceiving("a request for a weather request for Hamburg")
                    .path("/some-test-api-key/53.5511,9.9937")
                    .method("GET")
                .willRespondWith()
                    .status(200)
                    .body(FileLoader.read("classpath:weatherApiResponse.json"), ContentType.APPLICATION_JSON)
                .toPact();
    }

    @Test
    @PactVerification("weather_provider")
    public void shouldFetchWeatherInformation() throws Exception {
        Optional<WeatherResponse> weatherResponse = weatherClient.fetchWeather();
        assertThat(weatherResponse.isPresent(), is(true));
        assertThat(weatherResponse.get().getSummary(), is("Rain"));
    }
}
```

If you look closely, you'll see that the `WeatherClientConsumerTest` is very similar to the `WeatherClientIntegrationTest`. Instead of using Wiremock for the server stub we use Pact this time. In fact the consumer test works exactly as the integration test, we replace the real third-party server with a stub, define the expected response and check that our client can parse the response correctly. The difference is that the consumer test generates a **pact file** (found in `target/pacts/<pact-name>.json`) each time it runs. This pact file describes our expectations for the contract in a special JSON format.

You see that this is where the **consumer-driven** part of CDC comes from. The consumer drives the implementation of the interface by describing their expectations. The provider has to make sure that they fulfill all expectations and they're done. No gold-plating, no YAGNI and stuff.

We can take the pact file and hand it to the team providing the interface. They in turn can take this pact file and write a provider test using the expectations defined in there. This way they test if their API fulfills all our expectations.

Getting the pact file to the providing team can happen in multiple ways. A simple one is to check them into version control and tell the provider team to always fetch the latest version of the pact file. A more advances one is to use an artifact repository, a service like Amazon's S3 or the pact broker. Start simple and grow as you need.

In your real-world application you don't need both, an _integration test_ and a _consumer test_ for a client class. The sample codebase contains both to show you how to use either one. If you want to write CDC tests using pact I recommend sticking to the latter. The effort of writing the tests is the same. Using pact has the benefit that you automatically get a pact file with the expectations to the contract that other teams can use to easily implement their provider tests. Of course this only makes sense if you can convince the other team to use pact as well. If this doesn't work, using the _integration test_ and Wiremock combination is a decent plan b.
