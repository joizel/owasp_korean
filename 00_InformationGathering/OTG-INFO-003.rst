==========================================================================================
OTG-INFO-003
==========================================================================================

|

정보 노출을 확인하기 위해 웹 서버 메타 파일 확인

|

개요
==========================================================================================

이번 섹션에서는 웹 어플리케이션의 디렉토리 또는 폴더 경로 정보 노출에 대한 robots.txt 파일을 테스트하는 방법에 대해 설명합니다.
    
또한, Spiders, Robots, Crawler를 우회하는 디렉토리 리스트는 어플리키에션을 통한 실행 경로 맵에 의존하여 생성될 수 있습니다.(OTG-INFO-007)

|

테스트 목적
==========================================================================================

1. 웹 애플리케이션의 디렉토리의 정보 노출
2. Spiders, Robots, Crawler를 방지하기 위한 디렉토리 리스트 생성

|

테스트 방법
==========================================================================================

robots.txt
-------------------------------------------------------------------------------------------

Spiders, Robots, Crawler는 웹 페이지를 검색하고 더 많은 웹 컨텐츠를 검색하기 위해 반복적으로 하이퍼링크를 선회합니다.
허용 동작은 웹 루트 디렉토리에 robots.txt 파일의 Robots Exclusion Protocol에 의해 지정됩니다.
아래 2013년 8월 11에 http://www.google.com/robots.txt의 robots.txt 파일의 시작 부분
에 대한 샘플을 확인할 수 있습니다.

.. code-block:: console

    User-agent: *
    Disallow: /search
    Disallow: /sdch
    Disallow: /groups
    Disallow: /images
    Disallow: /catalogs
    ...

User-Agent 지시어는 특정 web spider/robot/crawler를 의미합니다.
예를 들어, User-Agent: Googlebot은 Goolg spider를 의미하고,
User-Agent: bingbot은 Microsoft/Yahoo crawler를 의미합니다.
User-Agent: *은 web spider/robot/crawler 전체를 의미합니다.

.. code-block:: console

    User-agent: *

Disallow 지시어는 web spider/robot/crawler에 의해 금지할 리소스를 지정합니다.
예를 들어, 아래와 같은 디렉토리는 금지 설정되어 있습니다.

.. code-block:: console

    ...
    Disallow: /search
    Disallow: /sdch
    Disallow: /groups
    Disallow: /images
    Disallow: /catalogs
    ...


Web spiders/robots/crawlers는 의도적으로 소셜 네트워크로 부터 여전히 유효한지 확인하는 robots.txt 파일에 지정된 Disallow 지시어를 무시할 수 있습니다.
따라서, robots.txt가 웹 컨텐츠를 써드 파티에 의해 접근, 저장, 게시되는 방법에 제한을 적용하는 메커니즘으로 간주되서는 안됩니다.

|

웹 루트 robots.txt - "wget", "curl"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

robots.txt 파일은 웹 서버의 웹 루트 디렉토리에서 검색됩니다.
예를 들어, www.google.com으로 부터 robots.txt를 검색하기 위해 
"wget" 또는 "curl" 을 사용합니다.

.. code-block:: console

    cmlh$ wget http://www.google.com/robots.txt
    --2013-08-11 14:40:36-- http://www.google.com/robots.txt
    Resolving www.google.com... 74.125.237.17, 74.125.237.18,
    74.125.237.19, ...
    Connecting to www.google.com|74.125.237.17|:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: unspecified [text/plain]
    Saving to: ‘robots.txt.1’

     [ <=> ] 7,074 --.-K/s in 0s

    2013-08-11 14:40:37 (59.7 MB/s) - ‘robots.txt’ saved [7074]

    cmlh$ head -n5 robots.txt
    User-agent: *
    Disallow: /search
    Disallow: /sdch
    Disallow: /groups
    Disallow: /images

.. code-block:: console

    cmlh$ curl -O http://www.google.com/robots.txt
     % Total % Received % Xferd Average Speed Time Time
    Time Current
     Dload Upload Total Spent Left Speed
    101 7074 0 7074 0 0 9410 0 --:--:-- --:--:-- --:--:--
    27312

    cmlh$ head -n5 robots.txt
    User-agent: *
    Disallow: /search
    Disallow: /sdch
    Disallow: /groups
    Disallow: /images

|

웹 루트 robots.txt - rockspider
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

"rockspider"는 파일의 Spiders/Robots/Crawlers와 웹 사이트의 디렉토리/폴더로 초기 범위를 지정하여 생성합니다.
예를 들어, "rockspider"를 사용하여 www.google.com으로 Allowed: directive를 기본으로 
초기 범위를 생성합니다.

.. code-block:: console

    cmlh$ ./rockspider.pl -www www.google.com

    "Rockspider" Alpha v0.1_2

    Copyright 2013 Christian Heinrich
    Licensed under the Apache License, Version 2.0

    1. Downloading http://www.google.com/robots.txt
    2. "robots.txt" saved as "www.google.com-robots.txt"
    3. Sending Allow: URIs of www.google.com to web proxy i.e.
    127.0.0.1:8080
     /catalogs/about sent
     /catalogs/p? sent
     /news/directory sent
    ...
    4. Done.

|

Google Webmaster 툴을 사용하여 robots.txt 분석
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

웹 사이트 관리자는 "Google Webmaster Tools"의 일부로 웹 사이트 분석 기능인
"Analyze robots.txt"를 사용할 수 있습니다. (https://www.google.com/webmasters/tools)
이 툴은 다음 절차로 테스트를 지원합니다.

1. 구글 계정으로 Google Webmaster Tools 가입
2. 대쉬보드에서 분석할 사이트 URL 기입
3. 이용할 방법을 선택하고 화면의 지시에 따릅니다.

|

<META> 태그
-----------------------------------------------------------------------------------------

<META> 태그는 각 HTML 문서의 HEAD 섹션에 위치하고 있습니다.

"<META NAME='ROBOTS' ... >"가 없다면 "Robots Exclusion Protocol"은 기본적으로 "INDEX,FOLLW"가 존재합니다.
그러므로, "Robots Exclusion Protocol"에 정의한 2개의 유효 항목은 "NOINDEX"와 "NOFOLLOW"와 같이 "No..."로 시작됩니다.

Web spiders/robots/crawlers는 의도적으로 robots.txt 파일과 같은 "<META NAME='ROBOTS'" 태그를 무시할 수 있습니다.
따라서, <META> 태그는 robots.txt에 보완할 수 있는 메커니즘으로 간주되서는 안됩니다.

|

<META> 태그 - Burp
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

웹 루트에 robots.txt 파일에 Disallow 지시어에 기초하여, 각각의 웹 페이지 내에 "<META NAME='ROBOTS'" 정규 표현식 검색은 웹 루트에 robots.txt 파일을 비교하는 결과를 수행합니다.

예를 들어, facebook.com의 robots.txt 파일은 "Disallow:/ac.php" 항목을 가지고 있습니다.
그리고 결과는 "<META NAME='ROBOTS'"를 찾는 것입니다. 

"INDEX,FOLLOW"는 "Robots Exclusion Protocol"에 의해 지정한 기본적인 <META> 태그 이며, "Disallow: /ac.php"는 robots.txt에 의해 목록화되어 있습니다.

|

Tools
==========================================================================================

- Browser (View Source function)
- curl
- wget
- rockspider

|

References
==========================================================================================

- "The Web Robots Pages": http://www.robotstxt.org/
- "Block and Remove Pages Using a robots.txt File": https://support.google.com/webmasters/answer/156449
- "(ISC)2 Blog: The Attack of the Spiders from the Clouds": http://blog.isc2.org/isc2_blog/2008/07/the-attack-of-t.html
- "Telstra customer database exposed": http://www.smh.com.au/it-pro/security-it/telstra-customer-database-exposed-20111209-1on60.html

|