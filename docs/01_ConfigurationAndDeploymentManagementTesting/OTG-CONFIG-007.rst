============================================================================================
OTG-CONFIG-007 (HTTP Strict Transport 보안 테스트)
============================================================================================

|

개요
============================================================================================

HSTS(HTTP Strict Transport Security) 헤더는 웹 사이트가 웹 브라우저와 통신해야 하는 메커니즘으로, 특정 도메인과 교환되는 모든 트래픽을 항상 https를 통해 전송해야하므로 정보가 암호화되지 않은 요청을 통해 전달되지 않도록 보호합니다.

이 보안 측정법의 중요성을 고려할 때 모든 데이터가 웹 브라우저에서 서버로 암호화되도록 하기 위해 웹 사이트가 이 HTTP 헤더를 사용하는지 확인하는 것이 중요합니다.

HTTP Strict Transport Security(HSTS) 기능을 사용하면 웹 응용 프로그램이 브라우저에 특정 응답 헤더를 사용하여 HTTP를 사용하여 지정된 도메인 서버에 연결하지 않아야 함을 알릴 수 있습니다. 
대신 HTTPS를 통해 사이트에 액세스하기위한 모든 연결 요청을 자동으로 설정해야합니다.

HTTP 엄격 전송 보안 헤더는 두 가지 지시문을 사용합니다.

- max-age: 브라우저가 모든 HTTP 요청을 HTTPS로 자동 변환해야하는 시간 (초)을 나타냅니다.
- includeSubDomains: 모든 웹 응용 프로그램의 하위 도메인에서 HTTPS를 사용해야 함을 나타냅니다.

다음은 HSTS 헤더 구현의 예입니다.

.. code-block:: console

    Strict-Transport-Security: max-age=60000;
    includeSubDomains

다음 보안 문제가 발생할 수 있는지 확인하려면 웹 응용 프로그램에서 이 헤더를 사용해야합니다.

- 공격자가 네트워크 트래픽을 스니핑하고 암호화되지 않은 채널을 통해 전송 된 정보에 액세스합니다.
- 공격자는 신뢰할 수없는 인증서를 수락하는 문제 때문에 중간 공격자를 악용합니다.
- 실수로 HTTPS 대신 HTTP를 사용하는 브라우저에서 주소를 입력 한 사용자 또는 실수로 http 프로토콜을 나타내는 웹 응용 프로그램의 링크를 클릭 한 사용자.

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

참고 문헌
============================================================================================

- OWASP HTTP Strict Transport Security: https://www.owasp.org/index.php/HTTP_Strict_Transport_Security
- OWASP Appsec Tutorial Series - Episode 4: Strict Transport Security: http://www.youtube.com/watch?v=zEV3HOuM_Vw
- HSTS Specification: http://tools.ietf.org/html/rfc6797

|