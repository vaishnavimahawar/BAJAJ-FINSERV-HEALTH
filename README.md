# BAJAJ-FINSERV-HEALTH
Qualifier 1
# BAJAJ-FINSERV- SPRINGBOOT APPLICATION
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Component;
import org.springframework.web.reactive.function.client.WebClient;

import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Mono;

@SpringBootApplication
@Slf4j
public class BajajFinservHealthApplication {

    public static void main(String[] args) {
        SpringApplication.run(BajajFinservHealthApplication.class, args);
    }
}

/**
 * A data class to represent the request body for the webhook generation API.
 */
@Data
class WebhookRequest {
    private String name;
    private String regNo;
    private String email;
}

/**
 * A data class to represent the response body from the webhook generation API.
 */
@Data
class WebhookResponse {
    private String webhook;
    private String accessToken;
}

/**
 * A data class to represent the request body for submitting the final query.
 */
@Data
class SolutionRequest {
    private String finalQuery;
}

/**
 * This component runs on application startup to handle the API calls and logic.
 * It implements ApplicationRunner to execute logic after the Spring Boot application context has started.
 */
@Component
@Slf4j
class WebhookRunner implements ApplicationRunner {

    private static final String GENERATE_WEBHOOK_URL = "https://bfhldevapigw.healthrx.co.in/hiring/generateWebhook/JAVA";
    private static final String SUBMIT_SOLUTION_URL = "https://bfhldevapigw.healthrx.co.in/hiring/testWebhook/JAVA";
    private final WebClient webClient;

    public WebhookRunner(WebClient.Builder webClientBuilder) {
        // Build a WebClient instance with a base URL
        this.webClient = webClientBuilder.baseUrl(GENERATE_WEBHOOK_URL).build();
    }

    @Override
    public void run(ApplicationArguments args) {
        log.info("Starting the application startup task...");

        // Step 1: Prepare the request body for webhook generation
        // REPLACE THE FOLLOWING VALUES with your actual details
        WebhookRequest requestBody = new WebhookRequest();
        requestBody.setName("John Doe");
        requestBody.setRegNo("REG12347");
        requestBody.setEmail("john@example.com");

        // Step 2: Send the POST request to generate the webhook
        Mono<WebhookResponse> webhookResponseMono = webClient.post()
            .contentType(MediaType.APPLICATION_JSON)
            .body(Mono.just(requestBody), WebhookRequest.class)
            .retrieve()
            .bodyToMono(WebhookResponse.class);

        // Step 3: Process the response and submit the solution
        webhookResponseMono.subscribe(
            response -> {
                log.info("Successfully received webhook and access token.");
                String webhookUrl = response.getWebhook();
                String accessToken = response.getAccessToken();

                // Determine the question based on the last two digits of the regNo
                String regNo = requestBody.getRegNo();
                String lastTwoDigits = regNo.substring(regNo.length() - 2);
                int numericValue = Integer.parseInt(lastTwoDigits);
                String finalQuery;

                if (numericValue % 2 != 0) {
                    // Question 1 for Odd regNo
                    log.info("RegNo ends in an odd number. Preparing for Question 1.");
                    // TODO: Replace the placeholder below with the actual SQL query for Question 1
                    finalQuery = "YOUR_SQL_QUERY_FOR_ODD_REGNO_HERE";
                } else {
                    // Question 2 for Even regNo
                    log.info("RegNo ends in an even number. Preparing for Question 2.");
                    // TODO: Replace the placeholder below with the actual SQL query for Question 2
                    finalQuery = "YOUR_SQL_QUERY_FOR_EVEN_REGNO_HERE";
                }

                // Step 4: Prepare the request body for submitting the solution
                SolutionRequest solutionBody = new SolutionRequest();
                solutionBody.setFinalQuery(finalQuery);

                // Step 5: Send the final query to the webhook URL with JWT token
                WebClient solutionClient = WebClient.builder().build();
                solutionClient.post()
                    .uri(webhookUrl)
                    .header(HttpHeaders.AUTHORIZATION, "Bearer " + accessToken)
                    .contentType(MediaType.APPLICATION_JSON)
                    .body(Mono.just(solutionBody), SolutionRequest.class)
                    .retrieve()
                    .bodyToMono(String.class)
                    .subscribe(
                        solutionResponse -> log.info("Solution submitted successfully. Response: {}", solutionResponse),
                        error -> log.error("Error submitting solution: {}", error.getMessage())
                    );
            },
            error -> log.error("Error generating webhook: {}", error.getMessage())
        );
    }
}
