# HTTP-calls-using-flux-and-mono
This section shows WebClient usage for making api calls using flux and mono


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

