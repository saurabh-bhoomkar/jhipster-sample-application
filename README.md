Below is a "super duper" prompt you can feed your agentic coder AI. This prompt is crafted to capture all the intricate details of our reactive Kafka-to-Oracle design, ensuring that your agentic coder AI builds a production-grade solution in Spring Boot on Java 21.

---

**Super Duper Prompt for the Agentic Coder AI**

You are an expert software engineer tasked with building a high-throughput, fully reactive Kafka listener application. The application must process up to 500,000 messages per minute and perform a “select-then-insert” operation on an Oracle database using Oracle’s reactive drivers (R2DBC or the latest reactive Oracle driver). The solution should be built on Spring Boot (Java 21) and adhere to the following specifications:

1. **Reactive Kafka Consumption:**
   - Use **Reactor Kafka** as the Kafka client. The consumer should subscribe to the relevant Kafka topic(s) and expose a nonblocking, backpressure-aware `Flux<String>` of messages.
   - Optionally incorporate an LMAX Disruptor as a low-latency ring buffer to decouple ingestion from processing if required, but the primary pipeline should be fully reactive.

2. **Processing Pipeline:**
   - Implement a `ReactiveProcessingService` that orchestrates the business logic. It should receive Kafka messages, perform necessary validations or transformations, and then invoke the reactive database operations.
   - Ensure the reactive pipeline uses Reactor operators like `flatMap`, `buffer`, and appropriate schedulers to manage parallelism and backpressure.

3. **Reactive Oracle Operations:**
   - Develop a `ReactiveOracleClient` that uses Oracle’s reactive driver to perform:
     - A nonblocking **SELECT** operation based on the message payload.
     - A nonblocking **INSERT** operation that uses the data obtained from the SELECT.
   - Implement reactive transactions:
     - Explicitly begin a transaction on a connection acquired from a pool (configured to not exceed 10,000 connections).
     - Chain the SELECT and INSERT operations within the same transaction.
     - Commit the transaction on success or roll back on any error using reactive operators (`onErrorResume`).
   - Avoid using Spring Data abstractions—manage the database operations directly using the reactive driver APIs.

4. **Transaction and Connection Management:**
   - Ensure that all transactional operations occur on the same database connection.
   - Use nonblocking, asynchronous calls for `beginTransaction()`, `commitTransaction()`, and `rollbackTransaction()`.
   - Properly close or release connections in a finally-like reactive manner.

5. **Error Handling and Observability:**
   - Integrate comprehensive error handling: any error in the pipeline should trigger an appropriate rollback and log the incident.
   - Use Reactor’s retry mechanisms where applicable to handle transient failures.
   - Ensure Kafka offsets are committed only after successful processing.

6. **Spring Boot Integration:**
   - Leverage Spring Boot for dependency injection, configuration, and monitoring/logging.
   - Ensure the application is production-ready with proper modularization and clear separation of concerns.

7. **Code Quality and Readability:**
   - Write production-quality code with comprehensive comments explaining the reactive design decisions.
   - Follow best practices for asynchronous programming, resource management, and reactive streams.
   - Provide a detailed architecture diagram (in PlantUML) illustrating the flow from Kafka consumption through processing to the Oracle DB operations.

---

Your final deliverable should be a fully functional Spring Boot application that meets these requirements. Use clear, modular, and well-documented code. The agentic coder AI must ensure the application is highly efficient, nonblocking, and scalable, fully leveraging the power of reactive programming for both Kafka and Oracle database operations.

---

This prompt encapsulates every nuance we discussed and should provide your agentic coder AI with clear guidance to build a robust, production-ready reactive Kafka-to-Oracle pipeline.
