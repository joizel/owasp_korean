============================================================================================
OTG-BUSLOGIC-003
============================================================================================

|

무결성 검사 테스트

|


개요
============================================================================================

대부분 어플리케이션은 숨겨진 입력 부분을 두고 사용자에 따라 서로 다른 필드를 표시하도록 설계되어 있습니다.
그러나, 이런 경우 프록시를 사용하여 서버측에 숨겨진 필드 값을 입력할 수 있습니다.
이 경우, 서버측 컨트롤 관계형 또는 서버측의 적절한 데이터가 서버와 어플리케이션의 특정 비즈니스 로직에 기반하여 허용되도록 편집해야합니다.

Additionally, the application must not depend on non-editable controls, drop-down menus or hidden fields for business logic processing because these fields remain non-editable only in the context of the browsers. 

Users may be able to edit their values using proxy editor tools and try to manipulate business logic. If the application exposes values related to business rules like quantity, etc. as non-editable fields it must maintain a copy on the server side and use the same for business logic processing. Finally, aside application/system data, log systems must be secured to prevent read, writing and updating. Business logic integrity check vulnerabilities is unique in that these misuse cases are application specific and if users are able to make changes one should only be able to write or update/edit specific artifacts at specific times per the business process logic. 

The application must be smart enough to check for relational edits and not allow users to submit information directly to the server that is not valid, trusted because it came from a non-editable controls or the user is not authorized to submit through the front end. Additionally, system artifacts such as logs must be "protected" from unauthorized read, writing and removal. 

|

예제
============================================================================================

|

예제 1 
-----------------------------------------------------------------------------------------

Imagine an ASP.NET application GUI application that only allows the admin user to change the password for other users in the system. The admin user will see the username and password fields to enter a username and password while other users will not see either field. However, if a non admin user submits information in the username and password field through a proxy they may be able to "trick" the server into believing that the request has come from an admin user and change password of other users. 

|

예제 2 
-----------------------------------------------------------------------------------------

Most web applications have dropdown lists making it easy for the user to quickly select their state, month of birth, etc. Suppose a Project Management application allowed users to login and depending on their privileges presented them with a drop down list of projects they have access to. What happens if an attacker finds the name of another project that they should not have access to and submits the information via a proxy. Will the application give access to the project? They should not have access even though they skipped an authorization business logic check. 

|

예제 3 
-----------------------------------------------------------------------------------------

Suppose the motor vehicle administration system required an employee initially verify each citizens documentation and information when they issue an identification or driver's license. At this point the business process has created data with a high level of integrity as the integrity of submitted data is checked by the application. Now suppose the application is moved to the Internet so employees can log on for full service or citizens can log on for a reduced self-service application to update certain information. At this point an attacker may be able to use an intercepting proxy to add or update data that they should not have access to and they could destroy the integrity of the data by stating that the citizen was not married but supplying data for a spouse's name. This type of inserting or updating of unverified data destroys the data integrity and might have been prevented if the business process logic was followed. 

|

예제 4 
-----------------------------------------------------------------------------------------

Many systems include logging for auditing and troubleshooting purposes. But, how good/valid is the information in these logs? Can they be manipulated by attackers either intentionally or accidentially having their integrity destroyed? 

|


테스트 방법
============================================================================================

일반 테스트 방법
-----------------------------------------------------------------------------------------

- Review the project documentation and use exploratory testing looking for parts of the application/system (components i.e. For example, input fields, databases or logs) that move, store or handle data/information. 
- For each identified component determine what type of data/information is logically acceptable and what types the application/system should guard against. Also, consider who according to the business logic is allowed to insert, update and delete data/information and in each component. 
- Attempt to insert, update or edit delete the data/information values with invalid data/information into each component (i.e. input, database, or log) by users that .should not be allowed per the busines logic workflow. 


구체적인 테스트 방법 1 
-----------------------------------------------------------------------------------------
 
- 프록시 캡쳐를 사용하여 HTTP 트래픽에 숨겨진 필드 검색
- 만약 히든 필드를 찾았다면, 이 필드를 GUI 어플리케이션과 비교하고, 프록시를 통해 비즈니스 처리를 우회하는 데이터 값을 입력하여 의도하지 않는 값을 처리


구체적인 테스트 방법 2 
-----------------------------------------------------------------------------------------

- 프록시 캡쳐를 사용하여 HTTP 트래픽에 non-editable 어플리케이션 영역에서 정보 삽입이 가능한 곳 검색
- 만약 삽입이 가능한 곳을 찾았다면, 이 부분을 어플리케이션과 비교하고, 프록시를 통해 비즈니스 처리를 우회하는 데이터 값을 입력하여 의도하지 않는 값을 처리

구체적인 테스트 방법 3 
-----------------------------------------------------------------------------------------

- 수정할 수 있는 어플리케이션 또는 시스템 구성을 목록화 (로그, 데이터베이스)
- 식별된 구성 요소에서 읽기, 수정, 삭제 시도. 예를 들어, 로그 파일을 확인하고, 테스터는 수집한 데이터 및 정보를 조작 시도

|

Related Test Cases 
============================================================================================

- 모든 입력 유효성 검사 테스트 케이스

|

Tools 
============================================================================================

- Various system/application tools such as editors and file manipulation tools. 
- OWASP Zed Attack Proxy (ZAP) - https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project 


|

References 
============================================================================================

- Implementing Referential Integrity and Shared Business Logic in a RDB - http://www.agiledata.org/essayreferentialIntegrity. html 
- On Rules and Integrity Constraints in Database Systems http://www.comp.nus.edu.sg/~lingtw/papers/IST92.teopk.pdf 
- Use referential integrity to enforce basic business rules in Oracle - http://www.techrepublic.com/article/use-referentialintegrity-to-enforce-basic-business-rules-in-oracle/ 
- Maximizing Business Logic Reuse with Reactive Logic - http:/ architects.dzone.com/articles/maximizing-business-logic 
- Tamper Evidence Logging - http://tamperevident.cs.rice.edu/Logging.html 

|

Remediation 
============================================================================================

The application must be smart enough to check for relational edits and not allow users to submit information directly to the server that is not valid, trusted because it came from a non-editable controls or the user is not authorized to submit through the front end. Additionally, any component that can be edited must have mechanisms in place to prevent unintentional/intentional writing or updating. 

|