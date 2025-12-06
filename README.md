# How to add Swagger Doc for Spring Boot App

This guide explains how to integrate **Swagger / OpenAPI documentation** into a Spring Boot application using **springdoc-openapi**.
Follow these steps in order:

* Step 1: Add Maven dependency
* Step 2: Add properties (including environment-based dev/local/prod config)
* Step 3: Create `SwaggerConfig` class
* Step 4: Run the project
* Step 5: Open Swagger UI URL

---

## Step 1: Add Maven Dependency

In your `pom.xml`, add the **SpringDoc OpenAPI UI** dependency:

```xml
<dependencies>
    <!-- Other dependencies -->

    <!-- Swagger / OpenAPI UI -->
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        <version>2.3.0</version>
    </dependency>
</dependencies>
```

After adding this, rebuild the project (`mvn clean install` or via IDE).
This dependency automatically exposes:

* OpenAPI JSON at: `/v3/api-docs`
* Swagger UI at: `/swagger-ui/index.html` (configurable)

---

## Step 2: Add Properties

We configure two things in properties:

1. **Swagger / SpringDoc properties (common for all environments)**
2. **Environment-based `server.url` for local, dev, and prod**

### 2.1 Common Swagger Properties (Base `application.properties`)

In `src/main/resources/application.properties`, add:

```properties
# Swagger / OpenAPI configuration
springdoc.swagger-ui.path=/swagger-ui/index.html
springdoc.api-docs.enabled=true
springdoc.swagger-ui.enabled=true
```

These properties:

* Enable Swagger UI
* Set the UI path to `/swagger-ui/index.html`
* Enable OpenAPI docs generation

(You are already using this pattern in your existing project. )

---

### 2.2 Environment-based Properties (local, dev, prod)

We usually maintain different property files for different environments, for example:

* `application-local.properties`
* `application-dev.properties`
* `application-prod.properties`

Each environment will have its own `server.url` which Swagger will use to show the correct base server in the UI.

#### a) `application-local.properties`

```properties
# Local environment server URL
server.url=http://localhost:8080
```

You can also add DB and other local configs here as needed.

#### b) `application-dev.properties`

```properties
# Dev environment server URL
server.url=https://dev-api.yourdomain.com
```

(Your current dev file already follows this pattern using a dev URL. )

#### c) `application-prod.properties`

```properties
# Prod environment server URL
server.url=https://api.yourdomain.com
```

(Your prod properties already have a similar `server.url` pointing to the live domain. )

#### d) Activating Profiles

Use Spring profiles to pick the correct environment:

* For **local** (default), you can set:

  ```properties
  # in application.properties
  spring.profiles.active=local
  ```

* For **dev**:

  Run with JVM arg:
  `-Dspring.profiles.active=dev`

* For **prod**:

  Run with JVM arg:
  `-Dspring.profiles.active=prod`

Based on the active profile, Spring will pick the right `server.url`, which is then injected into Swagger config.

---

## Step 3: Create `SwaggerConfig` Class

Create a configuration class, for example:
`src/main/java/com/yourcompany/yourapp/config/SwaggerConfig.java`

```java
package com.yourcompany.yourapp.config;

import io.swagger.v3.oas.models.Components;
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.security.SecurityRequirement;
import io.swagger.v3.oas.models.security.SecurityScheme;
import io.swagger.v3.oas.models.servers.Server;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SwaggerConfig {

    // This value will come from environment-specific properties (local/dev/prod)
    @Value("${server.url}")
    private String serverUrl;

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
                // Basic API Info
                .info(new Info()
                        .title("My Application APIs")
                        .version("1.0")
                        .description("API documentation for My Application"))
                // Server URL (different per environment)
                .addServersItem(new Server().url(serverUrl))
                // Security requirement (JWT bearer authentication)
                .addSecurityItem(new SecurityRequirement().addList("BearerAuth"))
                .components(new Components()
                        .addSecuritySchemes("BearerAuth",
                                new SecurityScheme()
                                        .type(SecurityScheme.Type.HTTP)
                                        .scheme("bearer")
                                        .bearerFormat("JWT")
                                        .name("Authorization")
                        )
                );
    }
}
```

This is conceptually same as the `SwaggerConfig` you already have in your existing project, just generalized.

* `@Value("${server.url}")` → picks URL from the active profile (local/dev/prod).
* `addServersItem(new Server().url(serverUrl))` → shows correct base URL in Swagger UI.
* `SecurityScheme` → adds JWT bearer token support so Swagger UI shows an **Authorize** button.

---

## Step 4: Run the Project

Start the Spring Boot application normally:

* Via IDE: `Run Application`
* Or using Maven:
  `mvn spring-boot:run`
* Or using jar:
  `java -jar your-app.jar --spring.profiles.active=dev`

Make sure the correct profile is active so that the right `server.url` and DB configs are used.

---

## Step 5: Open Swagger UI URL

Once the app is running, open Swagger UI in your browser.

* **Local environment:**

  ```text
  http://localhost:8080/swagger-ui/index.html
  ```

* **Dev environment:**

  ```text
  https://dev-api.yourdomain.com/swagger-ui/index.html
  ```

* **Prod environment:**

  ```text
  https://api.yourdomain.com/swagger-ui/index.html
  ```

The page will show:

* All **REST APIs** from your `@RestController` classes
* If JWT is configured, an **Authorize** button to enter the token
* Request/response models, parameters, status codes, etc.

---

## Notes / Common Issues

1. If `/swagger-ui/index.html` gives 404:

   * Check the dependency is present and version is correct.
   * Check `springdoc.swagger-ui.enabled=true`.
   * Check the path: `springdoc.swagger-ui.path=/swagger-ui/index.html`.

2. If server URL in Swagger UI is wrong:

   * Check `server.url` in the active profile.
   * Confirm the active profile via logs or `spring.profiles.active`.

---
