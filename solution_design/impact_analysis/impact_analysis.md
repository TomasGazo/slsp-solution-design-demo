# Impact Analysis

## Impacted Teams

### 👥 Team - Security
- OAuth 2.0 Authentication Server Implementation
- Simplified User Authentication
- Auth Storage Database Development

#### 🔒 Component - Authentication_Server
- Implement a standard OAuth 2.0 authentication server to secure API access for the Quiz Server, ensuring proper user authentication and authorization workflows.
- Configure the authentication process to allow users to log in using only their **name** and **email** as credentials, aligning with the application’s simplified requirements.
- Build an `Auth Storage` as a database to store and manage user credentials, tokens, and other authentication-related data, ensuring secure storage and efficient retrieval.

##### Specifications
- [Authentication_Server Specification - API Structure](../../specifications/authentication_server_spec/authentication_server-openapi.yaml)
- [Authentication_Server Specification - Business Logic](../../specifications/authentication_server_spec/authentication_server.md)

### 👥 Team - Backend
- Build a scalable Quiz Microservice on AWS with APIs for question retrieval, answer submission, progress tracking, and result evaluation.
- Implement a secure and efficient Quiz Storage system to manage quiz data, ensuring extensibility and support for future analytics integration.

#### 📡 Component - Quiz_Microservice
- Develop & deploy `Quiz_Microservice`
  - The creation of the `Quiz_Microservice` involves the deployment of a scalable backend service to AWS, which will process requests from the `Quiz_App` Application in accordance with the defined specifications.
  - This microservice is critical to the application’s architecture and functionality.
- The `Quiz_Microservice` will utilize a dedicated `Quiz Storage` system for persisting and retrieving data necessary for quiz operations.
- Implement a secure and efficient `Quiz Storage` system to manage quiz data, ensuring extensibility and support for future analytics integration.

##### Specifications
- [Quiz_Microservice Specification - API Structure](../../specifications/quiz_backend_spec/quiz-openapi.yaml)
- [Quiz_Microservice Specification - Mappings & Business Logic](../../specifications/quiz_backend_spec/quiz_backend_spec.md)

### 👥 Team - Frontend
- Develop Multiplatform Quiz Application
- Integrate with Quiz Microservice
- Implement User Signup via Authorization Server

#### 📱 Component - Quiz_App
- Develop a multiplatform mobile application (`Quiz_App`) to deliver quizzes (single predefined quiz for `DEMO`), providing users with immediate feedback after each question and a detailed end-of-quiz summary, including score, elapsed time, and answer history.
- Ensure the application interacts exclusively with the `Quiz_Microservice` for all data operations, including retrieving questions, submitting answers, and fetching progress and evaluation data.
- Integrate the application with the `Authorization_Server` to handle user signup and authentication, allowing users to securely register and log in using their name and email.

##### Specifications
- [Quiz Web Application Specification](../../specifications/quiz_frontend_spec/quiz_frontend_spec.md)