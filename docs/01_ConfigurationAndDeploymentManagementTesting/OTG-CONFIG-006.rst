==========================================================================================
OTG-CONFIG-006 (HTTP 메소드 테스트)
==========================================================================================

|

개요
==========================================================================================

HTTP는 웹 서버에서 작업을 수행하는 데 사용할 수있는 여러 가지 방법을 제공합니다. 
이러한 많은 방법은 개발자가 HTTP 응용 프로그램을 배포하고 테스트하는 데 도움이 되도록 설계되었습니다. 
이러한 HTTP 메소드는 웹 서버가 잘못 구성된 경우 악의적인 목적으로 사용될 수 있습니다. 
또한 서버의 HTTP TRACE 메서드를 사용하는 교차 사이트 스크립팅의 한 형태인 XST(Cross Site Tracing)가 검사됩니다.

GET 및 POST는 웹 서버에서 제공하는 정보에 액세스하는 데 가장 많이 사용되는 방법이지만 HTTP는 몇 가지 다른 방법을 허용합니다. 
RFC 2616(현재 표준 인 HTTP 버전 1.1을 설명 함)에서는 다음과 같은 8가지 방법을 정의합니다.

- HEAD
- GET
- POST
- PUT
- DELETE
- TRACE
- OPTIONS
- CONNECT

공격자가 웹 서버에 저장된 파일을 수정하고 일부 시나리오에서는 합법적 인 사용자의 자격 증명을 훔칠 수 있으므로 이러한 방법 중 일부는 웹 응용 프로그램의 보안 위험을 초래할 수 있습니다. 
보다 구체적으로, 사용 불가능하게 해야 하는 메소드는 다음과 같습니다.

- PUT: 이 메소드를 사용하면 클라이언트가 웹 서버에 새 파일을 업로드 할 수 있습니다. 공격자는 악의적인 파일(예: cmd.exe를 호출하여 명령을 실행하는 ASP 파일)을 업로드하거나 피해자의 서버를 단순히 파일 저장소로 사용하여 악성 파일을 악용할 수 있습니다.
- DELETE: 이 메소드는 클라이언트가 웹 서버의 파일을 삭제할 수 있게합니다. 공격자는 웹 사이트를 손상 시키거나 DoS 공격을 일으키는 매우 간단하고 직접적인 방법으로 악용 할 수 있습니다.
- CONNECT: 이 방법을 사용하면 클라이언트가 웹 서버를 프록시로 사용할 수 있습니다.
- TRACE: 이 메소드는 서버로 전송된 문자열이 무엇이든 간에 클라이언트에 간단하게 에코하여 디버깅 목적으로 주로 사용됩니다. 원래 무해하다고 가정된 이 방법은 Jeremiah Grossman이 발견한 교차 사이트 추적이라는 공격을 수행하는 데 사용할 수 있습니다.

응용 프로그램에 REST 웹 서비스(PUT 또는 DELETE가 필요할 수 있음)와 같은 이러한 메서드 중 하나 이상이 필요한 경우 해당 사용이 신뢰할 수있는 사용자와 안전한 조건으로 올바르게 제한되는지 확인하는 것이 중요합니다.


|

임의의 HTTP 메소드
==========================================================================================

Arshan Dabirsiaghi (링크 참조)는 많은 웹 응용 프로그램 프레임 워크에서 환경 수준의 액세스 제어 검사를 우회하기 위해 잘 선택되거나 임의의 HTTP 메소드를 사용할 수 있음을 발견했습니다.

많은 프레임 워크와 언어는 "HEAD"를 "GET"요청으로 취급합니다. 
응답에는 본문이 없어도 됩니다. 
GET 요청에 보안 제약 조건을 설정하여 "authenticatedUsers"만이 특정 서블릿이나 리소스에 대한 GET 요청에 액세스 할 수있는 경우 "HEAD"버전에서는 무시됩니다. 
이로 인해 권한이 부여되지 않은 GET 요청을 맹목적으로 제출할 수 있었습니다.
일부 프레임 워크에서는 "JEFF"또는 "CATS"와 같은 임의의 HTTP 메소드를 제한없이 사용할 수 있었습니다. 
이것들은 "GET" 메소드가 발행 된 것처럼 처리되었고 많은 언어와 프레임 워크에 대한 메소드 롤 기반 액세스 제어 검사의 대상이 아니며 권한이 부여되지 않은 GET 요청의 맹목적 제출을 다시 허용합니다.

대부분의 경우 "GET"또는 "POST"메소드를 명시적으로 검사한 코드는 안전합니다.

|

테스트 방법
==========================================================================================

지원하는 메소드 확인
------------------------------------------------------------------------------------------

이 테스트를 수행하려면 테스터는 검사중인 웹 서버가 지원하는 HTTP 메소드를 파악할 수 있는 방법이 필요합니다. 
OPTIONS HTTP 메소드는 테스터에게 가장 직접적이고 효과적인 방법을 제공합니다. 
RFC 2616에서는 "OPTIONS 메서드는 Request-URI로 식별되는 요청/응답 체인에서 사용할 수있는 통신 옵션에 대한 정보 요청"을 나타냅니다.

테스트 방법은 매우 간단하며 netcat(또는 telnet)만 실행하면 됩니다.

.. code-block:: console

    $ nc www.victim.com 80
    OPTIONS / HTTP/1.1
    Host: www.victim.com

    HTTP/1.1 200 OK
    Server: Microsoft-IIS/5.0
    Date: Tue, 31 Oct 2006 08:00:29 GMT
    Connection: close
    Allow: GET, HEAD, POST, TRACE, OPTIONS
    Content-Length: 0


예제에서 볼 수 있듯이 OPTIONS은 웹 서버가 지원하는 메소드 목록을 제공하며, 이 경우 TRACE 메소드가 사용 가능하다는 것을 알 수 있습니다. 
이 방법으로 인해 발생할 수 있는 위험은 다음 섹션에 설명되어 있습니다.

nmap과 http-methods NSE 스크립트를 사용하여 동일한 테스트를 실행할 수도 있습니다:

.. code-block:: console

    C:\Tools\nmap-6.40>nmap -p 443 --script http-methods localhost

    Starting Nmap 6.40 ( http://nmap.org ) at 2015-11-04 11:52 Romance Standard Time

    Nmap scan report for localhost (127.0.0.1)
    Host is up (0.0094s latency).
    PORT    STATE SERVICE
    443/tcp open  https
    | http-methods: OPTIONS TRACE GET HEAD POST
    | Potentially risky methods: TRACE
    |_See http://nmap.org/nsedoc/scripts/http-methods.html

    Nmap done: 1 IP address (1 host up) scanned in 20.48 seconds


|

XST 가능성 테스트
------------------------------------------------------------------------------------------

Note: 이 공격의 논리와 목표를 이해하려면 Cross Site Scripting 공격에 익숙해야합니다.

명백하게 무해한 TRACE 방법은 합법적 인 사용자의 자격 증명을 도용하기 위해 일부 시나리오에서 성공적으로 활용할 수 있습니다. 
이 공격 기법은 2003 년 Jeremiah Grossman이 Microsoft에서 Internet Explorer 6 SP1에 도입한 HTTPOnly 태그를 우회하여 쿠키가 JavaScript에 의해 액세스되지 않도록 보호하기 위해 발견되었습니다. 
실제로 Cross Site Scripting에서 가장 자주 발생하는 공격 패턴 중 하나는 document.cookie 개체에 액세스하여 공격자가 제어하는 ​​웹 서버로 보내어 피해자 세션을 가로 챌 수 있도록하는 것입니다. 
httpOnly로 쿠키에 태그를 지정하면 JavaScript가 액세스하지 못하므로 제 3자에게 전송되지 않습니다. 
그러나 이 시나리오에서는 TRACE 메서드를 사용하여이 보호를 무시하고 쿠키에 액세스 할 수 있습니다.

앞에서 언급했듯이 TRACE는 웹 서버에 전송 된 문자열을 반환합니다. 존재 여부를 확인하거나 위에 나온 OPTIONS 요청의 결과를 다시 확인하려면 테스터가 다음 예제와 같이 진행될 수 있습니다.


.. code-block:: console

    $ nc www.victim.com 80
    TRACE / HTTP/1.1
    Host: www.victim.com

    HTTP/1.1 200 OK
    Server: Microsoft-IIS/5.0
    Date: Tue, 31 Oct 2006 08:01:48 GMT
    Connection: close
    Content-Type: message/http
    Content-Length: 39

    TRACE / HTTP/1.1
    Host: www.victim.com

응답 본문은 정확히 원래 요청의 사본으로 대상에서이 메서드를 허용합니다. 
자, 위험은 어디에 숨어 있습니까? 
테스터가 웹 서버에 TRACE 요청을 보내도록 브라우저에 지시하고 이 브라우저에 해당 도메인에 대한 쿠키가 있는 경우 쿠키는 요청 헤더에 자동으로 포함되므로 결과 응답에 다시 표시됩니다. 
이 시점에서 쿠키 문자열은 JavaScript로 액세스 할 수 있으며 쿠키가 httpOnly로 태그 된 경우에도 최종적으로 제 3 자에게 보낼 수 있습니다.


Internet Explorer의 XMLHTTP ActiveX 컨트롤과 Mozilla 및 Netscape의 XMLDOM과 같이 브라우저가 TRACE 요청을 발행하는 여러 가지 방법이 있습니다. 그러나 보안상의 이유로 브라우저는 적대적인 스크립트가 상주하는 도메인에만 연결을 시작할 수 있습니다. 
이는 공격자가 공격을 수행하기 위해 TRACE 방법과 다른 취약점을 결합해야하므로 완화 요소입니다.


공격자는 Cross Site Tracing 공격을 성공적으로 시작하는 두 가지 방법이 있습니다.

- 또 다른 서버 측 취약점을 이용: 공격자는 정상적인 크로스 사이트 스크립팅 공격처럼 취약한 애플리케이션에 TRACE 요청을 포함하는 악의적인 JavaScript 스니펫을 주입합니다.
- 클라이언트 측 취약점 활용: 공격자는 적대적인 JavaScript 코드 단편을 포함하는 악의적인 웹 사이트를 만들고 해당 브라우저의 일부 도메인 간 취약성을 악용하여 JavaScript 코드가 해당 코드를 지원하는 사이트에 성공적으로 연결되도록합니다. TRACE 메서드를 사용하여 공격자가 훔치려고 시도하는 쿠키를 유래했습니다.

코드 샘플과 함께보다 자세한 정보는 Jeremiah Grossman이 작성한 원본 백서에서 찾을 수 있습니다.


|

임의의 HTTP 메소드 테스트
------------------------------------------------------------------------------------------

방문 제약 조건이있는 방문 페이지를 찾아서 일반적으로 로그인 페이지로 302 리디렉션하거나 강제로 로그인하도록 하십시오. 
이 예제의 테스트 URL은 많은 웹 애플리케이션과 마찬가지로 작동합니다. 
그러나 테스터가 로그인 페이지가 아닌 "200"응답을 얻으면 인증 및 권한 부여를 무시할 수 있습니다.

.. code-block:: console

    $ nc www.example.com 80
    JEFF / HTTP/1.1
    Host: www.example.com
    HTTP/1.1 200 OK
    Date: Mon, 18 Aug 2008 22:38:40 GMT
    Server: Apache
    Set-Cookie: PHPSESSID=K53QW...

프레임 워크 또는 방화벽 또는 응용 프로그램이 "JEFF" 메소드를 지원하지 않으면 오류 페이지(또는 바람직하게는 405 Not Allowed 또는 501 Not implemented 오류 페이지)를 발행해야합니다. 요청을 처리하면 이 문제에 취약합니다.

테스터가 시스템이 이 문제에 취약하다고 생각하면 CSRF와 유사한 공격을 하여 문제를보다 완벽하게 활용해야합니다.

- FOOBAR /admin/createUser.php?member=myAdmin
- JEFF /admin/changePw.php?member=myAdmin&passwd=foo123&confirm=foo123
- CATS /admin/groupEdit.php?group=Admins&member=myAdmin&action=add

테스트중인 애플리케이션과 테스트 요구 사항에 맞게 수정 된 위의 세 가지 명령을 사용하여 새로운 사용자가 생성되고 비밀번호가 지정되고 관리자가되는 행운이 있습니다.

|

HEAD 접속 제어 우회 테스트
------------------------------------------------------------------------------------------

방문 제약 조건이 있는 방문 페이지를 찾아서 일반적으로 로그인 페이지로 302 리디렉션하거나 강제로 로그인하도록 하십시오. 
이 예제의 테스트 URL은 많은 웹 애플리케이션과 마찬가지로 작동합니다. 
그러나 테스터가 로그인 페이지가 아닌 "200" 응답을 얻으면 인증 및 권한 부여를 무시할 수 있습니다.

.. code-block:: console

    $ nc www.example.com 80
    JEFF / HTTP/1.1
    Host: www.example.com

    HTTP/1.1 200 OK
    Date: Mon, 18 Aug 2008 22:38:40 GMT
    Server: Apache
    Set-Cookie: PHPSESSID=K53QW...
    Expires: Thu, 19 Nov 1981 08:52:00 GMT
    Cache-Control: no-store, no-cache, must-revalidate, postcheck=0,
    pre-check=0

    Pragma: no-cache
    Set-Cookie: adminOnlyCookie1=...; expires=Tue, 18-Aug-
    2009 22:44:31 GMT; domain=www.example.com
    Set-Cookie: adminOnlyCookie2=...; expires=Mon, 18-Aug-
    2008 22:54:31 GMT; domain=www.example.com
    Set-Cookie: adminOnlyCookie3=...; expires=Sun, 19-Aug-
    2007 22:44:30 GMT; domain=www.example.com
    Content-Language: EN
    Connection: close
    Content-Type: text/html; charset=ISO-8859-1

테스터가 "405 Method not allowed"또는 "501 Method Unimplemented"를 얻으면 대상(application/framework/language/system/firewall)이 올바르게 작동합니다. 
"200" 응답 코드가 반환되고 응답에 본문이 없으면 응용 프로그램이 인증이나 권한 부여없이 요청을 처리했으며 추가 테스트가 필요합니다.

테스터가 시스템이 이 문제에 취약하다고 생각하면 CSRF와 유사한 공격을 제기하여 문제를보다 완벽하게 활용해야합니다.

- HEAD /admin/createUser.php?member=myAdmin
- HEAD /admin/changePw.php?member=myAdmin&passwd=foo123&confirm=foo123
- HEAD /admin/groupEdit.php?group=Admins&member=myAdmin&action=add

테스트중인 애플리케이션과 테스트 요구 사항에 맞게 수정 된 위의 세 가지 명령을 사용하여 모든 사용자에게 맹인 요청 제출을 사용하여 새로운 사용자를 만들고 비밀번호를 할당하고 관리자를 만든 것입니다.

|

도구
==========================================================================================

- NetCat: http://nc110.sourceforge.net
- cURL: http://curl.haxx.se/

|

참고 문헌
==========================================================================================

Whitepapers
------------------------------------------------------------------------------------------

- RFC 2616: "Hypertext Transfer Protocol -- HTTP/1.1"
- RFC 2109 and RFC 2965: "HTTP State Management Mechanism"
- Jeremiah Grossman: "Cross Site Tracing (XST)": http://www.cgisecurity.com/whitehat-mirror/WH-WhitePaper_XST_ebook.pdf
- Amit Klein: "XS(T) attack variants which can, in some cases, eliminate the need for TRACE": http://www.securityfocus.com/archive/107/308433
• Arshan Dabirsiaghi: "Bypassing VBAAC with HTTP Verb Tampering" - http://static.swpag.info/download/Bypassing_VBAAC_with_HTTP_Verb_Tampering.pdf

|
