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
                        throw new CleanupException("Error fetching xsuaa access token");
                    }
                });
        return oAuthResponseDetailMono.block().getAccessToken();
    }

