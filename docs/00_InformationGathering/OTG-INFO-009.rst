==========================================================================================
OTG-INFO-009 (웹 응용 프로그램 핑거프린트)
==========================================================================================

|

개요
==========================================================================================

태양 아래에서는 새로운 것이 없으며 개발을 생각할 수 있는 거의 모든 웹 응용 프로그램이 이미 개발되었습니다.

전 세계적으로 활발히 개발되고 배포되는 무료 및 오픈 소스 소프트웨어 프로젝트가 많기 때문에 응용 프로그램 보안 테스트가 이러한 잘 알려진 응용 프로그램에 전적으로 또는 부분적으로 의존하는 대상 사이트에 직면하게 될 가능성이 매우 큽니다 (예 : Wordpress, phpBB, Mediawiki 등).

테스트 중인 웹 응용 프로그램 구성 요소를 잘 알고 있으면 테스트 프로세스에 도움이 되며, 테스트 중에 필요한 노력을 크게 줄일 수 있습니다.
이러한 잘 알려진 웹 응용 프로그램에는 응용 프로그램을 식별하기 위해 목록화 할 수 있는 HTML 헤더, 쿠키 및 디렉터리 구조가 있습니다.

|

테스트 목적
==========================================================================================

웹 응용 프로그램, 알려진 취약점을 결정하는 버전, 테스트 시 사용할 적절한 익스플로잇 확인

|


테스트 방법
==========================================================================================

Cookies
-----------------------------------------------------------------------------------------

웹 응용 프로그램을 식별하기 위한 신뢰할 수 있는 방법은 응용 프로그램에 지정된 쿠키이다.

HTTP-request 확인:

.. code-block:: console

    GET / HTTP/1.1
    User-Agent: Mozilla/5.0 (Windows NT 6.2; WOW64; rv:31.0)
    Gecko/20100101 Firefox/31.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8

    Accept-Language: en-US,en;q=0.5
    ‘’’Cookie: wp-settings-time-1=1406093286; wp-settingstime-2=1405988284’’’
    DNT: 1
    Connection: keep-alive
    Host: blog.owasp.org


CAKEPHP는 쿠키가 자동으로 설정되어, 사용하고 있는 프레임워크 정보를 제공합니다.
일반적인 쿠키명 목록은 아래 Common Application Identifiers 섹션에 나와있습니다.
그러나, 쿠키 이름은 바꾸는게 가능합니다.

|

HTML source code
-----------------------------------------------------------------------------------------

이 기술은 HTML 페이지 소스 코드에서 정확한 패턴을 찾는게 기본입니다.

테스터가 특정 웹 응용 프로그램을 인식하는 데 도움이 되는 많은 정보를 자주 볼 수 있습니다.
일반적인 마커 중 하나는 응용 프로그램 공개로 직접 이어지는 HTML 주석입니다.
종종 특정 응용 프로그램 별 경로, 즉 응용 프로그램 별 css 또는 js 폴더에 대한 링크를 찾을 수 있습니다.

마지막으로 특정 스크립트 변수는 특정 응용 프로그램을 가리킬 수도 있습니다.
아래 메타 태그에서 웹 사이트와 웹 사이트 버전에서 사용하는 응용 프로그램을 쉽게 배울 수 있습니다.
주석, 특정 경로 및 스크립트 변수는 모두 공격자가 응용 프로그램의 인스턴스를 신속하게 결정하는 데 도움이 될 수 있습니다.

.. code-block:: html

    <meta name="generator" content="WordPress 3.9.2" />

이러한 정보는 <head> </head> 태그, <meta> 태그 또는 페이지 끝 부분에 더 자주 배치됩니다.
그럼에도 불구하고 다른 유용한 주석과 숨겨진 필드를 검사하는 것과 같은 다른 목적에 유용 할 수 있기 때문에 전체 문서를 검사하는 것이 좋습니다.

|

특정 파일과 폴더
-----------------------------------------------------------------------------------------

HTML 소스에서 수집한 정보 외에도 공격자가 높은 정확도로 응용 프로그램을 결정하는 데 큰 도움이되는 또 다른 방법이 있습니다.

모든 응용 프로그램에는 서버에 고유한 특정 파일 및 폴더 구조가 있습니다.

HTML 페이지 소스에서 특정 경로를 볼 수 있지만 때때로 명시적으로 거기에 표시되지 않고 여전히 서버에 상주하지 않는 경우가 있습니다.

그들을 밝히기 위해 dirbusting으로 알려진 기술이 사용됩니다.

Dirbusting은 예상 할 수있는 폴더 및 파일 이름을 가진 대상을 강요하고 HTTP 응답을 모니터링하여 서버 내용을 표시합니다.

이 정보는 기본 파일을 찾고 이를 공격하고 웹 응용 프로그램을 핑거프린티하는 데 사용할 수 있습니다.

Dirbusting은 여러 가지 방법으로 수행 할 수 있습니다. 아래 예제는 Burp Suite의 정의된 목록 및 침입자 기능을 사용하여 WordPress 기반 대상에 대한 성공적인 dirbusting 공격을 보여줍니다.

일부 워드 프레스 관련 폴더 (예 : /wp-includes/, /wp-admin/ 및 /wp-content/)에서 HTTP 응답은 403(금지됨), 302 (찾음, wp-login.php) 및 200 (OK).
이것은 목표가 WordPress로 구동된다는 것을 나타내는 좋은 지표입니다.

같은 방법으로 다른 응용 프로그램 플러그인 폴더와 해당 버전을 더빙할 수 있습니다.
아래의 스크린 샷에서 Drupal 플러그인의 일반적인 CHANGELOG 파일을 볼 수 있습니다.
이 파일은 사용중인 응용 프로그램에 대한 정보를 제공하고 취약한 플러그인 버전을 공개합니다.

팁: dirbusting을 시작하기 전에 먼저 robots.txt 파일을 확인하는 것이 좋습니다.

때로는 응용 프로그램 관련 폴더 및 기타 중요한 정보가 있을 수 있습니다.
robots.txt 파일의 예가 아래 스크린 샷에 나와 있습니다.

특정 파일 및 폴더는 특정 응용 프로그램마다 다릅니다.
어떤 인프라가 제공되고 어떤 파일이 서버에 남아 있을 지 더 잘 이해할 수 있도록 침투 테스트 중에 해당 응용 프로그램을 설치하는 것이 좋습니다.
(http://code.google.com/p/fuzzdb/).

.. csv-table:: Common Application Identifiers
    :widths: 15, 10

    'phpBB','phpbb3_'
    'Wordpress','wp-settings'
    '1C-Bitrix','BITRIX_'
    'AMPcms','AMP'
    'Django CMS','django'
    'DotNetNuke','DotNetNukeAnonymous'
    'e107','e107'
    'EPiServer','EPiTrace, EPiServer'
    'Graffiti CMS','graffitibot'
    'Hotaru CMS','hotaru_mobile'
    'ImpressCMS','ICMSession'
    'Indico','MAKACSESSION'
    'InstantCMS','InstantCMS[logdate]'
    'Kentico CMS','CMSPreferredCulture'
    'MODx','SN4[12symb]'
    'TYPO3','fe_typo_user'
    'Dynamicweb','Dynamicweb'
    'LEPTON','lep[some_numeric_value]+sessionid'
    'Wix','Domain=.wix.com'
    'VIVVO','VivvoSessionId '


.. csv-table:: HTML source code 
    :widths: 15, 10

    'Wordpress','<meta name="generator" content="WordPress 3.9.2"/>'
    'phpBB','<body id="phpbb"'
    'Mediawiki','<meta name="generator" content="MediaWiki 1.21.9" />'
    'Joomla','<meta name="generator" content="Joomla! - Open Source Content Management" />'
    'Drupal','<meta name="Generator" content="Drupal 7 (http://drupal.org)" />'
    'DotNetNuke','DNN Platform - http://www.dnnsoftware.com'


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
    File produced no match. Error: Retrieved file doesn’t match
    known fingerprint. 527b085a3717bd691d47713dff74acf4

    Hit http://my_target/INSTALL.txt
    File produced no match. Error: Retrieved file doesn’t match
    known fingerprint. 14dfc133e4101be6f0ef5c64566da4a4

    Hit http://my_target/misc/drupal.js
    Possible versions based on result: 7.12, 7.13, 7.14

    Hit http://my_target/MAINTAINERS.txt
    File produced no match. Error: Retrieved file doesn’t match
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
------------------------------------------------------------------------------------------

- Saumil Shah: "An Introduction to HTTP fingerprinting" - http://www.net-square.com/httprint_paper.html
- Anant Shrivastava : "Web Application Finger Printing" - http://anantshri.info/articles/web_app_finger_printing.html

|

권고 사항
==========================================================================================

일반적인 조언은 위에 설명된 여러 도구를 사용하여 로그를 검사하여 침입자가 웹 프레임워크를 공개하는 데 정확히 도움이 되는지 이해하는 것입니다.
프레임워크 트랙을 숨기도록 변경한 후에 여러 번 스캔을 수행하면 더 나은 보안 수준을 달성하고 자동 스캔으로 프레임워크를 감지할 수 없는 지 확인할 수 있습니다.
다음은 프레임워크 마커 위치 및 몇 가지 추가적인 흥미로운 접근법에 따른 몇 가지 구체적인 권장 사항입니다.

|


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