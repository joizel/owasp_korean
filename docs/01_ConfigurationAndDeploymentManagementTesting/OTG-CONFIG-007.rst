============================================================================================
OTG-CONFIG-007 (HTTP Strict Transport 보안 테스트)
============================================================================================

|

개요
============================================================================================

The HTTP Strict Transport Security (HSTS) header is a mechanism that web sites have to communicate to the web browsers that all traffic exchanged with a given domain must always be sent over https, this will help protect the information from being passed over unencrypted requests.

Considering the importance of this security measure it is important to verify that the web site is using this HTTP header, in order to ensure that all the data travels encrypted from the web browser to the server.

The HTTP Strict Transport Security (HSTS) feature lets a web application to inform the browser, through the use of a special response header, that it should never establish a connection to the the specified domain servers using HTTP. 
Instead it should automatically establish all connection requests to access the site through HTTPS.

The HTTP strict transport security header uses two directives:

- max-age: to indicate the number of seconds that the browser
should automatically convert all HTTP requests to HTTPS.
- includeSubDomains: to indicate that all web application’s subdomains
must use HTTPS.

아래 HSTS 헤더 구현의 예제입니다.

.. code-block:: console

    Strict-Transport-Security: max-age=60000;
    includeSubDomains


웹 어플리케이션에 헤더 사용은 다음과 같은 보안 문제를 생성할 수 있는 지 찾아 확인해야 합니다.

- 공격자는 네트워크 트래픽을 스니핑하고 비보안 채널을 통해 전송하는 정보에 접근합니다.
- 공격자는 신뢰할 수 없는 인증서를 받아들이는 문제 때문에 MITM 공격을 악용할 수 있습니다.
- 사용자는 실수로 HTTPS 대신 HTTP를 넣어 브라우저에 주소를 입력한 사용자는 또는 잘못 표시된 HTTP 프로토콜인 웹 어플리케이션에 링크를 클릭하는 사용자

|

테스트 방법
============================================================================================

curl을 사용하여 HSTS 헤더의 존재 여부를 확인할 수 있습니다.

.. code-block:: console

    $ curl -s -D- https://domain.com/ | grep Strict

**예상 결과**

.. code-block:: console

    Strict-Transport-Security: max-age=...

|

References
============================================================================================

- OWASP HTTP Strict Transport Security: https://www.owasp.org/index.php/HTTP_Strict_Transport_Security
- OWASP Appsec Tutorial Series - Episode 4: Strict Transport Security: http://www.youtube.com/watch?v=zEV3HOuM_Vw
- HSTS Specification: http://tools.ietf.org/html/rfc6797

|