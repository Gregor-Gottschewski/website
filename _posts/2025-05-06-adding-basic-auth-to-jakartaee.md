---
layout: post
title:  "Using Basic Authentication in a Jakarta EE Application"
categories: jakartaee
---

# Using Basic Authentication in a Jakarta EE Application

Basic authentication is a simple way to secure your application by requiring users to provide username and password. The password is unencrypted sent over a HTTP header. Only by using HTTPS a secure authentication can be achieved. There are multiple ways of implementing basic authentication in Jakarta EE applications, but I will show you a way that can be combined with databases and other authentication mechanisms. Other ways are using the built-in security features of the application server.

At the end you will have a Jakarta EE application that uses basic authentication to secure a REST endpoint.

> **Where can I find the source code?**
>
> The complete source code of the example can be found on [GitHub](https://github.com/Gregor-Gottschewski/jakartaee-basic-auth).

## Dependencies and setup

This project was tested on WildFly 35.0.1 and JakartaEE 11 and depends on the following dependencies (the version are the latest versions at the time of writing):

```xml
<dependencies>
    <dependency>
        <groupId>jakarta.platform</groupId>
        <artifactId>jakarta.jakartaee-core-api</artifactId>
        <version>11.0.0</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>jakarta.servlet</groupId>
        <artifactId>jakarta.servlet-api</artifactId>
        <version>6.1.0</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>jakarta.security.enterprise</groupId>
        <artifactId>jakarta.security.enterprise-api</artifactId>
        <version>4.0.0</version>
    </dependency>
</dependencies>
```

> ⚠️ Warning: At time of writing, the latest version of WildFly 36.0.0 causes problems when following this tutorial. On WildFly 35.0.1 everything works as expected. If I find a solution, I will update this post.

## Let's start

For this project, just a simple structure with three classes is needed. One class contains the code for the REST endpoint, and the second class contains a so called identity store. An identity store is a black box, where credentials come in and identity goes out. I name the first two classes creatively `HelloWorldResource` and `BasicAuthIdentStore`. The `HelloWorldResource` class contains a simple REST endpoint with a `GET` method that returns either "Hello unauthenticated user" if not username and password are provided or "Hello [username]!" otherwise.

The most interesting part of the `HelloWorldResource` class is the use of `@Context SecurityContext securityContext`. The `SecurityContext` is a interface that provides information about the authenticated user. We use this interface to check if the user is authenticated. Te retrieve the username, you can use `securityContext.getUserPrincipal().getName()` like shown underneath.

```java
package me.gregorgott.jakartaeebasicauth;

import jakarta.enterprise.context.RequestScoped;
import jakarta.security.enterprise.authentication.mechanism.http.BasicAuthenticationMechanismDefinition;
import jakarta.ws.rs.Consumes;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.core.Context;
import jakarta.ws.rs.core.MediaType;
import jakarta.ws.rs.core.SecurityContext;

@RequestScoped
@Path("/hello-world")
@BasicAuthenticationMechanismDefinition(realmName = "my-realm")
public class HelloWorldResource {

    /**
     * This method is called when the user sends a GET request to the /hello-world endpoint.
     * If no auth header was transmitted, a "Not authenticated" message is returned.
     * Otherwise, the username and password are returned.
     */
    @GET
    @Consumes(MediaType.APPLICATION_JSON)
    public String createStory(@Context SecurityContext securityContext) {
        if (securityContext.getUserPrincipal() == null) {
            // this branch is executed if the user is not authenticated
            return "Not authenticated";
        }

        // a auth header was sent
        return "Hello " + securityContext.getUserPrincipal().getName() + "!";
    }
}

```

More interesting is the `BasicAuthIdentStore` class. This class implements the `IdentityStore` interface. The heart of this class is the `validate` method, that is called by the application server to authenticate the user. Normally, you would check the username and password by comparing it to a database or similar. In this example, I just check if a username and password were sent. 

```java
package me.gregorgott.jakartaeebasicauth;

import jakarta.enterprise.context.ApplicationScoped;
import jakarta.security.enterprise.credential.UsernamePasswordCredential;
import jakarta.security.enterprise.identitystore.CredentialValidationResult;
import jakarta.security.enterprise.identitystore.IdentityStore;

import java.util.Set;

@ApplicationScoped
public class BasicAuthIdentStore implements IdentityStore {

    public CredentialValidationResult validate(UsernamePasswordCredential credential) {
        String username = credential.getCaller();
        String password = credential.getPasswordAsString();

        if (username == null || password == null) {
            // no username and password were sent
            return CredentialValidationResult.INVALID_RESULT;
        }

        return new CredentialValidationResult(username, Set.of("user"));
    }
}
```

If you decompile the `UsernamePasswordCredential` class, you see that this class only stores username and password. This is the only moment were you can check the credentials, later in the `HelloWorldResource` only the username can be retrieved!

You have three options to return in `validate`:
1. `CredentialValidationResult.INVALID_RESULT` - the user is not authenticated.
2. `CredentialValidationResult.NOT_VALIDATED_RESULT` - the user is not authenticated, but the credentials were not checked.
3. `CredentialValidationResult` - the user is authenticated.

The last option uses the constructor `CredentialValidationResult(String callerName, Set<String> groups)`. The first parameter is the username and the second parameter is a set of roles. In this example, I just use the role "user". Another more simple constructor is `CredentialValidationResult(String callerName)`, which does not use roles. This constructor is used if you do not need roles in your application. Our code, generally does not need roles, but it shows you the usage of them.

The last class to implement is the `ApplicationMain` class. This class is nothing special and does not differ from a normal Jakarta EE application:

```java
package me.gregorgott.basicauthexample;

import jakarta.ws.rs.ApplicationPath;
import jakarta.ws.rs.core.Application;

import java.util.Set;

/**
 * This is the entry point of the application.
 */
@ApplicationPath("/api")
public class ApplicationMain extends Application {

    @Override
    public Set<Class<?>> getClasses() {
        return Set.of(
                StoryApi.class,
                ApplicationIdentityStore.class
        );
    }
}
```

## Testing the application

To test the application, you can use a tool like Postman or curl. The following command uses curl to send a GET request to the `/hello-world` endpoint without authentication:

```bash
curl -X GET http://localhost:8080/jakartaee-basic-auth-1.0.0/api/hello-world
```
This should return the message "Not authenticated". To test the application with authentication, use the following command:

```bash
curl -X GET http://localhost:8080/jakartaee-basic-auth-1.0.0/api/hello-world -u username:password
```

That's everything! Now you have a simple Jakarta EE application that uses basic authentication to secure a REST endpoint. I hope this tutorial was helpful and you learned something new. If you have any questions or feedback, feel free to [contact me](/impressum.md).

Happy Coding! 🚀