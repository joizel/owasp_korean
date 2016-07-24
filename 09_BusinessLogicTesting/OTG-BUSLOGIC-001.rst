============================================================================================
OTG-BUSLOGIC-001 (비즈니스 로직 데이터 인증 테스트)
============================================================================================

|

개요
============================================================================================

어플리케이션은 논리적으로 프론트 엔드에서 뿐만 아니라 시스템 어플리케이션의 서버 측에서도 유효한 데이터 입력이 되도록 해야합니다. 


Only verifying data locally may leave applications vulnerable to server injections through proxies or at handoffs with other systems. This is different from simply performing Boundary Value Analysis (BVA) in that it is more difficult and in most cases cannot be simply verified at the entry point, but usually requires checking some other system. 
For example: An application may ask for your Social Security Number. In BVA the application should check formats and semantics (is the value 9 digits long, not negative and not all 0's) for the data entered, but there are logic considerations also. SSNs are grouped and categorized. Is this person on a death file? Are they from a certain part of the country? 
Vulnerabilities related to business data validation is unique in that they are application specific and different from the vulnerabilities related to forging requests in that they are more concerned about logical data as opposed to simply breaking the business logic workflow. 
The front end and the back end of the application should be verifying and validating that the data it has, is using and is passing along is logically valid. Even if the user provides valid data to an application the business logic may make the application behave differently depending on data or circumstances. 

|

예제
============================================================================================

|

예제 1
-----------------------------------------------------------------------------------------

사용자가 카펫을 주문 할 수 있는 멀티미디어 전자상 거래 사이트를 운영한다고 가정합니다.
사용자는 카펫, 사이즈, 지불 방법을 선택하고, 프론트 엔드 어플리케이션은 모든 입력 정보가 정확하고 유효한지 확인합니다.
But, the business logic in the background has two paths, if the carpet is in stock it is directly shipped from your warehouse, but if it is out of stock in your warehouse a call is made to a partner's system and if they have it in-stock they will ship the order from their warehouse and reimbursed by them.
What happens if an attacker is able to continue a valid in-stock transaction and send it as out-of-stock to your partner? What happens if an attacker is able to get in the middle and send messages to the partner warehouse ordering carpet without payment? 

|

예제 2
-----------------------------------------------------------------------------------------

Many credit card systems are now downloading account balances nightly so the customers can check out more quickly for amounts under a certain value. The inverse is also true. I f I pay my credit card off in the morning I may not be able to use the available credit in the evening. Another example may be if I use my credit card at multiple locations very quickly it may be possible to exceed my limit if the systems are basing decisions on last night's data. 

|

테스트 방법
============================================================================================

일반 테스트 방법
-----------------------------------------------------------------------------------------

- 프로젝트 문서를 검토하고 데이터 입력 포인트 또는 시스템 또는 소프트웨어 사이에 입력할 수 있는 있는 포인트
를 찾아 테스트 공격 진행
- 어플리케이션 및 시스템에서 논리적으로 잘못된 데이터 입력 검색

|

구체적인 테스트 방법
-----------------------------------------------------------------------------------------

- "valid" 값 허용되는 지 확인하기 위해 어플리케이션의 프론트 엔트 GUI 기능 유효성 테스트 수행
- Using an intercepting proxy observe the HTTP POST/GET look ing for places that variables such as cost and quality are passed. Specifically, look for "hand-offs" between application/systems that may be possible injection of tamper points. 
- Once variables are found start interrogating the field with log ically "invalid" data, such as social security numbers or unique identifiers that do not exist or that do not fit the business logic. This testing verifies that the server functions properly and does not accept logically invalid data them. 

|

관련 테스트 케이스
============================================================================================

- 모든 입력 유효성 검사 테스트 케이스
- 계정 리스트 및 추측 가능한 사용자 테스트 (OTG-IDENT-004) 
- 세션 관리 스키마 우회 테스트 (OTG-SESS-001) 
- 노출 세션 변수 테스트 (OTG-SESS-004) 

|

Tools 
============================================================================================

- OWASP Zed Attack Proxy (ZAP)

|

References 
============================================================================================

- Beginning Microsoft Visual Studio LightSwitch Development - http://books.google.com/books?id=x76L_kaTgdEC&pg=PA280&lpg=PA280&dq=business+logic+example+valid+data+example&source=bl&ots=GOfQ-7f4Hu&sig=4jOejZVligZOrvjBFRAT4-jy8DI&hl=en&sa=X&ei=mydYUt6qE-OX54APu7IDgCQ&ved=0CFIQ6AEwBDgK#v=onepage&q=business%20logic%20example%20valid%20data%20example&f=false 

|

Remediation 
============================================================================================

The application/system must ensure that only "logically valid" data is accepted at all input and hand off points of the application or system and data is not simply trusted once it has entered the system. 

|