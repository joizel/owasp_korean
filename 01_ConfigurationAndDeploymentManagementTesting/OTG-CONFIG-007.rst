============================================================================================
OTG-CONFIG-007 (HTTP Strict Transport 보안 테스트)
============================================================================================

|

개요
============================================================================================

The HTTP Strict Transport Security (HSTS) header is a mechanism
that web sites have to communicate to the web browsers that all
traffic exchanged with a given domain must always be sent over
https, this will help protect the information from being passed
over unencrypted requests.

Considering the importance of this security measure it is important
to verify that the web site is using this HTTP header, in order to
ensure that all the data travels encrypted from the web browser
to the server.

The HTTP Strict Transport Security (HSTS) feature lets a web
application to inform the browser, through the use of a special
response header, that it should never establish a connection to
the the specified domain servers using HTTP. Instead it should
automatically establish all connection requests to access the site
through HTTPS.

The HTTP strict transport security header uses two directives:

- max-age: to indicate the number of seconds that the browser
should automatically convert all HTTP requests to HTTPS.
- includeSubDomains: to indicate that all web application’s subdomains
must use HTTPS.

Here’s an example of the HSTS header implementation:

.. code-block:: console

    Strict-Transport-Security: max-age=60000;
    includeSubDomains


The use of this header by web applications must be checked to
find if the following security issues could be produced:

- Attackers sniffing the network traffic and accessing
the information transferred through an unencrypted channel.

- Attackers exploiting a man in the middle attack because of the
 problem of accepting certificates that are not trusted.

- Users who mistakenly entered an address in the browser putting
HTTP instead of HTTPS, or users who click on a link in a web
application which mistakenly indicated the http protocol.
protocol

|

테스트 방법
============================================================================================

curl을 사용하여 HSTS 헤더의 존재 여부를 확인할 수 있습니다.

.. code-block:: console

    $ curl -s -D- https://domain.com/ | grep Strict

예상 결과:

.. code-block:: console

    Strict-Transport-Security: max-age=...

|

References
============================================================================================

- OWASP HTTP Strict Transport Security: https://www.owasp.org/index.php/HTTP_Strict_Transport_Security
- OWASP Appsec Tutorial Series - Episode 4: Strict Transport Security: http://www.youtube.com/watch?v=zEV3HOuM_Vw
- HSTS Specification: http://tools.ietf.org/html/rfc6797

|