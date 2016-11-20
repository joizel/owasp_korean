============================================================================================
OTG-IDENT-005 (취약 또는 강제 사용자 이름 정책 테스트)
============================================================================================

|

개요 
============================================================================================

User account names are often highly structured (e.g. Joe Bloggs account name is jbloggs and Fred Nurks account name is fnurks) and valid account names can easily be guessed. 

|

테스트 목적 
============================================================================================

Determine whether a consistent account name structure renders the application vulnerable to account enumeration. Determine whether the application's error messages permit account enumeration. 

|

테스트 방법 
============================================================================================

- 계정 이름의 구조 결정
- 유효/유효하지 않은 계정 명에 어플리케이션 응답을 평가
- 유효한 계정 명을 리스트화 하기 위해 유효/유효하지 않은 계정 명에 다른 응답 사용
- 유효한 계정 명을 리스트화 하기 위해 계정명 사전 사용

|

Remediation 
============================================================================================

Ensure the application returns consistent generic error messages in response to invalid account name, password or other user credentials entered during the log in process. 

|

Authentication Testing
============================================================================================

Authentication (Greek: αυθεντικός = real or genuine, from 'authentes' = author ) is the act of establishing or confirming something (or someone) as authentic, that is, that claims made by or about the thing are true. Authenticating an object may mean confirming its provenance, whereas authenticating a person often consists of verifying her identity. Authentication depends upon one or more authentication factors. 

In computer security, authentication is the process of attempting to verify the digital identity of the sender of a communication. A common example of such a process is the log on process. Testing the authentication schema means understanding how the authentication process works and using that information to circumvent the authentication mechanism. 