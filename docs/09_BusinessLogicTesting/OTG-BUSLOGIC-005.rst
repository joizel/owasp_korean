============================================================================================
OTG-BUSLOGIC-005 (기능 사용 제한 시간 테스트)
============================================================================================

|

개요
============================================================================================

어플리케이션에서 해결해야하는 문제 중 대부분은 기능 사용 또는 실행에 대한 시간 제한이 요구됩니다.


Applications must be "smart enough" to not allow the user to exceed their limit on the use of these functions since in many cases each time the function is used the user may gain some type of benefit that must be accounted for to properly compensate the owner. For example: an eCommerce site may only allow a users apply a discount once per transaction, or some applications may be on a subscription plan and only allow users to download three complete documents monthly. 

Vulnerabilities related to testing for the function limits are application specific and misuse cases must be created that strive to exercise parts of the application/functions/ or actions more than the allowable number of times. Attackers may be able to circumvent the business logic and execute a function more times than "allowable" exploiting the application for personal gain. 

|

예제
============================================================================================

Suppose an eCommerce site allows users to take advantage of any one of many discounts on their total purchase and then proceed to checkout and tendering. What happens of the attacker navigates back to the discounts page after taking and applying the one "allowable" discount? Can they take advantage of another discount? Can they take advantage of the same discount multiple times? 

|

테스트 방법
============================================================================================

- Review the project documentation and use exploratory testing looking for functions or features in the application or system that should not be executed more that a single time or specified number of times during the business logic workflow. 
- For each of the functions and features found that should only be executed a single time or specified number of times during the business logic workflow, develop abuse/misuse cases that may allow a user to execute more than the allowable number of times. For example, can a user navigate back and forth through the pages multiple times executing a function that should only execute once? or can a user load and unload shopping carts allowing for additional discounts. 

|

관련 테스트 케이스
============================================================================================

- 계정 리스트 및 추측 가능한 사용자 테스트 (OTG-IDENT-004) 
- 취약한 잠금 매카니즘 테스트 (OTG-AUTHN-003) 

|

References 
============================================================================================

- InfoPath Forms Services business logic exceeded the maximum limit of operations Rule - http://mpwiki.viacode.com/default.aspx?g=posts&t=115678 

|

Remediation 
============================================================================================

The application should have checks to ensure that the business logic is being followed and that if a function/action can only be executed a certain number of times, when the limit is reached the user can no longer execute the function. To prevent users from using a function over the appropriate number of times the application may use mechanisms such as cookies to keep count or through sessions not allowing users to access to execute the function additional times. 

|