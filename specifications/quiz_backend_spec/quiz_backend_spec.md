# 📡 Quiz_Microservice Specification

[[_TOC_]]

## Overview
Quiz Server Microservice that enable users to receive and answer quiz questions.

## 💾 Quiz Storage model
```mermaid
---
title: Quiz Storage ERD
---
%%{init: {'theme':'base'}}%%
erDiagram
    QUESTIONS {
        UUID _id PK "Unique identifier for the question."
        TEXT statement "The text of the question."
        UUID correct_option_id "Points to the correct option for this question."
        UUID next_question_id FK "Refers to the subsequent question (optional, for question sequencing)."
    }

    OPTIONS {
        UUID _id "Unique identifier for the option."
        UUID question_id FK "Links to the associated question."
        TEXT name "The text for the option."
    }

    ANSWERS {
        UUID _id PK "Unique identifier for the answer."
        UUID user_id "Refers to the user who answered."
        UUID question_id FK "Links to the associated question."
        UUID selected_option_id FK "Points to the selected option."
        BOOLEAN is_correct "Indicates whether the selected option is correct."
        DATETIME created_at "Timestamp when the answer was submitted."
    }

    QUIZ_PROGRESS {
        UUID _id PK "Unique identifier for the quiz progress record."
        UUID user_id "Refers to the user participating in the quiz."
        UUID active_question_id "References the current active question."
        DATETIME quiz_started_at "Start time of the quiz session."
        DATETIME quiz_ended_at "End time of the quiz session."
    }

    QUESTIONS ||--|{ OPTIONS : ""
    QUESTIONS ||--|| ANSWERS : ""
    ANSWERS ||--o| OPTIONS : ""
    QUESTIONS ||--o| QUESTIONS : ""
```

### 🗂️ Inital load
- [Questionnaire data - DEMO](assets/questionnaire.yaml)

## ✨ Quiz API

### 📜 API Commons
A shared set of standards or common guidelines applicable across various APIs or Features.

#### 🎯 Purpose
The application `Quiz_Microservice` servers to perform Create, Read, Update, and Delete (CRUD) operations on `Quiz Storage` to persist and retreive data.

#### 🔑 Authorization
APIs use `JWT token` provided to the API Comsumer by `Authentication_Server` to authorize any call.

`JWT` has to be sent with the `bearer` prefix with every request in `authorization` header parameter.

```http
Authorization: Bearer <JWT token>
```
**Authorization**: The HTTP header used for passing the credentials.

**Bearer**: The type of token being used. In this case, it's a Bearer token.

**`<token>`**: The actual JWT, which is a long string composed of three parts separated by dots (.). The three parts are:
- **Header**: Contains metadata about the token, such as the type of token and the signing algorithm used.
- **Payload**: Contains the claims. Claims are statements about an entity (typically, the user) and additional data.
- **Signature**: Used to verify that the sender of the JWT is who it says it is and to ensure that the message wasn't changed along the way.

```http
GET /quiz/progress HTTP/1.1
Host: https://quiz.demo.slsp.sk:8443
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### 📐 API Structure
- [**Quiz OpenAPI**](quiz-openapi.yaml)

### 🧠 Mappings & Business Logic

#### ⚙️ GET /quiz/progress - Get personalized quiz progress

##### 📤 Request

###### 🔧 Parameters
| Parameter name    | In     | Type   | Mand/Opt | Attribute description | Expected values / Format          | Authentication Server   | Target Attribute / Logic         |
| ----------------- | ------ | ------ | -------- | --------------------- | --------------------------------- | ----------------------- | -------------------------------- |
| **Authorization** | Header | String | M        | Authorization token   | OAuth 2.0 Bearer - Base64 encoded | POST /oauth2/introspect | Standard Client Credentials Flow |

##### 📥 Response

###### ✅ HTTP 200
Description of **Progress** resource attributes:
| Level | Attribute name    | Type/Enum | Mand/Opt | Attribute description                                                | Expected values / Format                                   | Quiz Storage  | Target Attribute / Logic                                                      |
| ----- | ----------------- | --------- | -------- | -------------------------------------------------------------------- | ---------------------------------------------------------- | ------------- | ----------------------------------------------------------------------------- |
| 1     | answeredQuestions | Integer   | M        | A number of questions that have already been answered in the quiz.   | Positive number                                            | ANSWERS       | Count _id in ANSWERS by user_id                                               |
| 1     | totalQuestions    | Integer   | M        | The total number of questions available in the quiz.                 | Positive number                                            | QUESTIONS     | Count _id in QUESTIONS by user_id                                             |
| 1     | correctAnswers    | Integer   | M        | The total number of questions answered correctly by the participant. | Positive number                                            | ANSWERS       | Count is_correct in ANSWERS by user_id                                        |
| 1     | activeQuestion    | String    | C        | An Id of the currently active question in the quiz.                  | UUIDv4                                                     | QUIZ_PROGRESS | active_question_id <br><br> Return only IF answeredQuestions < totalQuestions |
| 1     | quizStartedAt     | Timestamp | C        | The timestamp when the quiz started.                                 | Integer value representing the total time in milliseconds. | QUIZ_PROGRESS | quiz_started_at                                                               |
| 1     | quizEndedAt       | Timestamp | C        | The timestamp when the quiz ended.                                   | Integer value representing the total time in milliseconds. | QUIZ_PROGRESS | quiz_ended_at <br><br> Return only IF answeredQuestions == totalQuestions     |

##### 🔬 Internal Logic
```mermaid
%%{init: {'theme':'base'}}%%
sequenceDiagram
    autonumber
    participant client as 📱 Quiz_App
    participant quizServer as 📡 Quiz_Microservice
    participant db as 💾 Quiz Storage

    client ->>+ quizServer:  GET /quiz/progress

    %% Get User ID
    quizServer ->> quizServer: Extract client_id from Token introspection response
    note over quizServer, quizServer: client_id ~ user_id

    %% Get QUIZ_PROGRESS record
    quizServer ->>+ db: SELECT * FROM QUIZ_PROGRESS WHERE user_id = client_id
    note over quizServer, db: Retrieve QUIZ_PROGRESS record by UserId
    db -->>- quizServer: 🧩 QUIZ_PROGRESS record
      
    %% Check IF user is just starting the quiz
    opt IF QUIZ_PROGRESS record for UserId is empty
        quizServer ->>+ db: SELECT * FROM QUESTIONS q WHERE q._id NOT IN (SELECT next_question_id FROM QUESTIONS WHERE next_question_id IS NOT NULL) LIMIT 1;
        note over quizServer, db: Find & retrieve the first question in QUESTIONS
        db -->>- quizServer: 🧩 The first question's data

        quizServer ->>+ db: 🧩💾 INSERT INTO QUIZ_PROGRESS (_id, user_id, active_question_id, quiz_started_at, quiz_ended_at) <br> VALUES (UUID(), '<client_id>', '<Progress.activeQuestion>', NOW(), NULL);
        note over quizServer, db: Create QUIZ_PROGRESS for UserId AND set with Progress resource values
        deactivate db
    end
    
    quizServer ->>+ db: SELECT COUNT(*) AS answer_count FROM ANSWERS WHERE user_id = '<client_id>'
    db -->>- quizServer: 🧩 Count of ANSWERS by UserId
    note over db, quizServer: answeredQuestions, correctAnswers

    quizServer ->>+ db: SELECT COUNT(*) AS question_count FROM QUESTIONS WHERE user_id = '<client_id>'
    db -->>- quizServer: 🧩 Count of QUESTIONS by UserId
    note over db, quizServer: totalQuestions

    quizServer -->>- client: ✅💎 The personalised quiz progress resource
```

#### ⚙️ GET /quiz/questions/{id} - Get single quiz question

##### 📤 Request

###### 🔧 Parameters
| Parameter name    | In     | Type   | Mand/Opt | Attribute description | Expected values / Format          | Target Service                                | Target Attribute / Logic         |
| ----------------- | ------ | ------ | -------- | --------------------- | --------------------------------- | --------------------------------------------- | -------------------------------- |
| **Authorization** | Header | String | M        | Authorization token   | OAuth 2.0 Bearer - Base64 encoded | Authentication Server POST /oauth2/introspect | Standard Client Credentials Flow |
| id                | Path   | String | M        | Question Id           | UUIDv4                            | Quiz Storage QUESTIONS                        | QUESTIONS._id                    |

##### 📥 Response

###### ✅ HTTP 200
Description of **Question** resource attributes:
| Level | Attribute name | Type/Enum | Mand/Opt | Attribute description                                                                                                         | Expected values / Format | Quiz Storage | Target Attribute / Logic |
| ----- | -------------- | --------- | -------- | ----------------------------------------------------------------------------------------------------------------------------- | ------------------------ | ------------ | ------------------------ |
| 1     | statement      | String    | M        | Represents the main text or prompt of a quiz question.                                                                        | Plain text.              | QUESTIONS    | statement                |
| 1     | options        | Array     | M        | A list of possible answers or choices provided for a quiz question.                                                           |                          | QUESTIONS    | options                  |
| 2     | id             | String    | M        | A unique identifier for each option within a question.                                                                        | UUIDv4                   | OPTIONS      | _id by QUESTIONS._id     |
| 2     | name           | String    | M        | The text or label of the option, representing the content displayed to the participant as a possible answer for the question. | Plain text.              | OPTIONS      | name by QUESTIONS._id    |

###### 🔴 HTTP 404
| Error code     | Scope | Error Description           | Logic                                                                            |
| -------------- | ----- | --------------------------- | -------------------------------------------------------------------------------- |
| `ID_NOT_FOUND` |       | Question Id does not exist. | This error occurs when a specified Question Id, cannot be located in the system. |

##### 🔬 Internal Logic
```mermaid
%%{init: {'theme':'base'}}%%
sequenceDiagram
    autonumber
    participant client as 📱 Quiz_App
    participant quizServer as 📡 Quiz_Microservice
    participant db as 💾 Quiz Storage

    %% Get User ID
    client ->>+ quizServer: GET /quiz/questions/{id}

    %% Get Question
    quizServer ->>+ db: SELECT q._id, q.statement, o._id, o.name FROM QUESTIONS q JOIN OPTIONS o ON q._id = o._id WHERE q._id = '<id>'

    rect rgb(255,230,230)
        break ID_NOT_FOUND
            db -->> quizServer: 🔴 Empty result
            quizServer -->> client: 🔴 404 ID_NOT_FOUND
        end
    end    

    db ->>- quizServer: ✅ Data of requested question and its related options

    quizServer -->>- client: ✅💎 Question resource
```
#### ⚙️ POST /quiz/answer - Submit the answer and receive answer evaluation

##### 📤 Request

###### 🔧 Parameters
| Parameter name    | In     | Type | Mand/Opt | Attribute description | Expected values / Format          | Authentication Server   | Target Attribute / Logic         |
| ----------------- | ------ | ---- | -------- | --------------------- | --------------------------------- | ----------------------- | -------------------------------- |
| **Authorization** | Header | TEXT | M        | Authorization token   | OAuth 2.0 Bearer - Base64 encoded | POST /oauth2/introspect | Standard Client Credentials Flow |

###### 📝 Body
Description of **AnswerEntry** resource attributes:
| Level | Attribute name   | Type/Enum | Mand/Opt | Attribute description                                                                                   | Expected values / Format | Quiz Storage | Target Attribute / Logic |
| ----- | ---------------- | --------- | -------- | ------------------------------------------------------------------------------------------------------- | ------------------------ | ------------ | ------------------------ |
| 1     | questionId       | String    | M        | A unique identifier of the answered question.                                                           | UUIDv4                   | QUESTIONS    | _id                      |
| 1     | selectedOptionId | String    | M        | The unique identifier of the option chosen by the participant as their answer to the answered question. | UUIDv4                   | OPTIONS      | _id                      |

##### 📥 Response

###### ✅ HTTP 200
Description of **Answer** resource attributes:
| Level | Attribute name   | Type/Enum | Mand/Opt | Attribute description                                                                                   | Expected values / Format | Quiz Storage       | Target Attribute / Logic                                |
| ----- | ---------------- | --------- | -------- | ------------------------------------------------------------------------------------------------------- | ------------------------ | ------------------ | ------------------------------------------------------- |
| 1     | id               | String    | M        | A unique identifier assigned to a specific answer.                                                      | UUIDv4                   | ANSWERS            | _id                                                     |
| 1     | questionId       | String    | M        | A unique identifier of the answered question.                                                           | UUIDv4                   | ANSWERS            | question_id                                             |
| 1     | selectedOptionId | String    | M        | The unique identifier of the option chosen by the participant as their answer to the answered question. | UUIDv4                   | ANSWERS            | selected_option                                         |
| 1     | correctOptionId  | String    | M        | The unique identifier of the option that is the correct answer to a specific question.                  | UUIDv4                   | OPTIONS, QUESTIONS | correct_option_id in QUESTIONS by questionId in OPTIONS |

###### 🔴 HTTP 400
| Error code       | Scope | Error Description                                                               | Logic                                                                                                                                                 |
| ---------------- | ----- | ------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `INVALID_ANSWER` |       | This error occurs when the provided answer does not meet the expected criteria. | It could result from selecting an invalid option, submitting an empty or null response, or choosing an option not associated with the given question. |

##### 🔬 Internal Logic
```mermaid
%%{init: {'theme':'base'}}%%
sequenceDiagram
    autonumber
    participant client as 📱 Quiz_App
    participant quizServer as 📡 Quiz_Microservice
    participant db as 💾 Quiz Storage

    client ->>+ quizServer:  POST /quiz/answer

    %% Get User ID
    quizServer ->> quizServer: Extract client_id from Token introspection response
    note over quizServer, quizServer: client_id ~ user_id

    %% Input validations - selectedOptionId
    quizServer ->>+ db: SELECT q.* FROM QUESTIONS q JOIN OPTIONS o ON q._id = o.question_id WHERE o._id = '<option_id>'
    db -->>- quizServer: question_id by selectedOptionId result
    rect rgb(255,230,230)
        opt question_id by selectedOptionId != questionId
            break INVALID_ANSWER
                db -->> quizServer: 🔴 Empty result
                quizServer -->> client: 🔴 400 INVALID_ANSWER - selectedOptionId does not belong to questionId
            end
        end
    end

    %% Input validations - questionId
    %% Get correctOptionId
    quizServer ->>+ db: 🎯 SELECT * FROM OPTIONS WHERE _id = (SELECT correct_option_id FROM QUESTIONS WHERE _id = '<questionId>')
    note over quizServer, db: Retrieve the correct option by questionId
    rect rgb(255,230,230)
        break INVALID_ANSWER
            db -->> quizServer: 🔴 Empty result
            quizServer -->> client: 🔴 400 INVALID_ANSWER - questionId does not exist
        end
    end
    db -->>- quizServer: 🎯 correct_option_id

    %% Answer evaluation and storage
    alt IF selectedOptionId == correct_option_id
        quizServer -->>+ db: 📝✅ INSERT INTO ANSWERS (_id, user_id, question_id, selected_option_id, is_correct, created_at) <br> VALUES (UUID(), '<client_id>', '<questionId>', '<selectedOptionId>', TRUE, NOW()) 
        deactivate db
    else electedOptionId != correct_option_id
        quizServer -->>+ db: 📝❌ INSERT INTO ANSWERS (_id, user_id, question_id, selected_option_id, is_correct, created_at) <br> VALUES (UUID(), '<client_id>', '<questionId>', '<selectedOptionId>', FALSE, NOW()) 
        deactivate db
    end
    
    %% Update Progress - Check IF Next question exist
    quizServer ->>+ db: SELECT next_question_id FROM QUESTIONS WHERE _id = '<questionId>'
    db -->>- quizServer: ➡️ new_active_question_id
    
    %% Update Progress - New active question
    alt IF new_active_question_id is NOT NULL OR new_active_question_id is NOT EMPTY
        quizServer ->>+ db: 🔄 UPDATE QUIZ_PROGRESS SET active_question_id = '<new_active_question_id>' WHERE user_id = '<client_id>'
        note over quizServer, db: Update User's QUIZ_PROGRESS with new_active_question_id
    else 
    %% Update Progress - Quiz end's timestamp
        quizServer ->>+ db: 🔄 UPDATE QUIZ_PROGRESS SET active_question_id = NULL, quiz_ended_at = NOW() WHERE user_id = '<client_id>'
        note over quizServer, db: Update User's QUIZ_PROGRESS with quizEndedAt timestamp and remove active_question_id
    end

    quizServer -->>- client: ✅ User's Answer resource
```

## 📑 Related Documentation
- [🏗️ Solution Architecture - Component Diagram](../../solution_design/solution_architecture/assets/quiz_backend-component_diagram-simplified.svg)