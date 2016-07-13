============================================================================================
OTG-INFO-002
============================================================================================

|

검색 엔진으로 정보 노출에 대한 검색 실행

|

Summary
============================================================================================

There are direct and indirect elements to search engine discovery and reconnaissance. 
Direct methods relate to searching the indexes and the associated content from caches.
Indirect methods relate to gleaning sensitive design and configuration information by searching forums, newsgroups, and tendering websites.

Once a search engine robot has completed crawling, it commences indexing the web page based on tags and associated attributes, such as <TITLE>, in order to return the relevant search results [1].
If the robots.txt file is not updated during the lifetime of the web site, and inline
HTML meta tags that instruct robots not to index content have not been used, then it is possible for indexes to contain web content not intended to be included in by the owners.
Website owners may use the previously mentioned robots.txt, HTML meta tags, authentication, and tools provided by search engines to remove such content.

.. note::

    검색 엔진으로 검색하는 방법에는 직접 방법과 간접 방법이 있습니다.
    직접 방법은 캐시로 부터 인덱스와 관련 컨텐츠를 검색하는 것 입니다.
    간접 방법은 검색 포럼, 뉴스 그룹, 지불 웹사이트에서 민감한 디자인과 구성 정보를 수집하는 것 입니다.
    
    검색 엔진 로봇이 수집이 완료되면, 관련 검색 결과를 리턴하기 위하여 <TITLE>과 같은 태그 또는 속성에 기반한 웹페이지를 인덱싱하기 시작합니다.
    만약 robots.txt 파일이 웹사이트의 라이프타임동안 업데이트 되지 않으면, HTML 메타 태그 내에 
    웹사이트 소유자는 위에서 언급한 robots.txt, HTML 메타 태그, 인증, 툴

|

Test Objectives
============================================================================================

To understand what sensitive design and configuration information of the application/system/organization is exposed both directly (on the organization’s website) or indirectly (on a third party website).

.. note::

    

|


How to Test
============================================================================================

검색 엔진 사용 리스트

- Network diagrams and configurations
- Archived posts and emails by administrators and other key staff
- Log on procedures and username formats
- Usernames and passwords
- Error message content
- Development, test, UAT and staging versions of the website

|

Search operators
============================================================================================

**site:** 옵션을 사용하여 특정 도메인에 대한 검색 결과를 제한할 수 있습니다.
    수집할 때 컨텐츠와 알고리즘에 따라 다른 결과를 생성 할 수 있으므로, 하나의 검색 엔진에 테스트를 제한하지 마십시오.

검색 엔진 종류 

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

site:owasp.org

[이미지]

저장된 owasp.org의 index.html을 보기위해 다음 구문을 사용합니다.

cache:owasp.org

[이미지]

|

Google Hacking Database
============================================================================================

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