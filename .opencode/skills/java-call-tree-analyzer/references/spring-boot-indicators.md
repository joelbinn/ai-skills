# Spring Boot Analysis Indicators

Use these indicators to identify key architectural components during the call tree analysis.

## JPA Entities
Look for classes with:
- `@Entity`: Marks a class as a JPA entity.
- `@Table`: Often accompanies `@Entity`.
- `@Id`: Marks the primary key.
- `@Column`: Marks persistent fields.

**State Change Indicators:**
- Setter methods (`setName`, `setAmount`).
- Constructors (check if new instances are created).
- `repository.save()`, `repository.delete()`.

## External Systems & Integration
Look for these patterns to identify calls leaving the application boundary:

### 1. REST/HTTP Clients
- `@FeignClient`: Declarative REST clients.
- `RestTemplate`: Synchronous HTTP client.
- `WebClient`: Reactive/Asynchronous HTTP client.
- `HttpClient`: Native Java HTTP client.

### 2. Messaging
- `@JmsListener` / `JmsTemplate`: Java Message Service.
- `@KafkaListener` / `KafkaTemplate`: Apache Kafka.
- `@RabbitListener` / `RabbitTemplate`: RabbitMQ.

### 3. External API Services
- Classes ending in `Client` or `Gateway`.
- Classes in packages named `infrastructure.external`, `adapter.out.rest`, or similar.
- Service calls that reference third-party SDKs (e.g., Stripe, AWS, SendGrid).

## Infrastructure Boundaries
- **Repositories:** Interfaces extending `JpaRepository`, `CrudRepository`, or `Repository`. These mark the boundary to the database.
- **Controllers:** Classes annotated with `@RestController` or `@Controller`. These mark the entry point from external users.
