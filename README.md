# CSS API Documentation

## Overview

CSS is striving to to make exchanging data effortless for all, for this reason the CSS API for third parties has been developed.  This API will allow for approved external vendors and MLSs to access needed CSS scheduling and listing availability data.  The intended use of this API is to allow MLS technology companies to offer scheduling services directly from their product.

This service will use a JSON Web token for securely accessing the CSS API Service.  A JSON web token(s) will be created to authenticate that the requesting service is first, a legitimate request and second, allow for a secure transmission of data between parties.   Using these processes the external vendor can access, form and display the data as necessary from CSS.

Third Party integration will involve the following services: 
1. [Authentication](#authentication)
2. [Obtaining a Token](#obtaining-a-token)
3. [Using the Token](#using-the-token)
4. [API Discovery](#api-discovery)
5. [CSS Listing Determination](#css-listing-determination)

Please note that you must use the secure HTTPS protocol as opposed to the non-supported HTTP for all communication.

The following table lists all the base urls when communicating with CSS endpoints. This documentation will reference the production endpoints, but you can swap out the test versions during development.

| Description             | Production Endpoint       | Test Endpoint                  |
|-------------------------|---------------------------|--------------------------------|
| Main api                | https://api2.showings.com | https://api2-beta.showings.com |
| Authentication endpoint | https://auth.showings.com | https://auth.dev.showings.com  |


CSS APIs use the OAuth 2.0 protocol for authentication and authorization. CSS supports the OAuth 2.0 authorization code and client credential flows which are appropriate for use on web server applications.

## Authentication:  
Many API functions require authentication before they can be accessed. Authentication varies based on the API endpoints being used. Some endpoints such as scheduling a showing can only be performed on behalf of an existing CSS user. Other endpoints such as determining if a listing is managed by CSS can be called by an application.  

To request access to the CSS API you must submit the request form found at https://goo.gl/forms/DEvUycSUJUHagN2v1

Authentication is performed by posting to the CSS oauth2 service which will return the JSON web token that will be used when accessing the API server.  The token that is obtained is valid for one hour at which point it will expire and a new token will need to be generated.

## Obtaining a Token:
1. **Application Token**:
		
	To obtain an application token a POST request must be made to the following address: https://auth.showings.com/connect/token

	This request must include grant_type, client_id, client_secret and scope fields in the body using the the x-www-form-urlencoded content type.

	Request
	```
	POST /connect/Token Https/1.1
	Host: auth.showings.com
	Content-Type: application/x-www-form-urlencoded
	```


	| Key           | Value                   | State    |
	|---------------|-------------------------|----------|
	| grant_type    | client_credentials      | Required |
	| client_id     | [Provided by CSS]       | Required |
	| client_secret | [Provided by CSS]       | Required |
	| scope         | [determined by app use] | Required |

	Sample Response
	```json
	{
		"access_token": "fAwTwHZM4cOS4rnoD1maTaUu0P6xk0i9dA",
		"expires_in": 3600,
		"token_type": "Bearer"
	}
	```

2. **User Token**:
	
	To obtain a user token you must first send the user to https://auth.showings.com/connect/authorize in a web browser so that they can authorize your application to act on their behalf. You must include the following query string parameters in the request


	| Key           | Value                   | Description                                     |
	|---------------|-------------------------|-------------------------------------------------|
	| client_id     | [Provided by CSS]       | Id provided by css                              |
	| scope         | [determined by app use] | A list of scopes separated by spaces            |
	| response_type | code                    | Indicates authorization code flow usage         |
	| redirect_uri  | [client provided]       | Where to redirect the user after authentication |

	If the user successfully authenticates and grants your app access an authorization code will be sent as a query string parameter to the redirect_uri specified in the request.

	Sample Redirect
	https://www.showings.com/?code=2c5e9688283a3b51ab1e5390cdde298e

	This authorization code can then be exchanged for an access_token that can be used with the api. To exchange the code for a token you must perform the following request.

	Request
	```
	POST /connect/Token Https/1.1
	Host: auth.showings.com
	Content-Type: application/x-www-form-urlencoded
	```


	| Key           | Value               | State    |
	|---------------|---------------------|----------|
	| grant_type    | authorization_code  | Required |
	| client_id     | [Provided by CSS]   | Required |
	| client_secret | [Provided by CSS]   | Required |
	| code          | [from query string] | Required |
	| redirect_uri  | [same as above]     | Required |

	Sample Response
	```json
	{
		"access_token": "fAwTwHZM4cOS4rnoD1maTaUu0P6xk0i9dA",
		"expires_in": 3600,
		"token_type": "Bearer",
		"refresh_token": "62e7db79d6620777df46025e6345dc65"
	}
	```

	The access_token provided in this response is valid for 1 hour. After that time a new access_token must be obtained using the refresh_token.

	Request
	```
	POST /connect/Token Https/1.1
	Host: auth.showings.com
	Content-Type: application/x-www-form-urlencoded
	```


	| Key           | Value             | State    |
	|---------------|-------------------|----------|
	| grant_type    | refresh_token     | Required |
	| client_id     | [Provided by CSS] | Required |
	| client_secret | [Provided by CSS] | Required |
	| refresh_token | [from response]   | Required |

	The response is the same as before.

## Using the Token
	
To use the token when accessing the API server, Include the access_token in all requests in the Authorization header field.

Request
```
GET /CsslistingDetermination/12/13425134 HTTPS/1.1
Host: api2.showings.com
Authorization: Bearer fAwTwHZM4c0S4rnoD1maTaUu0P6xk0i9dA
```

If the need arises to validate the token, the following operation can be used.

```
/Development/ValidateAuthorizationHeader
```

If a valid token that hasn’t expired has been provided, a 200 response will be sent with the client_id and scope in the response body. 


## API Discovery

The following link will provide a list of all operations that are supported by the API which can be useful during development.

https://api2.showings.com/docs/sandbox/index

Note: All operations that have the parameter "Authorization" listed, require authentication, this authorization token does not necessarily grant you access to all operations with an authorization parameters.    If an "unauthorized" response received, then access to this operation is not permitted.  If you believe this is an error, please contact CSS.

Many endpoints require a specific scope to be included in the token. To determine what scope is required you can click the exclamation mark in the Swagger documentation.

![Screenshot](/help1.png?raw=true)

This endpoint requires either the mls or scheduling scope.

## CSS Listing Determination

If utilizing the determination service during a search and it is desired that an indicator be placed next to the listing services by CSS as part of the displayed search result, below is that symbol.   This link can be made available to to CSS members and non-members.  
![CSS Icon](/cssicon.png?raw=true)
