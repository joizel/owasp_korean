==========================================================================================
OTG-AUTHN-001
==========================================================================================

|

암호화된 채널에서 자격 증명 전송 테스트

|


개요
==========================================================================================

자격 증명 전송 테스트는 사용자의 인증 데이터가 악의적인 사용자에 의해 차단되는 것을 방지하기 위해 암호화 채널을 통해 전송되는 것을 확인하는 걸 의미합니다.
분석은 단순히 웹 어플리케이션에서 HTTPS와 같은 프로토콜을 사용하여 적절한 보안 조치를 취하는지, 데이터가 웹 브라우저로 부터 서버로 암호화되지 않은 상태로 이동하는지 이해하려고 노력하는데 초점을 맞추고 있습니다.

HTTPS 프로토콜은 전송되는 데이터를 암호화하고, 사용자가 원하는 위치를 향해 전송 될 수 있도록 TLS / SSL에 구축됩니다.
분명히, 트래픽이 암호화되어 있다는 사실은 완벽하게 안전하다는 것을 의미하지는 않습니다.
보안은 또한 사용된 암호화 알고리즘 및 어플리케이션을 사용한 키의 안정성에 의존하지만, 이러한 특정 항목은 이 섹션에서 설명되지 않을 것입니다.
TLS/SSL 채널의 안전성 테스트에 관한보다 상세한 설명은 취약한 SSL/TLS 테스트 챕터를 참조하십시오.

여기서는, 사용자가 순서대로 웹 양식에 넣어 데이터가 웹 사이트에 로그인 할 경우, 테스터는 공격자로부터 보호 보안 프로토콜을 사용하여 전송에 대한 이해를 하려고 노력합니다.
현재, 이 문제의 가장 일반적인 예는 웹 애플리케이션의 로그인 페이지입니다.
테스터는 사용자의 자격 증명이 암호화 된 채널을 통해 전송되는 것을 확인해야 합니다.

웹 사이트에 로그인하기 위해, 사용자는 일반적으로 POST 방식으로 웹 애플리케이션에 삽입 된 데이터를 송신하는 간단한 양식을 작성합니다.
이러한 데이터가 보안되지 않은 평문 형태로 데이터를 전송하는 HTTP 프로토콜을 사용하거나, 전송 중에 데이터를 암호화하는 HTTPS 프로토콜을 사용하여 전달 될 수 있습니다.
또한 사이트가 HTTP를 통한 액세스 로그인 페이지가 있지만, 실제로는 HTTPS를 통해 데이터를 전송하기도 합니다.
이 테스트는 공격자가 단순히 스니퍼 도구를 사용하여 네트워크를 스니핑하여 중요한 정보를 검색 할 수 있는지 확인하기 위한 것입니다.

|

How to Test
==========================================================================================

|

Black Box testing
-----------------------------------------------------------------------------------------

다음 예제에서 패킷 헤더를 캡쳐하고 검사하기 위하여 WebScarab을 사용할 것입니다.

**Example 1: HTTP를 통해 POST 메소드로 데이터 전송**

User, Pass 필드로 form이 구성된 로그인 페이지가 있다고 가정합니다.
그리고, 어플리케이션 접근 및 인증을 위한 입력 버튼이 있습니다.
만약 WebScarab로 우리의 요청 헤더를 본다면, 아래와 같을 것 입니다.

.. code-block:: console

    POST http://www.example.com/AuthenticationServlet
    HTTP/1.1
    Host: www.example.com
    User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; it;
    rv:1.8.1.14) Gecko/20080404
    Accept: text/xml,application/xml,application/xhtml+xml
    Accept-Language: it-it,it;q=0.8,en-us;q=0.5,en;q=0.3
    Accept-Encoding: gzip,deflate
    Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
    Keep-Alive: 300
    Connection: keep-alive
    Referer: http://www.example.com/index.jsp
    Cookie: JSESSIONID=LVrRRQQXgwyWpW7QMnS49vtW1yBdqn98CGlkP4jTvVCGdyPkmn3S!
    Content-Type: application/x-www-form-urlencoded
    Content-length: 64
    
    delegated_service=218&User=test&Pass=test&Submit=SUBMIT

해당 예제를 통해 테스터는 HTTP POST를 사용하여 www.example.com/AuthenticationServlet 페이지로 데이터를 요청하는 것을 이해할 수 있습니다.
그래서, 데이터는 암호없이 전송되고, 악의적인 사용자는 Wireshark와 같은 툴으로 
네트워크 스니핑으로 사용자명과 패스워드를 훔칠 수 있습니다.

**Example 2: HTTPS를 통해 POST 메소드로 데이터 전송**

보내는 데이터가 암호화된 HTTPS 프로토콜을 사용하는 웹 어플리케이션이라고 가정합니다.
이 경우 웹 어플리케이션에 로그인 할 때, 아래와 같을 것 입니다.

.. code-block:: console

    POST https://www.example.com:443/cgi-bin/login.cgi HTTP/1.1
    Host: www.example.com
    User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; it;
    rv:1.8.1.14) Gecko/20080404
    Accept: text/xml,application/xml,application/xhtml+xml,text/html
    Accept-Language: it-it,it;q=0.8,en-us;q=0.5,en;q=0.3
    Accept-Encoding: gzip,deflate
    Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
    Keep-Alive: 300
    Connection: keep-alive
    Referer: https://www.example.com/cgi-bin/login.cgi
    Cookie: language=English;
    Content-Type: application/x-www-form-urlencoded
    Content-length: 50

    Command=Login&User=test&Pass=test

HTTPS 프로토콜을 사용하여 www.example.com:443/cgi-bin/login.cgi으로 접근하려는
요청을 볼 수 있습니다. 자격 증명은 암호화 채널을 사용하여 보내지며,
악의적인 사용자는 스니퍼를 사용하여 자격 증명을 읽을 수 없습니다.

**Example 3: HTTP를 통해 연결된 페이지에 HTTPS를 통해 POST 메소드로 데이터 전송**

이제 HTTP를 통해 통신하는 데 인증 시에만 HTTPS를 사용하는 웹 페이지를 가정합니다.
예를 들어, 식별 없이 다양한 정보와 공개적으로 사용할 수 있는 서비스를 제공하는 큰 회사의 포털에 있을 때 이러한 상황은 발생하지만, 이 사이트는 사용자가 로그인 홈 페이지에서 액세스 할 수있는 전용 섹션이 있습니다.
그래서 로그인시에는 요청 헤더가 아래와 같을 것입니다.

.. code-block:: console

    POST https://www.example.com:443/login.do HTTP/1.1
    Host: www.example.com
    User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; it;
    rv:1.8.1.14) Gecko/20080404
    Accept: text/xml,application/xml,application/xhtml+xml,text/html
    Accept-Language: it-it,it;q=0.8,en-us;q=0.5,en;q=0.3
    Accept-Encoding: gzip,deflate
    Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
    Keep-Alive: 300
    Connection: keep-alive
    Referer: http://www.example.com/homepage.do
    Cookie: SERVTIMSESSIONID=s2JyLkvDJ9ZhX3yr5BJ3DFLkdphH-
    0QNSJ3VQB6pLhjkW6F
    Content-Type: application/x-www-form-urlencoded
    Content-length: 45
    
    User=test&Pass=test&portal=ExamplePortal

HTTPS 프로토콜을 사용하여 www.example.com:443/login.do으로 접근하려는 요청을 볼 수 있습니다. 그러나  Referer 를 보면, www.example.com/homepage.do라는 HTTP로 되어있습니다. HTTPS로 데이터를 보낼 때, SSLStrip 공격으로 확인할 수 있습니다.

**Example 4: HTTPS를 통해 GET 메소드로 데이터 전송**

마지막 예제에서는 어플리케이션이 GET 메소드를 사용하여 데이터를 전송한다 가정합니다.
이 메소드는 데이터가 URL에 일반 텍스트로 표시되기 때문에, 사용자 명 및 패스워드 같은 민감한 데이터를 전송하는 형태로 사용하지 않습니다.
예를 들어, 요청된 URL은 허가되지 않은 사람들이 서버 로그에서 중요한 데이터를 검색할 수 있습니다.

.. code-block:: console

    GET https://www.example.com/success.html?user=test&-
    pass=test HTTP/1.1
    Host: www.example.com
    User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; it;
    rv:1.8.1.14) Gecko/20080404
    Accept: text/xml,application/xml,application/xhtml+xml,-
    text/html
    Accept-Language: it-it,it;q=0.8,en-us;q=0.5,en;q=0.3
    Accept-Encoding: gzip,deflate
    Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
    Keep-Alive: 300
    Connection: keep-alive
    Referer: https://www.example.com/form.html
    If-Modified-Since: Mon, 30 Jun 2008 07:55:11 GMT
    If-None-Match: “43a01-5b-4868915f”

사용자는 데이터가 이전 요청의 본문의 URL에서 텍스트로 전송되어 있지 않은 것을 알 수 있습니다.
그러나, SSL/TLS 가 HTTP 보다 더 낮은 레벨인 레벨 5 프로토콜인 걸 고려해야합니다. 그래서 전체 HTTP 패킷은 여전히 스니퍼를 사용하여 악의적인 사용자에 읽을 수 없도록 URL이 암호화됩니다.

앞서 언급 한 바와 같이 그럼에도 불구하고, 상기 URL에 포함 된 정보는 프록시와 웹 서버 로그와 같은 다양한 위치에 저장 될 수 있기 때문에, 웹 애플리케이션에 중요한 데이터를 보낼 때는 GET 메소드를 사용하는 것이 좋은 방법이 아닙니다.


|

Gray Box testing
-----------------------------------------------------------------------------------------

Speak with the developers of the web application and try to
understand if they are aware of the differences between HTTP
and HTTPS protocols and why they should use HTTPS for transmitting
sensitive information. Then, check with them if HTTPS
is used in every sensitive request, like those in log in pages, to
prevent unauthorized users to intercept the data.


|

Tools
==========================================================================================

- WebScarab
- OWASP Zed Attack Proxy (ZAP)

|


References
==========================================================================================

Whitepapers
-----------------------------------------------------------------------------------------

- HTTP/1.1: Security Considerations - http://www.w3.org/Protocols/rfc2616/rfc2616-sec15.html
- SSL is not about encryption

|