 CRUD APIs for User Entity in Spring Boot using Firebase
A reference proof-of-concept that leverages [Firestore Database](https://firebase.google.com/docs/firestore) to perform CRUD operations and  [Firebase Authentication](https://firebase.google.com/docs/auth) with Spring-Security to authenticate users. 

 Application Flow and Security Configuration



Below is a sample controller method declared as public which will be exempted from authentication checks:

```java
@PublicEndpoint
@PostMapping(value = "/login")
public ResponseEntity<TokenSuccessResponse> login(@RequestBody UserLoginRequest userLoginRequest) {
  final var response = userService.login(userLoginRequest);
  return ResponseEntity.ok(response);
}
```
API requests to private endpoints, handling CRUD operations on the [Task](https://github.com/hardikSinghBehl/firebase-integration-spring-boot/blob/main/src/main/java/com/behl/flare/entity/Task.java) entity are intercepted by the [JwtAuthenticationFilter](https://github.com/hardikSinghBehl/firebase-integration-spring-boot/blob/main/src/main/java/com/behl/flare/filter/JwtAuthenticationFilter.java), which is added to the security filter chain and configured in the [SecurityConfiguration](https://github.com/hardikSinghBehl/firebase-integration-spring-boot/blob/main/src/main/java/com/behl/flare/configuration/SecurityConfiguration.java). The custom filter holds the responsibility for verifying the authenticity of the incoming access token by communicating with the Firebase Authentication service and populating the security context.

Post successful authentication, [AuthenticatedUserIdProvider](https://github.com/hardikSinghBehl/firebase-integration-spring-boot/blob/main/src/main/java/com/behl/flare/utility/AuthenticatedUserIdProvider.java) is responsible for retrieving the authenticated user-id from the security context. This helps the [Service layer](https://github.com/hardikSinghBehl/firebase-integration-spring-boot/blob/main/src/main/java/com/behl/flare/service/TaskService.java) maintain relationship between the current authenticated user and their corresponding Tasks. If an authenticated user attempts to perform any action on Tasks not owned by them, then the below API response is sent back to the client.

```json
{
  "Status": "403 FORBIDDEN",
  "Description": "Access Denied: Insufficient privileges to perform this action."
}
```

In the event of authentication failure, when the access token received in the HTTP Request Headers is not valid, the below API response is sent back to the client.

```json
{
  "Status": "401 UNAUTHORIZED",
  "Description": "Authentication failure: Token missing, invalid or expired"
}


### Local Setup

To run the application locally, ensure you have the following prerequisites:
* A private key associated with the service account to establish a connection with Firebase.
* The Web API key of the Firebase project you've created to invoke the Firebase Authentication REST API.
* The created Firebase Authentication service has the `Email/Password` native sign-in provider enabled.

Create a file named `private-key.json` in the base directory and paste the contents of the service account's private key into this file.

Execute the following commands in the project's base directory to build the application image and start the backend application container:

```bash
FIREBASE_PRIVATE_KEY=$(cat private-key.json)
```

```bash
FIREBASE_WEB_API_KEY=your-web-api-key-here
```

```bash
sudo docker-compose build
```

```bash
sudo FIREBASE_PRIVATE_KEY="$FIREBASE_PRIVATE_KEY" FIREBASE_WEB_API_KEY="$FIREBASE_WEB_API_KEY" docker-compose up -d
```

To remove the environment variables from memory after the application has started, the below commands can be executed

```bash
unset FIREBASE_PRIVATE_KEY
```

```bash
unset FIREBASE_WEB_API_KEY
```


