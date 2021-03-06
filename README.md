# Table of contents
1. [Banking APIs Documentation](#BankingAPIsDocumentation)
	1. [Construction](#Construction)
2. [OAuth APIs](#OAuthAPIs)
    1. [GenerateAuthCode](#GenerateAuthCode)
    2. [GenerateAccessToken](#GenerateAccessToken)
    3. [GenerateCCAccessToken](#GenerateCCAccessToken)
    4. [RefreshToken](#RefreshToken)
    5. [InvalidateToken](#InvalidateToken)
3. [Banking APIs](#BankingAPIs)
	1. [Banking APIs Introduction](#BankingAPIsIntroduction)
	1. [UsersAPI](#UsersAPI)
	1. [AccountsAPI](#AccountsAPI)
	1. [PaymentsAPI](#PaymentsAPI)
	1. [BalancesAPI](#BalancesAPI)
	1. [CustomerProfileAPI](#CustomerProfileAPI )


# Banking APIs Documentation <a name="BankingAPIsDocumentation"></a>

Design document for the Banking Project demonstration for MetroBank. 

## Construction <a name="Construction"></a>

The project has three sets of APIs : Banking APIs, CoreBankingBackend Mock API and OAuth API for generation of the OAuth Access tokens. 

### Associated Meta Structure

All endpoints described below are a part of the BankApp Developer App which has the following custom attributes : 
|Name|Value|
|---|---|
|accessTokenDurationInMS|3600000|
|authCodeDurationInMS|1800000|
|phoneNumber|123123123|
|quota|2000|
|quotaTimeunit|day|
|spikeArrest|100ps|

These attributes are used in the OAuth flows to set the expiry times and an attribute to the access tokens. 

### OAuth APIs <a name="OAuthAPIs"></a>

The OAuth APIs are a part of the BankingFunctionalities API Product and the BankApp Devveloper App.   

This API has multiple endpoints

#### GenerateAuthCode <a name="GenerateAuthCode"></a>
This resource covers the initial part of the three-legged oauth flow. the API expects the following parameters : 

URL : ```https://harivr-eval-prod.apigee.net/oauth/code```

|Parameter Type|Parameter name|Value|Comments|
|------------|-------|---|---|
|Method|Verb|```POST```|
|Form Parameter|response_type|```code```|
|Form Parameter|client_id|```3qS85PA1McC8i1Vk05bfNnLSa17RS87I```|APIKey for the registered App
|Form Parameter|scope|```READ WRITE```|Scope with which the Token is to be generated
|Header|Content-Type|```x-www-form-urlencoded```|
|Header|responseCode|200/302|200 returns the AuthCode as a response, 302 issues the redirect. Currently, the 302 redirect fails as there is no actual endpoint.

Sample Call: 

```
curl -X POST 'https://harivr-eval-prod.apigee.net/oauth/code' 
	-H 'Content-Type: application/x-www-form-urlencoded' 
    -H 'responseCode: 200' 
    -d 'response_type=code&client_id=3qS85PA1McC8i1Vk05bfNnLSa17RS87I&scope=READ'
```

The expiry time of these AuthCodes is set as the ```authCodeDurationInMS``` value from the BankApp developer app. 

#### GenerateAccessToken <a name="GenerateAccessToken"></a>

This resource generates the actual access token for the three-legged OAuth flow. 

URL : ```https://harivr-eval-prod.apigee.net/oauth/accesstoken```


|Parameter Type|Parameter name|Value|Comments|
|------------|-------|---|---|
|Method|Verb|```POST```|
|Form Parameter|code|```XXXXX```|The Auth Code that has been obtained from the GenerateAuthCode resource
|Form Parameter|grant_type|```authorization_code```|As per the OAuth RFC, the grant_type has to be passed to the OAuth endpoint to inform the Token Provider of the type of AccessToken that is required.
|Header|Content-Type|```x-www-form-urlencoded```|
|Header|Authorization|```Basic XXXXXXXXXXXXXXX```|The Basic Auth token identifies the Developer app under which the Token is to be generated. The token is generated by ```Base64Encode(ConsumerKey + ':' + ConsumerSecret)```

Sample Call: 

```
curl -X POST 'https://harivr-eval-prod.apigee.net/oauth/accesstoken'
  -H 'Authorization: Basic M3FTODVQQTFNY0M4aTFWazA1YmZObkxTYTE3UlM4N0k6UzF1a1hrenplS1M4amo5Rw=='
  -H 'Content-Type: application/x-www-form-urlencoded'
  -d 'code=1GriibYH&grant_type=authorization_code'
```

The expiry time of these AccessTokens is set as the ```accessTokenDurationInMS``` value from the BankApp developer app.  The phoneNumber attribute from the developer app is added as an attribute to the access token.

#### GenerateCCAccessToken <a name="GenerateCCAccessToken"></a>

This resource generates the Client-Credential type access token. 

URL : ```https://harivr-eval-prod.apigee.net/oauth/cctoken```


|Parameter Type|Parameter name|Value|Comments|
|------------|-------|---|---|
|Method|Verb|```POST```|
|Form Parameter|scope|```READ WRITE```|Scope with which the Token is to be generated
|Form Parameter|grant_type|```client_credentials```|As per the OAuth RFC, the grant_type has to be passed to the OAuth endpoint to inform the Token Provider of the type of AccessToken that is required.
|Header|Content-Type|```x-www-form-urlencoded```|
|Header|Authorization|```Basic XXXXXXXXXXXXXXX```|The Basic Auth token identifies the Developer app under which the Token is to be generated. The token is generated by ```Base64Encode(ConsumerKey + ':' + ConsumerSecret)```

Sample Call: 

```
curl -X POST 'https://harivr-eval-prod.apigee.net/oauth/cctoken' 
  -H 'Authorization: Basic M3FTODVQQTFNY0M4aTFWazA1YmZObkxTYTE3UlM4N0k6UzF1a1hrenplS1M4amo5Rw==' 
  -H 'Content-Type: application/x-www-form-urlencoded' 
  -d 'scope=WRITE%20READ&grant_type=client_credentials'
```

The expiry time of these AccessTokens is set as the ```accessTokenDurationInMS``` value from the BankApp developer app.  The phoneNumber attribute from the developer app is added as an attribute to the access token.

#### RefreshToken <a name="RefreshToken"></a>

This resource generates a new access token using the RefreshToken generated during the three-legged oauth flow. 

URL : ```https://harivr-eval-prod.apigee.net/oauth/refreshtoken ```


|Parameter Type|Parameter name|Value|Comments|
|------------|-------|---|---|
|Method|Verb|```POST```|
|Form Parameter|grant_type|```refresh_token```|As per the OAuth RFC, the grant_type has to be passed to the OAuth endpoint to inform the Token Provider of the type of AccessToken that is required.
|Header|Content-Type|```x-www-form-urlencoded```|
|Header|Authorization|```Basic XXXXXXXXXXXXXXX```|The Basic Auth token identifies the Developer app under which the Token is to be generated. The token is generated by ```Base64Encode(ConsumerKey + ':' + ConsumerSecret)```

Sample Call: 

```
curl -X POST 'https://harivr-eval-prod.apigee.net/oauth/refreshtoken'
  -H 'Authorization: Basic M3FTODVQQTFNY0M4aTFWazA1YmZObkxTYTE3UlM4N0k6UzF1a1hrenplS1M4amo5Rw==' 
  -H 'Content-Type: application/x-www-form-urlencoded' 
  -d 'grant_type=refresh_token&refresh_token=puAdtUtS4evqGLFCh1G3YqFepAjeFiVs'
```

#### InvalidateToken <a name="InvalidateToken"></a>

This resource generates the actual access token for the three-legged OAuth flow. 

URL : ```https://harivr-eval-prod.apigee.net/oauth/invalidate```


|Parameter Type|Parameter name|Value|Comments|
|------------|-------|---|---|
|Method|Verb|```POST```|
|Form Parameter|token|```ZLb1P5I3uApCEUcGDMImB6lfx84c```|The AccessToken that needs to be invalidated
|Header|Content-Type|```x-www-form-urlencoded```|

Sample Call: 

```
curl -X POST https://harivr-eval-prod.apigee.net/oauth/invalidate 
  -H 'Content-Type: application/x-www-form-urlencoded' 
  -d token=ZLb1P5I3uApCEUcGDMImB6lfx84c
```

### Banking APIs <a name="BankingAPIs"></a>

#### Introduction <a name="BankingAPIsIntroduction"></a>

Multiple BankingAPI endpoints have been designed and created: 
* UsersAPI -> GetUserInfo,UpdateUserInfo
* AccountsAPI
* BalancesAPI
* PaymentsAPI

These APIs are all secured using the OAuth2.0 tokens which are generated by the OAuth2.0 APIs described above. 

The services are secured using an OAuth2.0 access token. The request is converted to a SOAP request and forwarded to the CoreBankingService, which is a SOAP based service that mocks an actual Backend Service. The SOAP response is then transformed into a JSON payload for presentation to the customer. 

The traffic to these APIProxies is controlled using a Quota and a SpikeArrest policy which are incorporated into the APIProxy as a sharedflow Flow Callout. These policies use values which are set in the BankApp developer app - these values are obtained after the associated AccessToken that came in the request is validated. 


#### UsersAPI  <a name="UsersAPI"></a>

##### GetUserInfo : 

URL : ``` https://harivr-eval-prod.apigee.net/users/1001```


|Parameter Type|Parameter name|Value|Comments|
|------------|-------|---|---|
|Method|Verb|```GET```|
|Header|Content-Type|```application/json```|
|Header|Authorization|```Bearer XXXXXXXXX```|The access token generated using ```client_credential``` or ```authorization_code``` grants
|Path Parameter|userId|```1001```|The userId of whose details are required. Dummy values 1001 and 1002 have been created in the dummy backend. 

Sample call:

```
curl -X GET 'https://harivr-eval-prod.apigee.net/users/1001'
  -H 'Authorization: Bearer 21Utu4X7XBMs7p2QAOZYtFktWGxS' 
  -H 'Content-Type: application/json' 
```

This API responds with a JSON payload that contains some user details. 

##### UpdateUserInfo : 

URL : ``` https://harivr-eval-prod.apigee.net/users/1001```


|Parameter Type|Parameter name|Value|Comments|
|------------|-------|---|---|
|Method|Verb|```PUT```|
|Header|Content-Type|```application/json```|
|Header|Authorization|```Bearer XXXXXXXXX```|The access token generated using ```client_credential``` or ```authorization_code``` grants
|Path Parameter|userId|```1001```|The userId of whose details are required. Dummy values 1001 and 1002 have been created in the dummy backend. 
|Payload|firstName|```FName1```|
|Payload|lastName|```LName1```|
|Payload|email|```FName1.LName1@random.com```|
|Payload|phoneNo|```+417894561231```|

Sample call:

```
curl -X PUT 'https://harivr-eval-prod.apigee.net/users/1002'
  -H 'Authorization: Bearer IvH7ChAW4xpqq4rtyB5LwSQU3Upj'
  -H 'Content-Type: application/json' \
  -d '{
    "firstName": "FName22",
    "lastName": " LName22",
    "email": " FName2.LName2@random.com",
    "phoneNo": " +417894561222"
}'
```

This API responds with a JSON payload that contains some user details. 

#### AccountsAPI  <a name="AccountsAPI"></a>

##### Request Parameters : 

URL : ``` https://harivr-eval-prod.apigee.net/accounts```


|Parameter Type|Parameter name|Value|Comments|
|------------|-------|---|---|
|Method|Verb|```POST```|
|Header|Content-Type|```application/json```|
|Header|Authorization|```Bearer XXXXXXXXX```|The access token generated using ```client_credential``` or ```authorization_code``` grants
| Payload|accountId|```XYZ123456```|The accountId of whose details are required. This data is sent as a JSON payload

Sample call:

```
curl -X POST 'https://harivr-eval-prod.apigee.net/accounts'
  -H 'Authorization: Bearer 21Utu4X7XBMs7p2QAOZYtFktWGxS' 
  -H 'Content-Type: application/json' 
  -d '{
	"accountId": "XYZ123456"
}'
```

This API responds with a JSON payload that contains some Account information. 


#### PaymentsAPI  <a name="PaymentsAPI"></a>

##### Request Parameters : 

URL : ``` https://harivr-eval-prod.apigee.net/payments/qweqwe```


|Parameter Type|Parameter name|Value|Comments|
|------------|-------|---|---|
|Method|Verb|```GET```|
|Header|Content-Type|```application/json```|
|Header|Authorization|```Bearer XXXXXXXXX```|The access token generated using ```client_credential``` or ```authorization_code``` grants
|Path Parameter|paymentId|```qweqwe```|The paymentId whose details are required.
Sample call:

```
curl -X GET https://harivr-eval-prod.apigee.net/payments/qweqwe 
  -H 'Authorization: Bearer 21Utu4X7XBMs7p2QAOZYtFktWGxS' 
  -H 'Content-Type: application/json' 
```

This API responds with a JSON payload that contains some Account information. 

#### BalancesAPI  <a name="BalancesAPI"></a>

##### Request Parameters : 

URL : ``` https://harivr-eval-prod.apigee.net/balances```


|Parameter Type|Parameter name|Value|Comments|
|------------|-------|---|---|
|Method|Verb|```POST```|
|Header|Content-Type|```application/json```|
|Header|Authorization|```Bearer XXXXXXXXX```|The access token generated using ```client_credential``` or ```authorization_code``` grants
|Payload|accountId|```XYZ123456```|The accountId of whose balance details are required. This data is sent as a JSON payload

Sample call:

```
curl -X POST 'https://harivr-eval-prod.apigee.net/balances'
  -H 'Authorization: Bearer 21Utu4X7XBMs7p2QAOZYtFktWGxS' 
  -H 'Content-Type: application/json' 
  -d '{
	"accountId": "XYZ123456"
}'
```

This API responds with a JSON payload that contains the balance information. 

#### CustomerProfileAPI  <a name="CustomerProfileAPI"></a>

##### Request Parameters : 

URL : ``` https://harivr-eval-prod.apigee.net/customerprofile/1001```


|Parameter Type|Parameter name|Value|Comments|
|------------|-------|---|---|
|Method|Verb|```GET```|
|Header|Content-Type|```application/json```|
|Header|Authorization|```Bearer XXXXXXXXX```|The access token generated using ```client_credential``` or ```authorization_code``` grants
|Path Parameter|userId|```1001```|The userId whose consolidated details are required.
Sample call:

```
curl -X GET https://harivr-eval-prod.apigee.net/customerprofile/1001 
  -H 'Authorization: Bearer 21Utu4X7XBMs7p2QAOZYtFktWGxS' 
  -H 'Content-Type: application/json' 
```

This API responds with a JSON payload that contains the userInformation, PaymentInformation, BalanceInformation and some AccountInformation for that specific user. 








