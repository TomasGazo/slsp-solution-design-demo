# Quiz App Specification - Process

## Overview
A mobile-first web application that delivers quizzes with immediate feedback and end-of-quiz summary.

Application features design approach focused on usability, interactivity, and responsive design.

- Landing page with user authentication.
- Clean, distraction-free UI, focusing on one question at a time.
- Single-choice questions.
- Progress bar and question count (e.g., "Question 5").
- Immediate feedback upon answer submission (correct/incorrect indication).
- Next button to proceed.
- Quiz summary after the last question is answered. 

## Important links
| Description                         | Link                                                                                           |
| ----------------------------------- | ---------------------------------------------------------------------------------------------- |
| Solution Architecture               | [Architecture](../../solution_design/solution_architecture/solution_architecture.md)           |
| Authentication Server Specification | [Authentication Server Specification](../authentication_server_spec/authentication_server.md)  |
| OpenAPI Specification               | [Oauth Server Specification - Logic](../specifications/quiz_backend_spec/quiz_backend_spec.md) |

## Business Logic

### URL
API root: https://slsp.sk/quiz

| Parameter | Mandatory | Business Logic/Usage                                        |
| --------- | --------- | ----------------------------------------------------------- |
| channel   | Y         | process channel - allowed value: APP                        |
| token     | Y         | JWT token                                                   |
| language  | Y         | process language  - allowed values: SK, EN - `OUT-OF-SCOPE` |

Example whole URL: https://slsp.sk/quiz/?channel=APP&language=SK&token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

### Screen design
![Designs](assets/designs/Designs.png)

### Screen flow
![Designs](assets/diagram/Flow%20diagram.png)

### List of user tasks/screens

#### 01 SignUp

##### **Introduction**
The system will display the SignUp screen after loading the URL or after incorrectly filled input data. The screen will capture user information, which will be verified against the server.

##### **Screen description**

###### Graphic design
![01 SignUP - Design](assets/designs/01%20SignUP.png)

###### Screen element description
The parameters and actions are described in the table below:
| Element name | Element type | Mandatory | Editable | Visible | Validation | Mapping | Functionality                                   |
| ------------ | ------------ | --------- | -------- | ------- | ---------- | ------- | ----------------------------------------------- |
| {logo}       | Icon         | Y         | N        | Y       | -          | Static  | Static icon according to capture Graphic design |
| {title}      | String       | Y         | N        | Y       | -          | Static  | Static text according to capture Graphic design |
|{legend}| String       | Y         | N        | Y       | -          | Static  | Static text according to capture Graphic design |
| {name title} | String       | Y         | N        | Y       | -          | Static  | Static text according to capture Graphic design |         
{name value}     |       Input    |Y         |   Y      |        Y    |    The name is validated according to regex ^[a-zA-Z][a-zA-Z0-9_-]{2,15}$     |        Output mapping: attribute username in API POST /oauth2/authorize                |       Placeholder text "Enter your username" is visible only if the user does not write any character       |     
|Meno je nevalidné| Alert| N|N|Text is visible only if {name value} is invalid|-|Static|Static text according to capture Graphic design |
 | {email title} | String       | Y         | N        | Y       | -          | Static  | Static text according to capture Graphic design |         
{email value}     |       Input    |Y         |   Y      |        Y    |    The email is validated according to regex ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$     |        Output mapping: attribute email in API POST /oauth2/authorize                |       Placeholder text "Enter your email address" is visible only if the user does not write any character       | 
|Email je nevalidný| Alert| N|N|Text is visible only if {email value} is invalid|-|Static|Static text according to capture Graphic design | 
Actions         |  
Začať kvíz         |  Button        | Y         |   N         |    Y     |         -                                        |-| After clicking on the button is called API POST /oauth2/authorize and IF response is correct, then FE system store the token and GET /quiz/progress and GET /quiz/questions/{id} are called, , ELSE FE system will display the SignUp screen again.|

###### Sequence diagram
```mermaid
---
title: User Authorization
---
%%{init: {'theme':'base'}}%%
sequenceDiagram
    autonumber
    actor user as User
    participant quizApp as Quiz_App
    participant auth as Authentification_Server

    user ->>+ quizApp: Enter Quiz App
    quizApp -->> user: Quiz SignUP screen

    user ->> quizApp: Submit required Signup inputs
    quizApp ->> quizApp: Validate inputs
    
    quizApp ->>+ auth: POST /oauth2/authorize
    auth -->>- quizApp: Authorization token
    quizApp ->> quizApp: Store token

    quizApp -->>- user: User is authorized
```
#### 02 Question

##### **Introduction**
- The Question screen follows either after the SignUp screen and simultaneously if user was verified (API POST /oauth2/authorize HTTP 200), OR after the evaluation of a question, if it is not the last question.
   
- FE system will call API GET /quiz/progress after storing the token OR after the successful (HTTP 200) call of API POST /quiz/answer. 

- FE system will call API GET /quiz/questions/{id} after the successful (HTTP 200) call of API GET /quiz/progress and simultaneously IF answeredQuestions ≠ totalQuestions. Path parameter {id} represents identifier of the next question accotding to attribute activeQuestion from response API GET /quiz/progress. 

- FE system will call API POST /quiz/answer IF the user selects an answer to the question and simultaneously clicks the submit button.

##### **Screen description**

###### Graphic design
![02_01 Question - Design](assets/designs/02_01%20Question.png)
![02_02 Question evaluation - Design](assets/designs/02_02%20Question%20evaluation.png)

###### Screen element description
The parameters and actions are described in the table below:
| Element name         | Element type | Mandatory | Editable | Visible | Validation | Mapping                                                                                                                                                           | Functionality                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| -------------------- | ------------ | --------- | -------- | ------- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| {logo}               | Icon         | Y         | N        | Y       | -          | Static                                                                                                                                                            | Static icon according to capture Graphic design                                                                                                                                                                                                                                                                                                                                                                                                                        |
| {number of question} | String       | Y         | N        | Y       | -          | Input mapping: attribute id in API GET /quiz/questions/{id}                                                                                                       | Static text Otázka # according to capture Graphic design and simultaneously the attribut id                                                                                                                                                                                                                                                                                                                                                                            |
| {process completion} | Progress bar | Y         | N        | Y       | -          | Input mapping: attributes answeredQuestions and totalQuestions in API GET /quiz/progress                                                                          | Progress setting (answerdQuestions+1)/totalQuestions                                                                                                                                                                                                                                                                                                                                                                                                                     |
| {question text}      | String       | Y         | N        | Y       | -          | Input mapping: attribute statement in API GET /quiz/questions/{id}. Ouput mapping: attribute selectedOptionId in API POST /quiz/answer.                                                                                              | Text from the attribute statement is displayed                                                                                                                                                                                                                                                                                                                                                                                                                       |
| {Question options}   | Radio button | Y         | Y        | Y       | -          | Input mapping: attribute options[].id and options[].name in API GET /quiz/questions/{id}, attribute selectedOptionId and correctOptionId in API POST /quiz/answer. Ouput mapping: attribute selectedOptionId in API POST /quiz/answer. | The system arranges the questions in ascending order according to the attribute options[].id. The text of the options will be displayed from attribute options[].name. Static text A, B, C, D according to capture Graphic design. Upon the first loading of the question, no answer is selected. The system marks the selected answer. The correct answer will be highlighted. If the selected answer is not correct, then the wrong answer will also be highlighted. |
| {motivational message} |    SnackBar          |     N      |N          |Text is visible only if answer wasn't correct         |   -         |  Input mapping: attribute selectedOptionId and correctOptionId in API POST /quiz/answer                                                                                                                                                                 |      Static text according to capture Graphic design. Text is visible only if selectedOptionId ≠ correctOptionId (The answer is not correct).  |
Actions         |  
Odoslať OR Pokračovať    |  Button        | Y         |   N         |    Y     |         -                                        |-| If the answer was not selected OR the answer was selected but the button was not clicked, i.e., API  POST /quiz/answer was not called yet, then a static text Odoslať will be displayed, ELSE static text Pokračovať will be displayed. After clicking on the button is called API POST /quiz/answer and IF response is correct (HTTP 200), then system will display the correct answer according to correctOptionId. After clicking the button following the display of the correct answer, the system will call API GET /quiz/progress. IF API GET /quiz/progress is correct (HTTP 200) and simultaneously IF answeredQuestions ≠ totalQuestions, then system will call API GET /quiz/questions/{id}, where {id} represents identifier of the next question according to attribute activeQuestion from response API GET /quiz/progress and simultaneously the system will display the Question screen again. IF API GET /quiz/progress is correct (HTTP 200) and simultaneously IF answeredQuestions = totalQuestions, then system will display Quiz evaluation screen. |

###### Sequence diagram
```mermaid
---
title: Quiz Client-Server communication
---
%%{init: {'theme':'base'}}%%
sequenceDiagram
    autonumber
    actor user as User
    participant quizApp as Quiz_App
    participant quizServer as Quiz_Microservice

    user ->>+ quizApp: Start the Quiz
    quizApp ->>+ quizServer: GET /quiz/progress
    quizServer -->>- quizApp: Quiz progression data
    quizApp ->> quizApp: Update quiz progress bar
    
    loop Repeat until answeredQuestions = totalQuestions
        alt answeredQuestions ≠ totalQuestions
            quizApp ->>+ quizServer: GET /quiz/questions/{id}
            quizServer ->> quizServer: Retrieve quiz question
            quizServer -->>- quizApp: Statement and options
    
            quizApp -->> user: Display the question
            user ->> quizApp: Choose option as answer
            quizApp -->> user: Mark chosen option as selected
            user ->> quizApp: Submit the answer

            quizApp ->>+ quizServer: POST /quiz/answer
            quizServer ->> quizServer: Evaluate the answer
            quizServer -->>- quizApp: Answer evaluation
    
            alt Answer is correct
                quizApp ->> quizApp: Highlight the selected option as correct
            else Answer is wrong
                quizApp ->> quizApp: Highlight the correct option
                quizApp ->> quizApp: Highlight the selected option as wrong
            end

            quizApp -->> user: Receive answer evaluation
            user ->> quizApp: Continue (to next question)

            quizApp ->>+ quizServer: GET /quiz/progress
            quizServer -->>- quizApp: Quiz progress resource

        else answeredQuestions = totalQuestions
            user ->> quizApp: Continue
            quizApp ->>+ quizServer: GET /quiz/progress
            quizServer -->>- quizApp: Personalized quiz evaluation
            quizApp -->>- user: Display personalized quiz evaluation
    
        end
    end    
```
#### 03 Quiz evaluation

##### **Introduction**
- The Quiz evaluation screen follows after Question screen and if it is the last question.

##### **Screen description**

###### Graphic design
![03 Quiz evaluation - Design](assets/designs/03%20Quiz%20evaluation.png)

###### Screen element description
The parameters and actions are described in the table below:
| Element name          | Element type | Mandatory | Editable | Visible | Validation | Mapping                                                                              | Functionality                                                                                                            |
| :-------------------- | :----------- | :-------- | :------- | :------ | :--------- | :----------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------- |
| {logo}                | Icon         | Y         | N        | Y       | -          | Static                                                                               | Static icon according to capture Graphic design                                                                          |
| {title}               | String       | Y         | N        | Y       | -          | -                                                                                    | Static text according to capture Graphic design                                                                          |
| {subtitle}            | String       | Y         | N        | Y       | -          | -                                                                                    | Static text according to capture Graphic design                                                                          |
| {score title}         | String       | Y         | N        | Y       | -          | Static                                                                               | Static text according to capture Graphic design                                                                          |
| {score value}         | String       | Y         | N        | Y       | -          | Input mapping: attribute correctAnswers and totalQuestions in API GET /quiz/progress | System will display in format correctAnswers + "/" + totalQuestions                                                      |
| {quiz duration title} | String       | Y         | N        | Y       | -          | Static                                                                               | Static text according to capture Graphic design                                                                          |
| {quiz duration value} | String       | Y         | N        | Y       | -          | Input mapping: attribute quizStartedAt and quizEndedAt in API GET /quiz/progress     | System will calculate the difference between quizEndedAt and quizStartedAt. Duration is displayed in minutes and seconds |
| {motivational text}   | String       | Y         | N        | Y       | -          | Static                                                                               | Static text according to capture Graphic design                                                                          |
| Actions               |              |           |          |         |            |                                                                                      |                                                                                                                          |
| {entire screen}       | Button       | Y         | N        | Y       | -          | -                                                                                    | After clicking on the screen, the system will close the application.                                                     |

###### Sequence diagram
```mermaid
---
title: Quiz Client-Server communication
---
%%{init: {'theme':'base'}}%%
sequenceDiagram
    autonumber
    actor user as User
    participant quizApp as Quiz_App
    participant quizServer as Quiz_Microservice

    user ->>+ quizApp: Continue (after the last question)
    quizApp ->>+ quizServer: GET /quiz/progress
    
    quizServer -->>- quizApp: Personalized quiz evaluation
    quizApp -->>- user: Display personalized quiz evaluation   
```
#### Technical error

##### **Introduction**
- The Technical error screen will be displayed in case of an error, i.e. IF API GET /quiz/questions/{id} OR POST /quiz/answer will end HTTP 4xx/5xx.

##### **Screen description**

###### Graphic design
![Technical error - Design](assets/designs/Technical%20error.png)

###### Screen element description
The parameters and actions are described in the table below:
| Element name    | Element type | Mandatory | Editable | Visible | Validation | Mapping | Functionality                                                        |
| :-------------- | :----------- | :-------- | :------- | :------ | :--------- | :------ | :------------------------------------------------------------------- |
| {logo}          | Icon         | Y         | N        | Y       | -          | Static  | Static icon according to capture Graphic design                      |
| {title}         | String       | Y         | N        | Y       | -          | -       | Static text according to capture Graphic design                      |
| {subtitle}      | String       | Y         | N        | Y       | -          | -       | Static text according to capture Graphic design                      |
| Actions         |              |           |          |         |            |         |                                                                      |
| {entire screen} | Button       | Y         | N        | Y       | -          | -       | After clicking on the screen, the system will close the application. |