============================================================================================
OTG-INFO-002
============================================================================================

|

웹서버 핑거 프린팅

|

Summary
============================================================================================

Web server fingerprinting is a critical task for the penetration tester.
Knowing the version and type of a running web server allows testers to determine known vulnerabilities and the appropriate exploits to use during testing.

There are several different vendors and versions of web servers on the market today. 
Knowing the type of web server that is being tested significantly helps in the testing process and can also change the course of the test.
This information can be derived by sending the web server specific commands and analyzing the output, as each version of web server software may respond differently to these commands. By knowing how each type of web server responds to specific commands and keeping this information in a web server fingerprint database, a penetration tester can send these commands to the web server, analyze the response, and compare it to the database of known signatures.
Please note that it usually takes several different commands to accurately identify the web server, as different versions may react similarly to the same command. 
Rarely do different versions react the same to all HTTP commands. So by sending several different commands, the
tester can increase the accuracy of their guess.

.. note::

    

|

Test Objectives
============================================================================================

침투 테스트 시 사용할 취약점과 적합한 익스플로잇을 찾기하기 위해 실행중인 웹 서버의 버전과 형식을 확인합니다.

|


How to Test
============================================================================================

|

Black Box testing
-------------------------------------------------------------------------------------------

웹 서버를 식별하기 위한 가장 간단하고 기본적인 방법은 HTTP 응답 헤더에서 Server 필드를 확인하는 것입니다.
NetCat을 이용하여 다음과 같이 사용합니다.

.. code-block:: console

    $ nc 202.41.76.251 80
    HEAD / HTTP/1.0
    HTTP/1.1 200 OK
    Date: Mon, 16 Jun 2003 02:53:29 GMT
    Server: Apache/1.3.3 (Unix) (Red Hat/Linux)
    Last-Modified: Wed, 07 Oct 1998 11:18:14 GMT
    ETag: “1813-49b-361b4df6”
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

Protocol Behavior
-------------------------------------------------------------------------------------------

더 정교한 기술은 이용하는 여러 웹 서버의 다양한 특성을 이용하는 것입니다. 
아래 침투 테스터가 침투시 웹 서버 형태를 추론할 수 있는 또 다른 방법을 설명합니다.

|

HTTP header field ordering
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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

Malformed requests test
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

또 다른 유용한 침투 테스트는 악의적인 요청 또는 존재하지 않는 페이지 요청을 서버에 보내는 것입니다.
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
============================================================================================

- httprint - http://net-square.com/httprint.html
- httprecon - http://www.computec.ch/projekte/httprecon/
- Netcraft - http://www.netcraft.com
- Desenmascarame - http://desenmascara.me


Automated Testing
-------------------------------------------------------------------------------------------


Rather than rely on manual banner grabbing and analysis of the web
server headers, a tester can use automated tools to achieve the same
results. There are many tests to carry out in order to accurately fingerprint
a web server. Luckily, there are tools that automate these tests.
“httprint” is one of such tools. httprint uses a signature dictionary that
allows it to recognize the type and the version of the web server in
use.
An example of running httprint is shown below:

|

Online Testing
-------------------------------------------------------------------------------------------

Online tools can be used if the tester wishes to test more stealthily
and doesn’t wish to directly connect to the target website. An example 
of an online tool that often delivers a lot of information about target
Web Servers, is Netcraft. With this tool we can retrieve information
about operating system, web server used, Server Uptime, Netblock
Owner, history of change related to Web server and O.S.
An example is shown below:

OWASP Unmaskme Project is expected to become another online
tool to do fingerprinting of any website with an overall interpretation
of all the Web-metadata extracted. The idea behind this project
is that anyone in charge of a website could test the metadata the
site is showing to the world and assess it from a security point of
view.
While this project is still being developed, you can test a Spanish
Proof of Concept of this idea.


|

References
============================================================================================


- Saumil Shah: “An Introduction to HTTP fingerprinting” - http://www.net-square.com/httprint_paper.html
- Anant Shrivastava: “Web Application Finger Printing” - http://anantshri.info/articles/web_app_finger_printing.html

|

Remediation
============================================================================================

Protect the presentation layer web server behind a hardened reverse proxy.
Obfuscate the presentation layer web server headers.

- Apache
- IIS

|