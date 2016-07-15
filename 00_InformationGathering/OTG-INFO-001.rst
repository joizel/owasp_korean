============================================================================================
OTG-INFO-001
============================================================================================

|

정보 노출 확인을 위해 써치 엔진으로 검색 실행

|

개요
============================================================================================

써치 엔진으로 정보 노출을 검색하는 방법에는 직접 방법과 간접 방법이 있습니다.
직접 방법은 저장된 페이지로 부터 인덱스와 관련 컨텐츠를 검색하는 것 입니다.
간접 방법은 검색 포럼, 뉴스 그룹, 결재 웹 사이트에서 민감한 디자인과 구성 정보를 수집하는 것 입니다.

써치 엔진 로봇을 통한 수집이 완료되면, 관련 검색 결과를 리턴하기 위해 <TITLE>과 같은 태그 또는 관련 속성에 기초하여 상기 웹페이지를 인덱싱하기 시작합니다.
만약 robots.txt 파일이 웹사이트가 라이브할 동안 업데이트 되지 않았을 경우, 로봇이 명령한 내부 HTML 메타 태그 검색 결과가 관리자에 의해 포함된 것이 아닌 웹 컨텐츠를 포함한 인덱스가 될 수 있습니다.

웹사이트 관리자는 이러한 컨텐츠를 제거하기 위해 써치 엔진에 의해 제공되는 robots.txt, HTML 메타 태그, 인증, 툴 등을 사용할 수 있습니다.

|

테스트 목적
============================================================================================

어플리케이션/시스템/기관의 민감한 디자인과 설정 정보가 
직접 또는 간접적으로 노출되는 것 이해하기

|

테스트 방법
============================================================================================

써치 엔진 사용

- 네트워크 다이어그램 및 구성
- 관리자 및 기타 주요 직원에 의해 기록된 게시물과 이메일
- 절차 및 사용자명 형식 로그인
- 사용자명 및 패스워드
- 에러 메시지 컨텐츠
- 개발, 테스트, UAT, 웹사이트 버전

|

Search operators
-------------------------------------------------------------------------------------------

**site:** 옵션을 사용하여 특정 도메인에 대한 검색 결과를 제한할 수 있습니다.

수집할 때 컨텐츠와 알고리즘에 따라 다른 결과를 생성 할 수 있으므로, 하나의 검색 엔진에 테스트를 제한하지 마십시오.

검색 엔진 종류 

.. code-block:: html

    - Baidu
    - binsearch.info
    - Bing
    - Duck Duck Go
    - ixquick/Startpage
    - Google
    - Shodan
    - PunkSpider


Duck Duck Go와 ixquick/Startpage는 테스터에 관한 정보 유출을 최소화할 수 있습니다.

Google은 **cache:** 옵션을 제공하는데, 이 옵션은 Google 검색 결과에 **저장된 페이지** 와 동일합니다.
따라서, **site:** 옵션을 사용한 후 **저장된 페이지**를 클릭하면 됩니다.
Google SOAP Search API는 저장된 페이지 검색을 지원하기 위해 doGetCachedPage 와 관련 doGetCachedPageResponse SOAP 메시지를 지원합니다.
해당 구현은 OWASP **Google Hacking** 프로젝트에 의해 개발 중입니다.
PunkSpider는 웹 어플리케이션 취약점 검색 엔진입니다. 침투 테스터가 수동 작업을 하기 위한 용도로 사용됩니다.
그러나 스크립트 초보자들이 취약점을 찾는데 데모로 유용하게 사용됩니다.

**Example** 

일반적인 검색 엔진으로 owasp.org의 웹 컨텐츠를 찾기 위해, 다음 구문을 사용합니다.


.. code-block:: html

    site:owasp.org


저장된 owasp.org의 index.html을 보기위해 다음 구문을 사용합니다.


.. code-block:: html

    cache:owasp.org


|

Google Hacking Database
-------------------------------------------------------------------------------------------

Google Hacking Database는 Google으로 유용한 검색 쿼리 리스트입니다.

Google 검색 쿼리 종류

- Footholds
- Files containing usernames
- Sensitive Directories
- Web Server Detection
- Vulnerable Files
- Vulnerable Servers
- Error Messages
- Files containing juicy info
- Files containing passwords
- Sensitive Online Shopping Info

|

Tools
============================================================================================

[4] FoundStone SiteDigger: http://www.mcafee.com/uk/downloads/free-tools/sitedigger.aspx
[5] Google Hacker: http://yehg.net/lab/pr0js/files.php/googlehacker.zip
[6] Stach & Liu’s Google Hacking Diggity Project: http://www.stachliu.com/resources/tools/google-hacking-diggity-project/
[7] PunkSPIDER: http://punkspider.hyperiongray.com/


|

References
============================================================================================

[1] “Google Basics: Learn how Google Discovers, Crawls, and Serves Web Pages” - https://support.google.com/webmasters/answer/70897
[2] “Operators and More Search Help”: https://support.google.com/websearch/answer/136861?hl=en
[3] “Google Hacking Database”: http://www.exploit-db.com/google-dorks/


|

Remediation
============================================================================================

Carefully consider the sensitivity of design and configuration information before it is posted online.
Periodically review the sensitivity of existing design and configuration
information that is posted online.

|