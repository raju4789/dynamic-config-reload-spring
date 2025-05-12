# dynamic-config-reload-spring

A production-ready Spring Boot microservices system with:
- **Dynamic configuration reload** using Spring Cloud Config Server
- **Spring Cloud Bus with Kafka** for distributed refresh
- **Manual config refresh using HTTP Basic authentication**
- **Docker Compose** orchestration
- **Secure actuator endpoints**

---

## 🚀 Features

- Centralized configuration management with Spring Cloud Config Server
- Dynamic, distributed config refresh using Kafka and Spring Cloud Bus
- Secure actuator endpoints (public health/info, protected busrefresh)
- Manual config refresh via `/actuator/busrefresh` with HTTP Basic Auth
- Docker Compose setup for local development and testing

---

## 🗂️ Project Structure

```
dynamic-config-reload-spring/
│
├── config-server/         # Spring Cloud Config Server
│   ├── Dockerfile
│   ├── build.gradle
│   └── src/main/...
│
├── demo-service/          # Example microservice
│   ├── Dockerfile
│   ├── build.gradle
│   └── src/main/...
│
└── docker-compose.yml     # Orchestration for Zookeeper, Kafka, Config Server, Demo Service
```

---

## 🛠️ Prerequisites

- Docker & Docker Compose
- Java 17+
- [GitHub account](https://github.com/) (for config repo)

---

## 📦 Step 1: Create a Remote Config Repository

1. **Create a new GitHub repository** (public or private) for your configuration, e.g. `tourni-config`.
2. **Add a config file for your service** (e.g. `demo-service.yml`):

   ```yaml
   greeting:
     message: "Hello from Config Server!"
   ```

3. **Push the file to your repo** (e.g. `https://github.com/your-username/tourni-config.git`).

---

## ⚡ Step 2: Clone and Build This Project

```bash
git clone https://github.com/your-username/dynamic-config-reload-spring.git
cd dynamic-config-reload-spring
docker-compose build
docker-compose up
```

---

## 🔗 Step 3: Configure the Config Server

In `config-server/src/main/resources/application.yml`, set your config repo URL:

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/your-username/tourni-config.git
          clone-on-start: true
          default-label: main
```

---

## 🧑‍💻 Step 4: Develop Your Microservice

- In your microservice (e.g. `demo-service`), set the application name to match your config file:

  ```yaml
  spring:
    application:
      name: demo-service
  ```

- Use `@RefreshScope` on beans that should reload config dynamically:

  ```java
  @RefreshScope
  @RestController
  public class GreetingController {
      @Value("${greeting.message:Hello default}")
      private String message;

      @GetMapping("/greet")
      public String greet() {
          return message;
      }
  }
  ```

---

## 🔄 Step 5: Manual Config Refresh

1. **Update your config in the GitHub repo** (e.g., change `greeting.message`).
2. **Trigger a distributed refresh:**

   ```bash
   curl -X POST -u admin:admin http://localhost:8888/actuator/busrefresh
   ```

3. **All services will reload their config automatically.**
4. **Test the new value:**

   ```bash
   curl http://localhost:8080/greet
   ```

---

## 🔒 Security

- `/actuator/health` and `/actuator/info` are public.
- `/actuator/busrefresh` requires HTTP Basic Auth (`admin:admin` by default).
- All other endpoints require HTTP Basic Auth.
- **Change default credentials for production!**

---

## 📝 Example Config Server `application.yml`

```yaml
server:
  port: 8888

spring:
  security:
    user:
      name: admin
      password: admin
  cloud:
    bus:
      enabled: true
      monitor:
        enabled: true
    config:
      server:
        git:
          uri: https://github.com/your-username/tourni-config.git
          clone-on-start: true
          default-label: main
  kafka:
    bootstrap-servers: kafka:9092

management:
  endpoints:
    web:
      exposure:
        include: "bus-refresh,health,info"
  endpoint:
    shutdown:
      enabled: true
    health:
      probes:
        enabled: true
      show-details: always
```

---

## 🛡️ Production Best Practices

- Use strong, unique secrets and credentials.
- Store secrets in environment variables or a secrets manager.
- Use a private Git repo for config.
- Use managed Kafka for reliability.
- Restrict `/monitor` to accept only signed requests from GitHub if you enable webhooks.

---

## 📝 Troubleshooting

- **Config not updating?**  
  Make sure you POST to `/actuator/busrefresh` after updating the config repo.
- **Kafka errors?**  
  Ensure all services use `kafka:9092` as the bootstrap server in Docker Compose.
- **401 on `/actuator/busrefresh`?**  
  Use `-u admin:admin` with curl or Postman.

---

## 📚 References

- [Spring Cloud Config](https://spring.io/projects/spring-cloud-config)
- [Spring Cloud Bus](https://spring.io/projects/spring-cloud-bus)
- [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/actuator-api/htmlsingle/)

---

**Happy coding! 🚀**
```

---

**Copy and paste this into your `README.md`!**  
Let me know if you want to add a section for advanced security, troubleshooting, or a sample webhook filter.
