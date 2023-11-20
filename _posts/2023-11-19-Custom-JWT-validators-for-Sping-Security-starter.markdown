---
layout: post
title: "Custom JWT validators for Spring Security starter"
date: 2023-11-19 15:26:00 +0200
categories: spring security
description: Custom JWT validators for Spring Security starter
---

### Use case

It's a common case when REST API is secured with OAuth 2.0 and acts as a Resource server in terms of Auth 2.0. The
easiest way to achieve this with Spring Security is to
use [Spring Boot OAuth2 Resource Server starter](https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/index.html){:
target="_blank"}. This starter requires minimal configuration and works perfectly fine out of the box. You can see an
example of the project in the official Spring repository with
examples - [Spring Security Examples](https://github.com/spring-projects/spring-security-samples/blob/main/servlet/spring-boot/java/oauth2/resource-server/hello-security){:
target="_blank"}.

### Problem

The starter comes with some default JWT validators, e.g. `JwtTimestampValidator` to check if token is expired
and `JwtIssuerValidator` to check that issuer URI matches the expected one (you can see the implementation
in [Spring Security repo](https://github.com/spring-projects/spring-security/blob/d9587875619f568054a58107fd80d38bff59c1d7/oauth2/oauth2-jose/src/main/java/org/springframework/security/oauth2/jwt/JwtValidators.java#L54){:
target="_blank"}).
However, if you need to add some custom validator for JWT, there is no way to do this without redeclaring all the things
that autoconfiguration does for you. As you can see in
the [starter's source code](https://github.com/spring-projects/spring-boot/blob/3.1.x/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/security/oauth2/resource/servlet/OAuth2ResourceServerJwtConfiguration.java){:
target="_blank"}, default validators are created inside the configuration instead of being injected to it (that would be
a *Spring way* to do it).

### Solution

Starting from Spring Boot 3.2.0 it's not an issue anymore. Because now you can add custom JWT validators to
Spring Security configuration simply by adding validator beans to your context. The beans should be of type
`OAuth2TokenValidator<Jwt>`.

As an example let's consider a case where we need to add a custom JWT claim validation.
Spring Security provides a suitable `JwtClaimValidator` class that implements `OAuth2TokenValidator` interface.
In that case, to add the validation we should declare a bean the following way:

```java
  @Bean
  public JwtClaimValidator<String> customJwtClaimValidator() {
    return new JwtClaimValidator<>(
        "claimName",
        actualValue -> "expectedValue".equals(actualValue)
    );
  }
```

In this example we validate that claim `claimName` has value equal to `expectedValue`, otherwise the token is considered
not valid.

### Appreciations

Special thank you to [Dmitry Denshchikov](https://medium.com/@HereAndBeyond) for the invaluable editing assistance,
and to Spring Boot team for reviewing and accepting my PR for this feature.