# Impact Analysis

## Impacted Teams

### Team - Security

#### Component - Authentication_Server
- Provide standard Oauth 2.0 solution to secure API access of Quiz Server for the users.
- Users will use only name and email as authorization credentials. See User Authroization in Quiz We Application specification.

##### Specifications
- [Authentication_Server Specification - API Structure](../../specifications/authentication_server_spec/authentication_server-openapi.yaml)
- [Authentication_Server Specification - Business Logic](../../specifications/authentication_server_spec/authentication_server.md)

### Team - Backend
- Provide data for the Quiz Application

#### Component - Quiz_Microservice
- Create new Microservice deployed to AWS to process Quiz Web Application requests according to specification.

##### Specifications
- [Quiz Server Specification - Structure](../../specifications/quiz_backend_spec/quiz_backend_spec-openapi.yaml)
- [Quiz Server Specification - Logic](../../specifications/quiz_backend_spec/quiz_backend_spec.md)

### Team - Frontend

#### Component - Quiz_App
- Create new mobile-first web application that delivers quizzes with immediate feedback and end-of-quiz summary.
- Application will be interacting with Quiz Server as its only source of data.

##### Specifications
- [Quiz Web Application Specification](../../specifications/quiz_frontend_spec/quiz_frontend_spec.md)