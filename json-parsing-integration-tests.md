# Parsing and Writing JSON
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
