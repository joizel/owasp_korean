============================================================================================
OTG-SESS-003 (쿠키 고정 테스트)
============================================================================================

|

개요
============================================================================================

When an application does not renew its session cookie(s) after a successful user authentication, it could be possible to find a session fixation vulnerability and force a user to utilize a cookie known by the attacker. 
In that case, an attacker could steal the user session (session hijacking).

Session fixation vulnerabilities occur when:

- A web application authenticates a user without first invalidating the existing session ID, thereby continuing to use the session ID already associated with the user.
- An attacker is able to force a known session ID on a user so that, once the user authenticates, the attacker has access to the authenticated session.

In the generic exploit of session fixation vulnerabilities, an attacker creates a new session on a web application and records the associated session identifier. 
The attacker then causes the victim to authenticate against the server using the same session identifier, giving the attacker access to the user’s account through the active session.

Furthermore, the issue described above is problematic for sites that issue a session identifier over HTTP and then redirect the user to a HTTPS log in form. 
If the session identifier is not reissued upon authentication, the attacker can eavesdrop and steal the identifier and then use it to hijack the session.

|

테스트 방법
============================================================================================

Black Box Testing
-----------------------------------------------------------------------------------

- 요청

.. code-block:: html

    GET www.example.com

- 응답

.. code-block:: html

    HTTP/1.1 200 OK
    Date: Wed, 14 Aug 2008 08:45:11 GMT
    Server: IBM_HTTP_Server
    Set-Cookie: JSESSIONID=0000d8eyYq3L0z2fgq10m4vrt4:-1;
    Path=/; secure
    Cache-Control: no-cache=”set-cookie,set-cookie2”
    Expires: Thu, 01 Dec 1994 16:00:00 GMT
    Keep-Alive: timeout=5, max=100
    Connection: Keep-Alive
    Content-Type: text/html;charset=Cp1254
    Content-Language: en-US

- 어플리케이션은 클라이언트로 JSESSIONID=0000d-8eyYq3L0z2fgq10m4v-rt4:-1 세션 설정

- POST 요청

.. code-block:: html

    POST https://www.example.com/authentication.php HTTP/1.1
    Host: www.example.com
    User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; it;
    rv:1.8.1.16) Gecko/20080702 Firefox/2.0.0.16
    Accept: text/xml,application/xml,application/xhtml+xml,text/
    html;q=0.9,text/plain;q=0.8,image/png,*/*;q=0.5
    Accept-Language: it-it,it;q=0.8,en-us;q=0.5,en;q=0.3
    Accept-Encoding: gzip,deflate
    Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
    Keep-Alive: 300
    Connection: keep-alive
    Referer: http://www.example.com
    Cookie: JSESSIONID=0000d8eyYq3L0z2fgq10m4v-rt4:-1
    Content-Type: application/x-www-form-urlencoded
    Content-length: 57

    Name=Meucci&wpPassword=secret!&wpLoginattempt=Log+in

- 응답

.. code-block:: html

    HTTP/1.1 200 OK
    Date: Thu, 14 Aug 2008 14:52:58 GMT
    Server: Apache/2.2.2 (Fedora)
    X-Powered-By: PHP/5.1.6
    Content-language: en
    Cache-Control: private, must-revalidate, max-age=0
    X-Content-Encoding: gzip
    Content-length: 4090
    Connection: close
    Content-Type: text/html; charset=UTF-8
    ...
    HTML data
    ...


- 세션 하이재킹 가능 확인

**예상 결과**

The tester can send a valid session identifier to a user (possibly using a social engineering trick), wait for them to authenticate, and subsequently verify that privileges have been assigned to this cookie.

|

Gray Box Testing
--------------------------------------------------------------------------------

개발자들과 이야기 하고 세션 토큰이 사용자 인증이 성공한 후 새로운 세션 토큰을 구현하는 경우에 대해 이해합니다.

**예상 결과** 

어플리케이션은 사용자를 인증하기 전에 항상 먼저 기존 session ID를 무효화하고, 인증에 성공하면 다른 sessionID를 제공합니다.

|

Tools
============================================================================================

- Hijack: http://yehg.net/lab/pr0js/files.php/jhijackv0.2beta.zip
- OWASP WebScarab: OWASP_WebScarab_Project

|

References
============================================================================================

Whitepapers
--------------------------------------------------------------------------------

- Session Fixation
- ACROS Security: http://www.acrossecurity.com/papers/session_fixation.pdf
- Chris Shiflett: http://shiflett.org/articles/session-fixation

|