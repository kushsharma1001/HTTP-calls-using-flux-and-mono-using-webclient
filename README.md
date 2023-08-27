# WebClient: HTTP-calls-using-flux-and-mono and Retry using Webclient
This section shows WebClient usage for making api calls using flux and mono.
Retry mechanism using webclient can be read about here: https://ajinkyakarode24.medium.com/spring-boot-retry-mechanism-using-webclient-http-client-40c4d23e0b7e <br/>


public String getAccessToken(String baseUrl, String clientId, String clientSecret) {

        WebClient webClient = WebClient.builder()
                .baseUrl(baseUrl)
                .build();

        Mono<OAuthResponseDetail> oAuthResponseDetailMono = webClient.post()
                .uri("/oauth/token")
                .headers(headers -> headers.setBasicAuth(clientId, clientSecret))
                .body(BodyInserters.fromFormData("grant_type", "client_credentials"))
                .exchange()
                .flatMap(response -> {
                    if (response.statusCode()
                            .is2xxSuccessful()) {
                        return response.bodyToMono(OAuthResponseDetail.class);
                    } else {
                        throw new CleanupException("Error fetching access token");
                    }
                });
        return oAuthResponseDetailMono.block().getAccessToken();
    }
    
    
    public String getTaskId(String taskName, String accessToken) {
    WebClient webClient = WebClient.builder()
                .baseUrl(trmRouteUrl)
                .build();

        String subDomainName = subAccountBase.getSubdomainId();
        String taskId = null;
        String trmDeprovisionRoute = "/api/user/v1/info/details/";

        TrmTenantDetails tenantDetails = new TrmTenantDetails();
        tenantDetails.setTimeOutMs(20000L);
        tenantDetails.setTaskName(taskName);

        Mono<TRMResponseHandler> trmResponse = webClient.method(HttpMethod.DELETE)
                .uri(trmDeprovisionRoute + taskName)
                .headers(headers -> {
                    headers.setBearerAuth(accessToken);
                    headers.setContentType(MediaType.APPLICATION_JSON);
                })
                .body(Mono.just(tenantDetails), TrmTenantDetails.class)
                .retrieve()
                .bodyToMono(TRMResponseHandler.class);

        taskId = trmResponse.block()
                .getTaskId();
                
                }

https://www.baeldung.com/spring-webflux <br/>

Another simple comparision on how restTemplate and webCLient works:

@RestController
@RequestMapping("/serv1")
@CrossOrigin("*")
public class Service1Controller {
	@Autowired
	RestTemplate restTemplate;

	@GetMapping(value = "/restTemplate")
	public ResponseEntity<?> getDataFromService2RestTemplate() {
		System.out.println("Service1 Controller Called");
		ResponseEntity<String> responseEntity = restTemplate.getForEntity("http://localhost:8081/serv2", String.class);
		return ResponseEntity.ok().body("Recieved Message: " + responseEntity.getBody());
	}

	@GetMapping(value = "/webclient")
	public ResponseEntity<?> getDataFromService2WebClient() {
		Mono<String> message = WebClient.create("http://localhost:8081").get().uri(uriBuilder -> uriBuilder.path("/serv2").build()).retrieve()
				.bodyToMono(String.class);
		return ResponseEntity.ok().body("Recieved Message: " + message.block());
	}
}
