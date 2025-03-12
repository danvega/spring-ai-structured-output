# Structured Output in Spring AI

A demonstration project showcasing how to use Spring AI to retrieve structured data from Large Language Models (LLMs) in a Spring Boot application.

## Introduction

This project demonstrates one of Spring AI's most powerful features: the ability to convert free-form text responses from AI models into structured Java objects. By leveraging Spring AI's structured output capabilities, you can seamlessly integrate AI-generated content into your application's data flow without manual parsing.

The sample application creates a REST endpoint that returns a list of NBA teams as structured Java objects, fetched directly from an OpenAI model.

## Project Requirements

- Java 23
- Maven 3.6+
- Spring Boot 3.4.3
- Spring AI 1.0.0-M6
- OpenAI API Key

## Dependencies

This project relies on the following key dependencies:

- `spring-boot-starter-web`: For creating RESTful endpoints
- `spring-ai-openai-spring-boot-starter`: Spring AI's integration with OpenAI
- `spring-boot-starter-test`: For unit testing

The complete dependency list is available in the `pom.xml` file.

## Getting Started

### Prerequisites

Before running the application, make sure you have:

1. JDK 23 installed on your machine
2. Maven configured correctly
3. An OpenAI API key

### Configuration

The application requires an OpenAI API key to function. Set this up by adding your API key as an environment variable:

```bash
export OPENAI_API_KEY=your-api-key-here
```

The application is configured to use the `gpt-4o` model by default, as specified in `application.properties`:

```properties
spring.application.name=output
spring.ai.openai.api-key=${OPENAI_API_KEY}
spring.ai.openai.chat.options.model=gpt-4o
```

You can modify this configuration to use other OpenAI models if needed.

## How to Run the Application

### Using Maven

To run the application with Maven:

```bash
mvn spring-boot:run
```

### Using Java

Alternatively, you can build the JAR file and run it directly:

```bash
mvn clean package
java -jar target/output-0.0.1-SNAPSHOT.jar
```

Once running, the application will be available at `http://localhost:8080`.

## Using the API

The application exposes a single endpoint:

- GET `/teams` - Returns a list of NBA teams as JSON

Example response:

```json
[
  {
    "teamName": "Lakers",
    "city": "Los Angeles"
  },
  {
    "teamName": "Celtics",
    "city": "Boston"
  },
  // ... more teams
]
```

## Understanding the Code

### Domain Model

The application defines a simple record to represent an NBA team:

```java
public record NbaTeam(String teamName, String city) {
}
```

### Controller

The `TeamsController` handles the web requests and interacts with the AI model:

```java
@RestController
public class TeamsController {

    private final ChatClient chatClient;

    public TeamsController(ChatClient.Builder builder) {
        this.chatClient = builder.build();
    }

    @GetMapping("/teams")
    public List<NbaTeam> teams() {
        return chatClient.prompt()
                .user("Please name all of the teams in the NBA.")
                .call()
                .entity(new ParameterizedTypeReference<List<NbaTeam>>() {});
    }
}
```

### How Structured Output Works

The magic happens in the `entity()` method call. Spring AI automatically:

1. Sends your prompt to the LLM
2. Parses the response
3. Maps the results to your Java object
4. Returns a strongly-typed result

The `ParameterizedTypeReference` tells Spring AI what type of object to convert the response into - in this case, a list of `NbaTeam` objects.

## Advanced Techniques

### Custom Prompting

You can customize the prompt to get more specific information:

```java
@GetMapping("/easternConference")
public List<NbaTeam> easternTeams() {
    return chatClient.prompt()
            .user("Please name all teams in the NBA's Eastern Conference.")
            .call()
            .entity(new ParameterizedTypeReference<List<NbaTeam>>() {});
}
```

### Error Handling

Consider adding error handling to manage rate limits or AI service outages:

```java
try {
    return chatClient.prompt()
            .user("Please name all of the teams in the NBA.")
            .call()
            .entity(new ParameterizedTypeReference<List<NbaTeam>>() {});
} catch (Exception e) {
    log.error("Error fetching teams from AI", e);
    return fallbackTeamsList();
}
```

## Testing

The project includes basic setup for testing with `ApplicationTests.java`. For more comprehensive testing, consider:

1. Mocking the `ChatClient` to avoid API calls during tests
2. Creating integration tests with a test API key
3. Adding response validation to ensure data quality

## Conclusion

Spring AI's structured output capabilities offer a seamless way to integrate AI responses into your Spring applications. This approach eliminates the need for manual parsing and conversion of AI responses, making it easy to work with LLMs in a type-safe manner.

Feel free to explore this example, modify the code, and adapt it to your specific use cases. Spring AI opens up numerous possibilities for embedding AI capabilities directly into your Spring applications.

## Further Resources

- [Spring AI Documentation](https://docs.spring.io/spring-ai/reference/)
- [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [OpenAI API Documentation](https://platform.openai.com/docs/api-reference)
