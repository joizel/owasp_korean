==========================================================================================
OTG-INFO-002 (웹서버 핑거프린팅)
==========================================================================================

|

개요
==========================================================================================

웹 서버 핑거프린팅은 침투 테스터를 위한 중요한 작업 입니다.

웹 서버 형태와 버전을 아는 것은 테스터에게는 사용할 수 있는 취약점과 적절한 익스플로잇을 
정의할 수 있습니다. 최근에는 웹 서버의 벤더와 버전이 여러개 존재합니다.
테스트를 할 때 웹 서버 형태를 아는 것은 테스트 과정에서 상당히 도움이 됩니다.

웹 서버의 각각의 형태는 특정 명령에 응답하는 데이터베이스에 대해 정보를 보유하고 있어, 
응답 데이터 분석을 통해 알려진 시그너처와 비교해 볼 수 있습니다.

웹 서버를 식별하기 위한 여러가지 명령들이 있으며, 같은 명령에 유사하게 반응 하는 다른 버전들도 있습니다.
거의 서로 다른 버전의 모든 HTTP 명령에 동일한 반응하지 않습니다. 여러 명령을 전송함으로써, 테스터는 추측의 정확도를 높일 수 있습니다.

|

테스트 목적
==========================================================================================

침투 테스트 시 사용할 취약점과 적합한 익스플로잇을 찾기하기 위해 실행중인 웹 서버의 버전과 형식을 확인합니다.

|

테스트 방법
==========================================================================================

Black Box testing
-----------------------------------------------------------------------------------------

웹 서버를 식별하기 위한 가장 간단하고 기본적인 방법은 HTTP 응답 헤더에서 Server 필드를 확인하는 것입니다.
NetCat을 이용하여 다음과 같이 사용합니다.

.. code-block:: console

    $ nc 202.41.76.251 80
    HEAD / HTTP/1.0
    HTTP/1.1 200 OK
    Date: Mon, 16 Jun 2003 02:53:29 GMT
    Server: Apache/1.3.3 (Unix) (Red Hat/Linux)
    Last-Modified: Wed, 07 Oct 1998 11:18:14 GMT
    ETag: "1813-49b-361b4df6"
    Accept-Ranges: bytes
    Content-Length: 1179
    Connection: close
    Content-Type: text/html


Server 필드로 부터 Apache 1.3.3 이라는 것을 확인할 수 있으며, Linux 운영체제인 것도 확인 가능합니다.
아래 다른 서버에 대한 예제를 4개 더 보도록 하겠습니다.

Apache 1.3.23 서버

.. code-block:: console

    HTTP/1.1 200 OK
    Date: Sun, 15 Jun 2003 17:10: 49 GMT
    Server: Apache/1.3.23
    Last-Modified: Thu, 27 Feb 2003 03:48: 19 GMT
    ETag: 32417-c4-3e5d8a83
    Accept-Ranges: bytes
    Content-Length: 196
    Connection: close
    Content-Type: text/HTML

Microsoft IIS 5.0 서버

.. code-block:: console

    HTTP/1.1 200 OK
    Server: Microsoft-IIS/5.0
    Expires: Yours, 17 Jun 2003 01:41: 33 GMT
    Date: Mon, 16 Jun 2003 01:41: 33 GMT
    Content-Type: text/HTML
    Accept-Ranges: bytes
    Last-Modified: Wed, 28 May 2003 15:32: 21 GMT
    ETag: b0aac0542e25c31: 89d
    Content-Length: 7369

Netscape Enterprise 4.1 서버

.. code-block:: console

    HTTP/1.1 200 OK
    Server: Netscape-Enterprise/4.1
    Date: Mon, 16 Jun 2003 06:19: 04 GMT
    Content-type: text/HTML
    Last-modified: Wed, 31 Jul 2002 15:37: 56 GMT
    Content-length: 57
    Accept-ranges: bytes
    Connection: close

SunONE 6.1 서버

.. code-block:: console

    HTTP/1.1 200 OK
    Server: Sun-ONE-Web-Server/6.1
    Date: Tue, 16 Jan 2007 14:53:45 GMT
    Content-length: 1186
    Content-type: text/html
    Date: Tue, 16 Jan 2007 14:50:31 GMT
    Last-Modified: Wed, 10 Jan 2007 09:58:26 GMT
    Accept-Ranges: bytes
    Connection: close

위의 테스트 방법은 정확성에 문제가 있을 수 있습니다. 
서버 베너 문자열을 수정하거나 난독화 시키는 방법이 있기 때문입니다. 

.. code-block:: console

    403 HTTP/1.1 Forbidden
    Date: Mon, 16 Jun 2003 02:41: 27 GMT
    Server: Unknown-Webserver/1.0
    Connection: close
    Content-Type: text/HTML; charset=iso-8859-1



위의 경우 응답 헤더에 Server 필드가 난독화되어 있어, 침투 테스터는 웹 서버의 기본 정보를 확인할 수 없습니다.

|

프로토콜 행위
-----------------------------------------------------------------------------------------

더 정교한 기술은 이용하는 여러 웹 서버의 다양한 특성을 이용하는 것입니다. 
아래 침투 테스터가 침투시 웹 서버 형태를 추론할 수 있는 또 다른 방법을 설명합니다.

|

HTTP 헤더 필드 순서
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

HTTP 응답 헤더 순서를 관찰하여 웹 서버를 구분할 수 있습니다.
아래 응답 헤더 부분을 보면 Apache, Netscape Enterprise, IIS가 Date 필드와 
Server 필드 사이의 순서가 다른 것을 확인할 수 있습니다.

Apache 1.3.23 서버

.. code-block:: console

    $ nc apache.example.com 80
    HEAD / HTTP/1.0
    HTTP/1.1 200 OK
    Date: Sun, 15 Jun 2003 17:10: 49 GMT
    Server: Apache/1.3.23
    Last-Modified: Thu, 27 Feb 2003 03:48: 19 GMT
    ETag: 32417-c4-3e5d8a83
    Accept-Ranges: bytes
    Content-Length: 196
    Connection: close
    Content-Type: text/HTML

Microsoft IIS 5.0 서버

.. code-block:: console

    $ nc iis.example.com 80
    HEAD / HTTP/1.0
    HTTP/1.1 200 OK
    Server: Microsoft-IIS/5.0
    Content-Location: http://iis.example.com/Default.htm
    Date: Fri, 01 Jan 1999 20:13: 52 GMT
    Content-Type: text/HTML
    Accept-Ranges: bytes
    Last-Modified: Fri, 01 Jan 1999 20:13: 52 GMT
    ETag: W/e0d362a4c335be1: ae1
    Content-Length: 133 

Netscape Enterprise 4.1 서버

.. code-block:: console

    $ nc netscape.example.com 80
    HEAD / HTTP/1.0
    HTTP/1.1 200 OK
    Server: Netscape-Enterprise/4.1
    Date: Mon, 16 Jun 2003 06:01: 40 GMT
    Content-type: text/HTML
    Last-modified: Wed, 31 Jul 2002 15:37: 56 GMT
    Content-length: 57
    Accept-ranges: bytes
    Connection: close

SunONE 6.1 서버

.. code-block:: console

    $ nc sunone.example.com 80
    HEAD / HTTP/1.0
    HTTP/1.1 200 OK
    Server: Sun-ONE-Web-Server/6.1
    Date: Tue, 16 Jan 2007 15:23:37 GMT
    Content-length: 0
    Content-type: text/html
    Date: Tue, 16 Jan 2007 15:20:26 GMT
    Last-Modified: Wed, 10 Jan 2007 09:58:26 GMT
    Connection: close



|

악의적 리퀘스트 테스트
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

또 다른 유용한 침투 테스트는 악의적인 리퀘스트 또는 존재하지 않는 페이지 요청을 서버에 보내는 것입니다.
아래를 확인해보면 서버마다 다른 응답 헤더를 보내는 것을 확인할 수 있습니다.

Apache 1.3.23 서버

.. code-block:: console

    $ nc apache.example.com 80
    GET / HTTP/3.0
    HTTP/1.1 400 Bad Request
    Date: Sun, 15 Jun 2003 17:12: 37 GMT
    Server: Apache/1.3.23
    Connection: close
    Transfer: chunked
    Content-Type: text/HTML; charset=iso-8859-1

    $ nc apache.example.com 80
    GET / JUNK/1.0
    HTTP/1.1 200 OK
    Date: Sun, 15 Jun 2003 17:17: 47 GMT
    Server: Apache/1.3.23
    Last-Modified: Thu, 27 Feb 2003 03:48: 19 GMT
    ETag: 32417-c4-3e5d8a83
    Accept-Ranges: bytes
    Content-Length: 196
    Connection: close
    Content-Type: text/HTML

Microsoft IIS 5.0 서버

.. code-block:: console

    $ nc iis.example.com 80
    GET / HTTP/3.0
    HTTP/1.1 200 OK
    Server: Microsoft-IIS/5.0
    Content-Location: http://iis.example.com/Default.htm
    Date: Fri, 01 Jan 1999 20:14: 02 GMT
    Content-Type: text/HTML
    Accept-Ranges: bytes
    Last-Modified: Fri, 01 Jan 1999 20:14: 02 GMT
    ETag: W/e0d362a4c335be1: ae1
    Content-Length: 133

    $ nc iis.example.com 80
    GET / JUNK/1.0
    HTTP/1.1 400 Bad Request
    Server: Microsoft-IIS/5.0
    Date: Fri, 01 Jan 1999 20:14: 34 GMT
    Content-Type: text/HTML
    Content-Length: 87

Netscape Enterprise 4.1 서버

.. code-block:: console

    $ nc netscape.example.com 80
    GET / HTTP/3.0 

    HTTP/1.1 505 HTTP Version Not Supported
    Server: Netscape-Enterprise/4.1
    Date: Mon, 16 Jun 2003 06:04: 04 GMT
    Content-length: 140
    Content-type: text/HTML
    Connection: close

    $ nc netscape.example.com 80
    GET / JUNK/1.0
    <HTML><HEAD><TITLE>Bad request</TITLE></HEAD>
    <BODY><H1>Bad request</H1>
    Your browser sent to query this server could not understand.
    </BODY></HTML>

SunONE 6.1 서버

.. code-block:: console

    $ nc sunone.example.com 80
    GET / HTTP/3.0
    HTTP/1.1 400 Bad request
    Server: Sun-ONE-Web-Server/6.1
    Date: Tue, 16 Jan 2007 15:25:00 GMT
    Content-length: 0
    Content-type: text/html
    Connection: close

    $ nc sunone.example.com 80
    GET / JUNK/1.0
    <HTML><HEAD><TITLE>Bad request</TITLE></HEAD>
    <BODY><H1>Bad request</H1>
    Your browser sent a query this server could not understand.
    </BODY></HTML>

|


Tools
==========================================================================================

- httprint: http://net-square.com/httprint.html
- httprecon: http://www.computec.ch/projekte/httprecon/
- Netcraft: http://www.netcraft.com
- Desenmascarame: http://desenmascara.me


Automated Testing
-----------------------------------------------------------------------------------------

웹 서버 헤더 분석과 배너 수집을 위해 수동으로 의존하는 것보다, 동일한 결과를 얻기 위해 자동화된 도구를 사용하는 것이 좋습니다.
웹 서버를 정확하게 핑거프린트하기 위해 수행하는 테스트 방법이 여러가지 있는데, 그 중 "httprint" 라는 툴이 있습니다.
httprint는 웹 서버 버전과 형태를 인식하기 위해 시그너처 데이터베이스를 사용합니다.

|

Online Testing
-----------------------------------------------------------------------------------------

온라인 툴은 직접 타겟 웹 사이트에 연결하기를 원치 않을 경우 사용할 수 있습니다.
Netcraft라는 툴은 웹 서버 서버 가동 시간, Netblock 소유자, 웹 서버 관련 히스토리 등의 정보를 확인할 수 있습니다.
OWASP Unmaskme 프로젝트는 추출한 모든 웹 메타 데이터의 전반적인 해석과 모든 웹 사이트의 핑거프린트를 수행하는 또 다른 온라인 도구가 될 것으로 예상됩니다.

|

References
==========================================================================================

- Saumil Shah: "An Introduction to HTTP fingerprinting": http://www.net-square.com/httprint_paper.html
- Anant Shrivastava: "Web Application Finger Printing": http://anantshri.info/articles/web_app_finger_printing.html

|

Remediation
==========================================================================================

강화된 리버스 프록시 뒤에 표현 계층 웹 서버를 보호합니다.
표현 계층 웹 서버 헤더를 난독화합니다.

- Apache
- IIS

|