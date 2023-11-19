---
layout: post
title:  "Custom JWT validators for Spring Security starter"
date:   2023-11-19 15:26:00 +0200
categories: spring security
description: Custom JWT validators for Spring Security starter
---

### Use case

It's a common case when REST API is secured with OAuth 2.0 and acts as a Resouce server in terms of Auth 2.0. The easiest way to achieve this with Spring Security is to use [Spring Boot OAuth2 Resource Server starter](https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/index.html){:target="_blank"}. This starter requires minimal configuration and works perfectly fine out of the box. You can see an example of the project in the official Spring repository with examples - [Spring Security Examples](https://github.com/spring-projects/spring-security-samples/blob/main/servlet/spring-boot/java/oauth2/resource-server/hello-security){:target="_blank"}.

### Problem

The starter comes with some default JWT validators, e.g. [token expiration and issuer URI validators](https://github.com/spring-projects/spring-security/blob/main/oauth2/oauth2-jose/src/main/java/org/springframework/security/oauth2/jwt/JwtValidators.java#L52C35-L52C35){:target="_blank"}. However, if you need to add some custom validator for JWT, there is now way to do this without redeclaring all the things that autoconfiguration does for you. As you can see in the [starter's source code](https://github.com/spring-projects/spring-boot/blob/3.1.x/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/security/oauth2/resource/servlet/OAuth2ResourceServerJwtConfiguration.java){:target="_blank"}, default validators are created inside the configuration instead of being injected to it (that would be a *Spring way* to do it).

### Solution

Starting from Spring Boot 3.2.0 it's not an issue anymore. Because now you can add custom JWT validators to Spring Security configuration simply by adding validator beans to your context. The beans shoud be of type `OAuth2TokenValidator<Jwt>`. Let's imagine we need to validate that some JWT claim has some predefined static value. Here is an example for this case:

```java
  @Bean
  public JwtClaimValidator<String> customJwtClaimValidator() {
    return new JwtClaimValidator<>(
        "claimName",
        actualValue -> "expectedValue".equals(actualValue)
    );
  }
```

Spring Security provides easy to configure `JwtClaimValidator` class for this. It accepts a claim name and a function to validate claim's actual value.
