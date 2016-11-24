==========================================================================================
OTG-INFO-008 (웹 응용 프로그램 프레임워크 핑거프린트)
==========================================================================================

|

개요
==========================================================================================

웹 프레임워크 핑거프린트는 정보 수집 과정 중 중요한 작업 중 하나입니다.

이러한 프레임워크가 침투 테스터에 의해 이미 테스트 되었다면, 프레임워크의 유형을 아는 것은 자동으로 큰 이점을 제공할 수 있습니다.

패치되지 않은 버전의 알려진 취약점 뿐 아니라 프레임워크의 특정 구성 오류 및 핑거프린팅 프로세스를 중요하게 만드는 알려진 파일 구조이기도 합니다.
여러 다른 공급 업체 및 웹 프레임 워크 버전이 널리 사용됩니다.
이에 대한 정보는 테스트 과정에서 크게 도움이되며 테스트 과정을 변경하는 데 도움이 될 수 있습니다.
이러한 정보는 특정 공통 위치에 대한 신중한 분석을 통해 얻을 수 있습니다.

대부분의 웹 프레임 워크에는 공격자가 사이트를 발견하는 데 도움이 되는 여러 마커가 있습니다.
이것은 기본적으로 모든 자동 도구가 수행하는 것으로 사전 정의 된 위치에서 마커를 찾고 알려진 서명의 데이터베이스와 비교합니다.
더 나은 정확성을 위해 일반적으로 여러 마커가 사용됩니다.

|

테스트 목적
==========================================================================================

보안 테스트 방법에 대한 더 나은 이해를 갖기 위해 사용하는 웹 프레임워크 유형을 정의
    
|


테스트 방법
==========================================================================================

|

Black Box Testing
-----------------------------------------------------------------------------------------

현재 프레임워크를 정의하기 위해 찾을 수 있는 가장 일반적인 위치가 있습니다.

- HTTP 헤더
- Cookies
- HTML source code
- 특정 파일과 폴더

|

HTTP 헤더
-----------------------------------------------------------------------------------------

웹 프레임워크를 식별하는 가장 기본적인 형태는 HTTP response 헤더에서 X-Powered-By 필드를 찾는 것입니다.
많은 툴들이 목표에 대한 핑거프린트를 하는데 사용될 수 있습니다. 가장 간단한 것 중 하나가 netcat 유틸리티입니다. 다음 HTTP Request와 Response를 고려해봅니다.

.. code-block:: console

    $ nc 127.0.0.1 80
    HEAD / HTTP/1.0

    HTTP/1.1 200 OK
    Server: nginx/1.0.14
    Date: Sat, 07 Sep 2013 08:19:15 GMT
    Content-Type: text/html;charset=ISO-8859-1
    Connection: close
    Vary: Accept-Encoding
    X-Powered-By: Mono

X-Powered-By 필드로 부터, Mono라는 웹 응용 프로그램 프레임워크인 걸 확인하였습니다.
이 방법은 간단하고 빠르지만, 100% 완벽하지는 않습니다.
설정에 의해 쉽게 X-Powered-By 헤더를 비활성화 할 수 있습니다.
또한, HTTP 헤더를 통해 난독화할 수 있는 몇가지 기술이 있습니다.
그래서, 아래 같은 예제에서 테스터는 X-Powered-By 헤더가 누락되거나 다음과 같은 응답을 얻을 수 있습니다.

.. code-block:: console

    HTTP/1.1 200 OK
    Server: nginx/1.0.14
    Date: Sat, 07 Sep 2013 08:19:15 GMT
    Content-Type: text/html;charset=ISO-8859-1
    Connection: close
    Vary: Accept-Encoding
    X-Powered-By: Blood, sweat and tears

때때로는 특정 웹 프레임워크를 가리키는 HTTP 헤더가 있습니다.
다음 예제에서, HTTP 리퀘스트로 부터 정보에 의해 PHP 버전을 포함하는 X-Powered-By 헤더를 볼 수 있습니다.
그러나, X-Generator 헤더는 실제로 사용되는 프레임워크가 Swiftlet를 가리키며, 침투 테스터는 공격 경로를 확장하는 데 도움이 됩니다.
핑거프린팅을 수행할 때, 항상 HTTP 헤더 노출에 대한 검사를 주의깊게 해야합니다.

.. code-block:: console

    HTTP/1.1 200 OK
    Server: nginx/1.4.1
    Date: Sat, 07 Sep 2013 09:22:52 GMT
    Content-Type: text/html
    Connection: keep-alive
    Vary: Accept-Encoding
    X-Powered-By: PHP/5.4.16-1~dotdeb.1
    Expires: Thu, 19 Nov 1981 08:52:00 GMT
    Cache-Control: no-store, no-cache, must-revalidate, postcheck=0, pre-check=0
    Pragma: no-cache
    X-Generator: Swiftlet

|

Cookies
-----------------------------------------------------------------------------------------

현재 웹 프레임워크를 결정하는 또 다른 유사한 방법은 프레임워크의 특정 쿠키 값입니다.

.. code-block:: console

    GET /cake HTTP /1.1
    Host: defcon-moscow.org
    User-Agent: Mozilla75.0 |Macintosh; Intel Mac OS X 10.7; rv:
    Accept: text/html, application/xhtml + xml, application/xml;
    Accept - Language: ru-ru, ru; q=0.8, en-us; q=0.5 , en; q=0 . 3
    Accept - Encoding: gzip, deflate
    DNT: 1
    Cookie: CAKEPHP=rm72kprivgmau5fmjdesbuqi71;
    Connection: Keep-alive
    Cache-Control: max-age=0

프레임워크 사용에 대한 정보를 CAKEPHP 쿠키로 자동 설정됩니다. 

.. code-block:: console

    /**
    * The name of CakePHP`s session cookie.
    *
    * Note the guidelines for Session names states: "The session
    name references
    * the session id in cookies and URLs. It should contain only alphanumeric
    * characters."
    * @link http://php.net/session_name
    */
    Configure::write('Session.cookie', 'CAKEPHP');

그러나, 이러한 변경은 X-Powered-By 헤더의 변경보다 쉽게 수행되지 않으므로, 이 방법은 더 안정적인 것으로 간주 될 수 있습니다.

|

HTML source code
-----------------------------------------------------------------------------------------

이 기법은 HTML 페이지 소스 코드에서 특정 패턴을 찾는 것에 기반합니다.

종종 테스터가 특정 웹 프레임워크를 인식하는 데 도움이되는 많은 정보를 찾을 수 있습니다.
일반적인 마커 중 하나는 프레임워크 공개로 직접 이어지는 HTML 주석입니다.

종종 특정 프레임워크 별 경로, 즉 프레임워크 별 css 또는 js 폴더에 대한 링크를 찾을 수 있습니다.
마지막으로 특정 스크립트 변수는 특정 프레임워크를 가리킬 수도 있습니다.
아래 스크린 샷에서 언급된 마커를 통해 사용된 프레임워크와 버전을 쉽게 배울 수 있습니다.

주석, 특정 경로 및 스크립트 변수는 모두 공격자가 ZK 프레임 워크의 인스턴스를 신속하게 결정하는 데 도움이 될 수 있습니다.

이러한 정보는 <head> </head> 태그, <meta> 태그 또는 페이지 끝 부분에 더 자주 배치됩니다.
그럼에도 불구하고 다른 유용한 주석과 숨겨진 필드를 검사하는 것과 같은 다른 목적에 유용할 수 있기 때문에 전체 문서를 검사하는 것이 좋습니다.
때때로 웹 개발자는 사용된 프레임워크에 대한 정보를 숨기지 않습니다.
다음과 같이 페이지 하단에서 우연히 마주치는 것이 가능할 수 있습니다.

|

일반적인 프레임워크
-----------------------------------------------------------------------------------------

.. csv-table:: Cookies
    :header: "프레임워크","Cookie 이름"

    "Zope", "zope3"
    "CakePHP","cakephp"
    "Kohana", "kohanasession"
    "Laravel","laravel_session"

.. csv-table:: HTML 소스 코드
    :header: "일반적인 마커"

    "%framework_name%"
    "powered by"
    "built upon"
    "running"

.. csv-table:: 특수 마커
    :header: "프레임워크","키워드"

    "Adobe ColdFusion", "<!-- START headerTags.cfm"
    "Microsoft ASP.NET", "__VIEWSTATE"
    "ZK", "<!-- ZK"
    "Business Catalyst","<!-- BC_OBNW -->"
    "Indexhibit","ndxz-studio"


|

특정 파일과 폴더
-----------------------------------------------------------------------------------------

특정 파일 및 폴더는 특정 프레임워크마다 다릅니다.
어떤 인프라가 제공되고 어떤 파일이 서버에 남아있을지 더 잘 이해할 수 있도록 침투 테스트 중에 해당 프레임워크를 설치해 보는 것이 좋습니다.
(http://code.google.com/p/fuzzdb/).

|

도구
==========================================================================================

WhatWeb
-----------------------------------------------------------------------------------------

http://www.morningstarsecurity.com/research/whatweb

현재 시장에서 가장 우수한 지문 인식 도구 중 하나입니다.
기본 Kali Linux 빌드에 포함됩니다. 언어: Ruby 지문 일치는 다음으로 이루어집니다.

- Text strings (case sensitive)
- Regular expressions
- Google Hack Database queries (limited set of keywords)
- MD5 hashes
- URL recognition
- HTML tag patterns
- Custom ruby code for passive and aggressive operations


|

BlindElephant
-----------------------------------------------------------------------------------------

https://community.qualys.com/community/blindelephant

이 훌륭한 도구는 정적 파일 체크섬 기반의 원칙에 따라 작동합니다.
버전 차이로 인해 매우 높은 품질의 지문을 제공합니다. 언어: Python

성공적인 지문 샘플 출력

.. code-block:: console

    pentester$ python BlindElephant.py http://my_target drupal
    Loaded /Library/Python/2.7/site-packages/blindelephant/
    dbs/drupal.pkl with 145 versions, 478 differentiating paths,
    and 434 version groups.
    Starting BlindElephant fingerprint for version of drupal at
    http://my_target
    
    Hit http://my_target/CHANGELOG.txt
    File produced no match. Error: Retrieved file doesn`t match
    known fingerprint. 527b085a3717bd691d47713dff74acf4
    
    Hit http://my_target/INSTALL.txt
    File produced no match. Error: Retrieved file doesn`t match
    known fingerprint. 14dfc133e4101be6f0ef5c64566da4a4
    
    Hit http://my_target/misc/drupal.js
    Possible versions based on result: 7.12, 7.13, 7.14

    Hit http://my_target/MAINTAINERS.txt
    File produced no match. Error: Retrieved file doesn`t match
    known fingerprint. 36b740941a19912f3fdbfcca7caa08ca 

    Hit http://my_target/themes/garland/style.css
    Possible versions based on result: 7.2, 7.3, 7.4, 7.5, 7.6, 7.7,
    7.8, 7.9, 7.10, 7.11, 7.12, 7.13, 7.14
    ...

    Fingerprinting resulted in:
    7.14
    
    Best Guess: 7.14

|

Wappalyzer
-----------------------------------------------------------------------------------------

http://wappalyzer.com

Wapplyzer는 Firefox Chrome 플러그인입니다.
정규식 일치에서만 작동하며 브라우저에 로드할 페이지 이외의 다른 것을 필요로 하지 않습니다.
그것은 브라우저 수준에서 완전히 작동하고 아이콘의 형태로 결과를 제공합니다.
때로는 오탐(false positive)이 있지만, 때때로 페이지를 탐색한 후 대상 웹 사이트를 구성하는 데 사용된 기술을 이해하는 데 매우 편리합니다.

플러그인의 샘플 출력은 아래 스크린 샷에 나와 있습니다.

|

참고 문헌
==========================================================================================

Whitepapers
-----------------------------------------------------------------------------------------

- Saumil Shah: "An Introduction to HTTP fingerprinting" - http://www.net-square.com/httprint_paper.html
- Anant Shrivastava : "Web Application Finger Printing" - http://anantshri.info/articles/web_app_finger_printing.html


|

권고 사항
==========================================================================================

일반적인 조언은 위에 설명된 여러 도구를 사용하여 로그를 검사하여 침입자가 웹 프레임워크를 공개하는 데 정확히 도움이 되는지 이해하는 것입니다.
프레임워크 트랙을 숨기도록 변경한 후에 여러 번 스캔을 수행하면 더 나은 보안 수준을 달성하고 자동 스캔으로 프레임워크를 감지할 수 없는 지 확인할 수 있습니다.
다음은 프레임워크 마커 위치 및 몇 가지 추가적인 흥미로운 접근법에 따른 몇 가지 구체적인 권장 사항입니다.

**HTTP headers**

구성을 확인하고 기술이 사용하는 정보를 공개하는 모든 HTTP 헤더를 비활성화하거나 난독화 하십시오. 
다음은 Netscaler를 사용하는 HTTP 헤더 난독화에 대한 흥미로운 기사입니다.

http://grahamhosking.blogspot.ru/2013/07/obfuscating-http-header-using-netscaler.html

|

**Cookies**

해당 구성 파일을 변경하여 쿠키 이름을 변경하는 것이 좋습니다.

|

**HTML source code**

수동으로 HTML 코드의 내용을 확인하고 명시적으로 프레임워크를 가리키는 모든 것을 제거하십시오.

일반 지침 :

- 프레임 워크를 공개하는 시각적 마커가 없는지 확인하십시오.
- 불필요한 주석(저작권, 버그 정보, 특정 프레임 워크 주석)을 제거하십시오.
- META 및 생성기 태그 제거
- 회사의 css 또는 js 파일을 사용하고 프레임 워크 관련 폴더에 파일을 저장하지 않습니다.
- 페이지에서 기본 스크립트를 사용하지 않거나 사용해야 할 경우 난독 화합니다.

|

**특정 파일과 폴더**

일반 지침:

- 서버에서 불필요하거나 사용되지 않는 파일을 제거하십시오. 이것은 텍스트 파일이 버전 및 설치에 대한 정보를 공개함을 의미합니다.
- 외부 파일에 액세스 할 때 404 응답을 얻기 위해 다른 파일에 대한 액세스를 제한하십시오. 예를 들어, htaccess 파일을 수정하고, RewriteCond 또는 RewriteRule을 추가하여 이를 수행 할 수 있습니다. 두 가지 일반적인 WordPress 폴더에 대한 이러한 제한의 예가 아래에 나와 있습니다.

.. code-block:: console

    RewriteCond %{REQUEST_URI} /wp-login\.php$ [OR]
    RewriteCond %{REQUEST_URI} /wp-admin/$
    RewriteRule $ /http://your_website [R=404,L]


그러나 이것 만이 액세스를 제한하는 유일한 방법은 아닙니다. 이 프로세스를 자동화하기 위해 특정 프레임워크 관련 플러그인이 존재합니다.
WordPress의 한 가지 예가 StealthLogin입니다. (http://wordpress.org/plugins/stealth-login-page).

|

**추가 접근법**

일반 지침:

1. 체크섬 관리

이 접근법의 목적은 체크섬 기반 스캐너를 이기고, 해시로 파일을 공개하지 못하게하는 것입니다. 
일반적으로 체크섬 관리에는 두 가지 방법이 있습니다.

- 파일을 저장할 위치를 변경합니다 (예 : 다른 폴더로 이동하거나 기존 폴더 이름 바꾸기).
- 내용을 수정하십시오 : 완전히 다른 해시 합계로 약간의 수정 결과가 있어도 파일 끝에 단일 바이트를 추가하는 것이 큰 문제는 아닙니다.

2. 제어가능한 혼돈

재미있는 방법은 스캐너를 속이고 공격자를 혼란시키기 위해 다른 프레임워크에서 가짜 파일과 폴더를 추가하는 것입니다. 그러나 기존 파일과 폴더를 덮어 쓰지 말고 현재 프레임워크를 깨뜨리지 않도록 주의하십시오!

|