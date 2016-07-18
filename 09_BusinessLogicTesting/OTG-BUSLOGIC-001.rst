============================================================================================
OTG-BUSLOGIC-001
============================================================================================

|

비즈니스 로직 데이터 인증 테스트

|

개요
============================================================================================

The application must ensure that only logically valid data can be entered at the front end as well as directly to the server side of an application of system. Only verifying data locally may leave applications vulnerable to server injections through proxies or at handoffs with other systems. This is different from simply performing Boundary Value Analysis (BVA) in that it is more difficult and in most cases cannot be simply verified at the entry point, but usually requires checking some other system. 
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

The user selects their carpet, enters the size, makes the payment, and the front end application has verified that all entered information is correct and valid for contact information, size, make and color of the carpet. But, the business logic in the background has two paths, if the carpet is in stock it is directly shipped from your warehouse, but if it is out of stock in your warehouse a call is made to a partner's system and if they have it in-stock they will ship the order from their warehouse and reimbursed by them. What happens if an attacker is able to continue a valid in-stock transaction and send it as out-of-stock to your partner? What happens if an attacker is able to get in the middle and send messages to the partner warehouse ordering carpet without payment? 

|

예제 2
-----------------------------------------------------------------------------------------

Many credit card systems are now downloading account balances nightly so the customers can check out more quickly for amounts under a certain value. The inverse is also true. I f I pay my credit card off in the morning I may not be able to use the available credit in the evening. Another example may be if I use my credit card at multiple locations very quickly it may be possible to exceed my limit if the systems are basing decisions on last night's data. 

|

테스트 방법
============================================================================================

Generic Test Method 

- Review the project documentation and use exploratory testing looking for data entry points or hand off points between systems or software. 
- Once found try to insert logically invalid data into the application/system. 

Specific Testing Method: 

- Perform front-end GUI Functional Valid testing on the application to ensure that the only "valid" values are accepted. 
- Using an intercepting proxy observe the HTTP POST/GET look ing for places that variables such as cost and quality are passed. Specifically, look for "hand-offs" between application/systems that may be possible injection of tamper points. 
- Once variables are found start interrogating the field with log ically "invalid" data, such as social security numbers or unique identifiers that do not exist or that do not fit the business logic. This testing verifies that the server functions properly and does not accept logically invalid data them. 

|

Related Test Cases 
============================================================================================

- All Input Validation test cases 
- Testing for Account Enumeration and Guessable User Account (OTG-IDENT-004) 
- Testing for Bypassing Session Management Schema (OTG-SESS-001) 
- Testing for Exposed Session Variables (OTG-SESS-004) 

|

Tools 
============================================================================================

OWASP Zed Attack Proxy (ZAP) - https://www.owasp.org index.php/OWASP_Zed_Attack_Proxy_Project 

ZAP는 웹 어플리케이션에서 취약점을 찾아주는 통합 침투 테스트 툴로 사용하기 쉽습니다.
폭 넓은 보안 경력을 가진 사람들이 사용하도록 설계되었고, 개발자나 기능 테스터들에게 이상적입니다.

ZAP는 자동 스캐너뿐만 아니라 수동으로 보안 취약점을 찾을 수 있는 툴 셋도 제공합니다.

|

References 
============================================================================================

Beginning Microsoft Visual Studio LightSwitch Development - http://books.google.com/books?id=x76L_kaTgdEC&pg=PA280&lpg=PA280&dq=business+logic+example+valid+data+example&source=bl&ots=GOfQ-7f4Hu&sig=4jOejZVligZOrvjBFRAT4-jy8DI&hl=en&sa=X&ei=mydYUt6qE-OX54APu7IDgCQ&ved=0CFIQ6AEwBDgK#v=onep
age&q=business%20logic%20example%20valid%20data%20example&f=false 

|

Remediation 
============================================================================================

The application/system must ensure that only "logically valid" data is accepted at all input and hand off points of the application or system and data is not simply trusted once it has entered the system. 

|