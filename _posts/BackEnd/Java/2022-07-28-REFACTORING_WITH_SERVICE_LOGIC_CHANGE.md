---
layout: post
title: "[Code Refactoring] 서비스 로직 변경에 따른 Refactoring"
tags: [BackEnd JAVA]
---

## Intro

안녕하세요 **Noah**입니다

이번엔 제가 오늘 Refactoring 진행한 내용을 공유드리려 합니다.

기존에는 동영상을 관리하던 Brightcove라는 Solution과 콘텐츠를 관리하는 Contentstack이라는 Solution을 혼용하여 사용하였습니다.
때문에 API 서버에서는 두 서버 모두와 통신하기 위한 Rest API Manager 객체를 생성하였었습니다.<br/>

하지만 고객사 정책으로 Contentstack Solution은 배제되면서 Brightcove Solution 서버와만 통신하면 되는 상황이 되어 Refactoring을 진행한 경험을 공유하고자 합니다.<br/>
(서버에서 현재 발생하고 있던 1가지 이슈 해결 내용도 포함되어 있습니다.)
<br/><br/><br/><br/>

## Refactoring을 진행한 REST API 요청 관리 객체
Refactoring은 org.apache.http API에서 org.springframework.http API로 변경하는 작업을 진행하였습니다.<br/>
작업을 진행하게 된 이유는 org.apache.http를 사용하여 Brightcove Solution과 통신 시 charset을 설정하여도 한글이 깨지는 현상이 존재했기 때문입니다.<br/>
필자는 해당 이슈를 해결하면서도 어지러운 코드들을 깔끔하게 정리하기 위하여 API를 변경하기로 결정하였습니다.

- <strong style="color: #bb4177;">'기존 코드'</strong>
    ```java
    @Slf4j
    @Getter
    public class RestApiManager {
    
        private HttpRequestBase httpMethod;
        private String token;
    
        public RestApiManager(HttpRequestBase httpMethod, String solutionTypeCode){
            this.httpMethod = httpMethod;
            this.addToken();
        }
    
        /**
         * @apiNote Response 결과를 String형식으로 변환해주는 Method
         * @param client - CloseableHttpClient를 활용해 Request요청 및 Response 수신
         */
        private String getResponseString(CloseableHttpClient client) throws IOException {
            HttpResponse response = client.execute(this.httpMethod);
            HttpEntity entity = response.getEntity();
            String result;
            if(entity != null){
                result = EntityUtils.toString(entity);
            } else {
                result = "";
            }
            return result;
        }
    
    
        /**
         * @apiNote http Method를 활용해 실제 데이터 통신
         * @param data - Json형식으로 작성된 String 타입의 데이터를 parameter로 넘겨주면 됨
         */
        public String httpExecute(String data) {
            CloseableHttpClient client = HttpClientBuilder.create().build();
            if (data != null){
                try {
                    setExcuteData(data);
                } catch (UnsupportedEncodingException e){
                    Util.customLogger(ApiCode.ERR_8995, "Error 내용 : " + e.getMessage(), this.getClass().getName());
                }
            }
            try {
                return getResponseString(client);
            } catch (Exception ex) {
                log.error("HttpExecute Exception : " + ex);
                throw new CustomException(ApiCode.ERR_9995);
            }
        }
    
        /**
         * @apiNote Http Method에 맞춰 Data 맵핑해주는 Method
         * @param data - Mapping할 데이터
         */
        private void setExcuteData(String data) throws UnsupportedEncodingException {
            if(this.httpMethod instanceof HttpGet) {
            } else if(this.httpMethod instanceof HttpPost){
                ((HttpPost)this.httpMethod).setEntity(new StringEntity(data));
            } else if(this.httpMethod instanceof HttpPut){
                ((HttpPut)this.httpMethod).setEntity(new StringEntity(data));
            }
        }
    
    
        /**
         * @apiNote http GET method를 활용해 실제 데이터 통신
         * @since 2022.02.24
         * @deprecated httpExecute으로 통합하여 Deprecated
         */
        public String httpGetExecute(){
            CloseableHttpClient client = HttpClientBuilder.create().build();
            try {
                return getResponseString(client);
            } catch (Exception ex) {
                log.error("HttpGetExecute Exception : " + ex);
                throw new CustomException(ApiCode.ERR_9995);
            }
        }
    
        /**
         * @apiNote http POST method를 활용해 실제 데이터 통신
         * @param data - Json형식으로 작성된 String 타입의 데이터를 parameter로 넘겨주면 됨
         * @deprecated httpExecute으로 통합하여 Deprecated
         */
        public String httpPostExecute(String data) throws UnsupportedEncodingException {
            CloseableHttpClient client = HttpClientBuilder.create().build();
            HttpPost request = (HttpPost)this.httpMethod;
            request.setEntity(new StringEntity(data));
            try {
                return getResponseString(client);
            } catch (Exception ex) {
                log.error("HttpPostExecute Exception : " + ex);
                throw new CustomException(ApiCode.ERR_9995);
            }
        }
    
        /**
         * @apiNote http PUT method를 활용해 실제 데이터 통신
         * @param data - Json형식으로 작성된 String 타입의 데이터를 parameter로 넘겨주면 됨
         * @return HttpResponse
         * @deprecated httpExecute으로 통합하여 Deprecated
         */
        @Deprecated
        public String httpPutExecute(String data) throws UnsupportedEncodingException {
            CloseableHttpClient client = HttpClientBuilder.create().build();
            HttpPut request = (HttpPut)this.httpMethod;
            request.setEntity(new StringEntity(data));
            try {
                return getResponseString(client);
            } catch (Exception ex) {
                log.error("HttpPutExecute Exception : " + ex);
                throw new CustomException(ApiCode.ERR_9995);
            }
        }
    
        /**
         * @apiNote 각 ENUM 타입별로 맞는 Token 조회 및 필드 내 적용
         */
        private void addToken() {
            this.setBrightcoveAccessToken();
            this.addTokenWithBearerToken();
        }
    
        /**
         * @apiNote X Api Key Header
         */
        public void setXApiKeyHeader() {
            this.addHeader("X-API-KEY", BrightcoveConfig.getXApiKey());
            this.addHeader("Content-Type", "application/json; charset=UTF-8");
        }
    
    
        /**
         * @apiNote 조회된 token값 적용
         */
        private void addTokenWithBearerToken(){
            if (null == this.token){
                log.error("전송할 Token이 없습니다.");
                return;
            }
            this.httpMethod.addHeader("Authorization", "Bearer " + this.token);
        }
    
        /**
         * @apiNote 조회된 token값 적용
         */
        private void addTokenWithNormalToken(){
            if (null == this.token){
                log.error("전송할 Token이 없습니다.");
                return;
            }
            this.httpMethod.addHeader("Authorization", this.token);
        }
    
        /**
         * @apiNote 필요에 따라 Header 내 값 추가
         * @param key - header에 사용할 key값
         * @param value - header에 사용할 value값
         */
        public void addHeader(String key, String value){
            this.httpMethod.addHeader(key, value);
        }
    
        /**
         * @apiNote Brightcove Restful API에 Request 요청 전달 시 필요한 Token 발행
         *          Brightcove 측으로 부터 받은 가이드 활용하여 제작
         */
        private void setBrightcoveAccessToken(){
            // Request 요청 전 필요한 값 세팅
            CredentialsProvider credsProvider = new BasicCredentialsProvider();
            credsProvider.setCredentials(AuthScope.ANY, new UsernamePasswordCredentials(BrightcoveConfig.getOAuthClientId(), BrightcoveConfig.getoAuthClientSecret()));
            AuthCache authCache = new BasicAuthCache();
            authCache.put(new HttpHost("oauth.brightcove.com", 443, "https"), new BasicScheme());
            HttpClientContext context = HttpClientContext.create();
            context.setCredentialsProvider(credsProvider);
            context.setAuthCache(authCache);
    
            try {
                // Request 요청
                HttpPost request = new HttpPost(BrightcoveConfig.getAccessTokenUri());
                ArrayList<NameValuePair> postParameters = new ArrayList<>();
                postParameters.add(new BasicNameValuePair("grant_type", "client_credentials"));
                request.setEntity(new UrlEncodedFormEntity(postParameters));
                CloseableHttpClient client = HttpClientBuilder.create().build();
                HttpResponse response = client.execute(request, context);
    
                // Response 수신한 access_token 데이터 확인 후 token값 리턴
                HttpEntity entity = response.getEntity();
                JsonElement parser = JsonParser.parseString(EntityUtils.toString(entity));
    
                // 현 객체의 필드값에 조회한 값을 token으로 지정
                this.token = parser.getAsJsonObject().get("access_token").getAsString();
            } catch (UnsupportedEncodingException uee){
                log.error("uee : " + uee);
            } catch (ClientProtocolException cpe){
                log.error("cpe : " + cpe);
            } catch (IOException io){
                log.error("io : " + io);
            }
        }
    }
    ```
    위 내용을 보게 되면 Brightcove Solution 외에도 다른 솔루션들과 혼용하여 사용할 수 있도록 공통 유틸을 만들려는 시도를 한 것을 볼 수 있습니다. 
    하지만 결국 서비스 정책상 Brightcove Solution만 사용하기로 결정되면서 위와 같이 지저분한 코드는 필요 없게 되었습니다.
    <br/><br/>
    또한 header를 설정할 시 Content-Type을 application/json으로 설정하였음에도 한글이 깨지는 현상을 보고 json기본 설정이 utf-8로 설정되는 것은 알았으나 혹시 명시해야하는가 싶어 charset 설정도 한 것을 보실 수 있습니다.
    결론적으로는 계속 한글이 깨지는 현상을 목격하였고 결국 아래와 같은 코드로 변경하였습니다.
    <br/><br/><br/>
    
- <strong style="color: #bb4177;">'변경된 코드'</strong>
    API를 org.springframework.http로 변경하였으며 get, post, put, delete와 같은 Http 메서드별 요청이 사라진 것을 보실 수 있습니다.<br/>
    또한 작동 방식이 간결해져 별도의 주석 작업 또한 필요가 없어져 더 간결한 코드형태를 볼 수 있어 만족스럽습니다.
    
    ```java
    public class BrightcoveRestApiManager {
    private static final Gson gson = new GsonBuilder().setFieldNamingPolicy(FieldNamingPolicy.LOWER_CASE_WITH_UNDERSCORES).create();
    
        public JsonObject request(String url, String params, HttpMethod method){
            return gson.fromJson(getResponseData(url, getEntity(params), method).getBody(), JsonObject.class);
        }
    
        @NotNull
        private HttpEntity<String> getEntity(String params) {
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_JSON);
            headers.setBearerAuth(getAuthToken());
            headers.set("X-API-KEY", BrightcoveConfig.getXApiKey());
            return new HttpEntity<>(params, headers);
        }
    
        @Nullable
        private ResponseEntity<String> getResponseData(String url, HttpEntity<String> entity, HttpMethod method) {
            HttpComponentsClientHttpRequestFactory requestFactory = new HttpComponentsClientHttpRequestFactory();
            requestFactory.setConnectTimeout(100000);
            requestFactory.setReadTimeout(100000);
            RestTemplate restTemplate = new RestTemplate(requestFactory);
            restTemplate.getMessageConverters().add(0, new StringHttpMessageConverter(StandardCharsets.UTF_8));
    
            try {
                ResponseEntity<String> responseData = restTemplate.exchange(url, method, entity, String.class);
                return responseData;
            } catch (Exception e){
                Util.customLogger(ApiCode.ERR_9996, method.name() + " 요청 전달 중 Error : " + e.getMessage(), this.getClass().getName());
                throw new CustomException(ApiCode.ERR_9996);
            }
        }
        
        // Solution 가이드 문서대로 token호출
        private String getAuthToken(){
            CredentialsProvider credsProvider = new BasicCredentialsProvider();
            credsProvider.setCredentials(AuthScope.ANY, new UsernamePasswordCredentials(BrightcoveConfig.getOAuthClientId(), BrightcoveConfig.getoAuthClientSecret()));
            AuthCache authCache = new BasicAuthCache();
            authCache.put(new HttpHost("oauth.brightcove.com", 443, "https"), new BasicScheme());
            HttpClientContext context = HttpClientContext.create();
            context.setCredentialsProvider(credsProvider);
            context.setAuthCache(authCache);
    
            try {
                // Request 준비
                HttpPost request = new HttpPost(BrightcoveConfig.getAccessTokenUri());
                ArrayList<NameValuePair> postParameters = new ArrayList<>();
                postParameters.add(new BasicNameValuePair("grant_type", "client_credentials"));
                request.setEntity(new UrlEncodedFormEntity(postParameters));
                
                // Request 전달
                CloseableHttpClient client = HttpClientBuilder.create().build();
                HttpResponse response = client.execute(request, context);
                
                // Response 결과 처리
                org.apache.http.HttpEntity entity = response.getEntity();
                JsonElement parser = JsonParser.parseString(EntityUtils.toString(entity));
                return parser.getAsJsonObject().get("access_token").getAsString();
            } catch (UnsupportedEncodingException e){
                throw new CustomException(ApiCode.ERR_9996);
            } catch (ClientProtocolException e){
                throw new CustomException(ApiCode.ERR_9996);
            } catch (IOException e){
                throw new CustomException(ApiCode.ERR_9996);
            }
        }
    }
    ```
    Parameter를 받아야하는 부분에서 따로 Parameter를 전달할 요소가 따로 없다면 'params' 변수를 빈값("")으로 값을 전달하도록 규약을 만들었고 모든 요청은 request()라는 메서드를 통해 요청하도록 만들었습니다.
    (getAuthToken() Method는 기존 방식을 그대로 사용하고 있는 것을 확인하실 수 있을텐데 이는 해당 Solution 업체 공식 가이드라 해당 내용을 그대로 사용하고 있는 것입니다.)
<br/><br/><br/><br/>
    
## 관리 객체 변경 후 호출 방식의 변화
REST API 요청 관리 객체를 변경한 후로 Request를 전달하는 방식 또한 바뀌게 되었습니다.

- 기존 객체 호출부
    ```java
    RestApiManager restApiManager = new RestApiManager(new HttpPost(url), "02");
    restApiManager.setXApiKeyHeader();
    String responseString = restApiManager.httpExecute(requestData);
    ```
    기존 내용을 보면 요청할 Solution타입을 "02"로 설정 필요하며 사용할 헤더 정보도 여기서 설정이 필요합니다.<br/> 이 두가지가 모두 세팅되어야 요청 전달이 가능했습니다.

- 변경 후 호출부
    ```java
    BrightcoveRestApiManager brightcoveRestApiManager = new BrightcoveRestApiManager();
    JsonObject result = brightcoveRestApiManager.request(url, requestData, HttpMethod.PUT);
    ```
    Brightcove에 맞춰 Header 등 필요 데이터 설정은 이미 객체에서 끝났기 때문에 전달할 URL, Body에 담을 데이터, Http Method만 선택하시면 됩니다.

이와 같이 관리 객체를 변경함으로써 호출하는 코드도 더 간결해지고 request 요청을 보내는 목적에 맞게 파라미터도 세팅할 수 있게 되었습니다.
<br/><br/><br/><br/>

## 글을 마치며
프로젝트를 시작하기 전 어떤 API를 어디에 사용할 것인지 결정하는 부분이 굉장히 중요하다는 사실을 이번에 느낄 수 있었고
더 이상 사용하지 않는 서비스 로직이 있다면 과감하게 주석처리를 하든 아니면 새로운 객체를 생성하든 해서 
코드를 더 간결하고 서비스 친화적으로 작성하는 것이 중요하다는 것을 이번 기회를 통해 많이 느꼈습니다.

이상입니다. 긴 글 읽어주셔서 감사합니다.