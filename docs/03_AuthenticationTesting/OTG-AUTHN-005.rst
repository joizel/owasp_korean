==========================================================================================
OTG-AUTHN-005 (패스워드 기억 취약점 테스트)
==========================================================================================

|

개요
==========================================================================================

Browsers will sometimes ask a user if they wish to remember the
password that they just entered. The browser will then store the
password, and automatically enter it whenever the same authentication
form is visited. This is a convenience for the user.
Additionally some websites will offer custom “remember me”
functionality to allow users to persist log ins on a specific client
system.

Having the browser store passwords is not only a convenience
for end-users, but also for an attacker. If an attacker can gain access
to the victim’s browser (e.g. through a Cross Site Scripting
attack, or through a shared computer), then they can retrieve the
stored passwords. It is not uncommon for browsers to store these
passwords in an easily retrievable manner, but even if the browser
were to store the passwords encrypted and only retrievable
through the use of a master password, an attacker could retrieve
the password by visiting the target web application’s authentication
form, entering the victim’s username, and letting the browser
to enter the password.

Additionally where custom “remember me” functions are put in
place weaknesses in how the token is stored on the client PC (for
example using base64 encoded credentials as the token) could
expose the users passwords. Since early 2014 most major browsers
will override any use of autocomplete=”off” with regards to
password forms and as a result previous checks for this are not
required and recommendations should not commonly be given for
disabling this feature. However this can still apply to things like
secondary secrets which may be stored in the browser inadvertently.

|

테스트 방법
==========================================================================================

- 쿠키에 저장되어 있는 패스워드 검색
  어플리케이션에 의해 저장된 쿠키 검사
  자격 증명이 일반 텍스트로 저장되지 않고, 해쉬되어 있는지 확인
- 해싱 메커니즘 검사: 만약 잘 알려진 알고리즘을 사용한 경우 여러 사용자명 테스트를 통해 쉽게 유추가 가능한지 확인
- 자격 증명이 로그 중 전송되는 부분만 확인. 어플리케이션에서 모든 요청과 함께 보내지지 않는지 확인
- 민감한 form 필드 고려

|

References
==========================================================================================

Ensure that no credentials are stored in clear text or are easily retrievable
in encoded or encrypted forms in cookies.

|