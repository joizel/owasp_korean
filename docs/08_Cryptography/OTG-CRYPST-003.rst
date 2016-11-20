============================================================================================
OTG-CRYPST-003 (민감한 정보가 암호화되지 않은 채널에서 보내지는 경우 침투 테스트)
============================================================================================

|

개요
==========================================================================================

네트워크를 통해 민감한 데이터가 전송될 때 보호되어야만 합니다.
만약 데이터가 HTTPS 또는 다른 방법의 암호화로 전송된다면, 보호 메카니즘은 
제한이나 취약점이 없어야 합니다.

경험적으로 데이터가 저장될 때 보호되어야 하는 경우, 전송 시에도 역시 보호되어야 합니다.

민감한 데이터에 대한 몇가지 예제:

- 인증에 사용하는 정보들(Credentials, PINs, Session identifiers, Tokens, Cookies 등)
- 법률, 규정 또는 특정 조직의 정책에 의해 보호되는 정보 (Credit Cards, 고객 데이터)

만약 어플리케이션이 민감한 정보를 암호화되지 않은 채널을 통해 전송한다면, 보안 위험이 발생할 수 있습니다.


|

테스트 방법
==========================================================================================

보호되어야하는 다양한 형태의 정보들은 어플리케이션에서 평문으로 전송될 것입니다.
정보가 HTTPS 대신 HTTP를 통해 전송되거나 취약한 암호문을 사용한다면 확인이 가능합니다.


예제 1: HTTP를 통한 Basic 인증
-------------------------------------------------------------------------------------------

일반적인 예제는 HTTP를 통한 Basic 인증을 이용하는 것입니다.
Basic 인증을 사용할 때, 사용자 인증은 인코딩이 아닌 암호화가 되고, HTTP 헤더로 전송됩니다.

아래 에제에서 테스터는 curl을 사용하여 테스트 할 수 있습니다.

어플리케이션이 HTTPS 대신 기본 인증 및 HTTP를 사용하는 방법을 참고

.. code-block:: html

    curl -kis http://example.com/restricted/
    HTTP/1.1 401 Authorization Required
    Date: Fri, 01 Aug 2013 00:00:00 GMT
    WWW-Authenticate: Basic realm="Restricted Area"
    Accept-Ranges: bytes Vary:
    Accept-Encoding Content-Length: 162
    Content-Type: text/html

    <html><head><title>401 Authorization Required</title></
    head>
    <body bgcolor=white> <h1>401 Authorization Required</
    h1> Invalid login credentials! </body></html>

|

예제 2: HTTP를 통한 Form 기반 인증 수행
-------------------------------------------------------------------------------------------

또 다른 전형적인 예제는 HTTP를 통해 사용자 인증 정보를 전송하는 인증 form 입니다.

아래 예제에서는 form의 "action" 속성을 사용하여 HTTP를 보내는 걸 볼 수 있습니다.
웹 프록시로 HTTP 트래픽을 검사하여 이 이슈를 보는 것이 가능합니다.

.. code-block:: html

    <form action="http://example.com/login">
    <label for="username">User:</label>
    <input type="text" id="username" name="username" value=""/>
    <br />
    <label for="password">Password:</label>
    <input type="password" id="password" name="password" value=""/>
    <input type="submit" value="Login"/>
    </form>

|

예제 3: HTTP를 통한 Session ID를 포함한 Cookie 전송
-------------------------------------------------------------------------------------------

Session ID Cookie는 보호된 채널을 통해 전송되어야 합니다.
만약 cookie가 secure 플래그가 설정되어 있지 않으면, 어플리케이션에서 암호화되지 않은 것을 전송하는 것이 허용됩니다.

아래처럼 cookie 설정이 Secure 플래그 없이 수행되면, 프로세스 전체 로그는 HTTP에서 수행되고 HTTPS는 수행되지 않습니다.

.. code-block:: html

    https://secure.example.com/login

    POST /login HTTP/1.1
    Host: secure.example.com
    User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9;
    rv:25.0) Gecko/20100101 Firefox/25.0
    Accept: text/html,application/xhtml+xml,application/xml;
    q=0.9,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Referer: https://secure.example.com/
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 188

    HTTP/1.1 302 Found
    Date: Tue, 03 Dec 2013 21:18:55 GMT
    Server: Apache
    Cache-Control: no-store, no-cache, must-revalidate, maxage=
    0
    Expires: Thu, 01 Jan 1970 00:00:00 GMT
    Pragma: no-cache
    Set-Cookie: JSESSIONID=BD99F321233AF69593EDF52B123B5BDA;
    expires=Fri, 01-Jan-2014 00:00:00 GMT;
    path=/; domain=example.com; httponly
    Location: private/
    X-Content-Type-Options: nosniff
    X-XSS-Protection: 1; mode=block
    X-Frame-Options: SAMEORIGIN
    Content-Length: 0
    Keep-Alive: timeout=1, max=100
    Connection: Keep-Alive
    Content-Type: text/html
    ----------------------------------------------------------

    http://example.com/private

    GET /private HTTP/1.1
    Host: example.com
    User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9;
    rv:25.0) Gecko/20100101 Firefox/25.0
    Accept: text/html,application/xhtml+xml,application/xml;
    q=0.9,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Referer: https://secure.example.com/login
    Cookie: JSESSIONID=BD99F321233AF69593EDF52B123B5BDA;
    Connection: keep-alive

    HTTP/1.1 200 OK
    Cache-Control: no-store
    Pragma: no-cache
    Expires: 0
    Content-Type: text/html;charset=UTF-8
    Content-Length: 730
    Date: Tue, 25 Dec 2013 00:00:00 GMT
    ----------------------------------------------------------

|

Tools
==========================================================================================

- curl

|

References
==========================================================================================

OWASP Resources
-------------------------------------------------------------------------------------------

- OWASP Testing Guide - 취약한 SSL/TLS 암호, 불충분한 전송 계층 보호 침투 테스트 (OTG-CRYPST-001)
- OWASP TOP 10 2010 - Insufficient Transport Layer Protection
- OWASP TOP 10 2013 - Sensitive Data Exposure
- OWASP ASVS v1.1 - V10 Communication Security Verification Requirements
- OWASP Testing Guide - 쿠키 속성 테스트 (OTG-SESS-002)

|