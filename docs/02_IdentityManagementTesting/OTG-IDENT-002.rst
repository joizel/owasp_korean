============================================================================================
OTG-IDENT-002 (사용자 등록 처리 테스트)
============================================================================================

|

개요
============================================================================================

Some websites offer a user registration process that automates (or semi-automates) the provisioning of system access to users. The identity requirements for access vary from positive identification to none at all, depending on the security requirements of the system. Many public applications completely automate the registration and provisioning process because the size of the user base makes it impossible to manage manually. However, many corporate applications will provision users manually, so this test case may not apply. 

|

테스트 목적
============================================================================================

- 사용자 등록에 대한 신원 요구 사항이 비즈니스 및 보안 요구 사항과 일치하는지 확인
- 등록 절차 확인

|

테스트 방법
============================================================================================

사용자 등록에 대한 신원 요구 사항이 비즈니스 및 보안 요구 사항과 일치하는지 확인
------------------------------------------------------------------------------------------------

1. Can anyone register for access? 
2. Are registrations vetted by a human prior to provisioning, or are they automatically granted if the criteria are met? 
3. Can the same person or identity register multiple times? 
4. Can users register for different roles or permissions? 
5. What proof of identity is required for a registration to be successful? 
6. Are registered identities verified? 

등록 절차 확인
-----------------------------------------------------------------------------------------

[1] Can identity information be easily forged or faked? 
[2] Can the exchange of identity information be manipulated during registration? 

**예제**

In the WordPress example below, the only identification requirement is an email address that is accessible to the registrant. 


In contrast, in the Google example below the identification requirements include name, date of birth, country, mobile phone number, email address and CAPTCHA response. While only two of these can be verified (email address and mobile number), the identification requirements are stricter than WordPress. 

|

Tools 
============================================================================================

A HTTP proxy can be a useful tool to test this control. 

|

References 
============================================================================================

User Registration Design 

|

Remediation 
============================================================================================

Implement identification and verification requirements that correspond to the security requirements of the information the credentials protect. 

|