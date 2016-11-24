==========================================================================================
OTG-INFO-005 (정보 노출을 확인하기 위해 웹 페이지 주석과 메타 데이터 확인)
==========================================================================================

|

개요
==========================================================================================

프로그래머가 소스코드에 대한 상세한 주석 및 메타 데이터를 포함시키는 것은 매우 흔한 일이고, 심지어 권고됩니다.
그러나, HTML 코드에 포함된 주석과 메타 데이터는 잠재적인 공격자에게 제공해서는 안될 내부 정보를 공개할 수도 있습니다.
주석과 메타 데이터는 정보 유출 여부를 판단하기 위해 확인되어야 합니다.


|

테스트 목적
==========================================================================================

응용 프로그램을 더 잘 이해하고 모든 정보 유출을 찾기 위해 웹 페이지 주석과 메타 데이터를 확인

|


테스트 방법
==========================================================================================

HTML 주석은 종종 응용 프로그램에 관한 디버깅 정보를 포함하도록 개발자에 의해 사용됩니다.
테스터는 ""을 시작으로 하는 HTML 주석을 찾을 것 입니다.

|

Black Box Testing
-----------------------------------------------------------------------------------------

응용 프로그램에 관한 민감한 정보를 포함하는 HTML 소스코드 주석을 확인하십시오.
SQL 코드, 사용자명, 패스워드, 내부 IP 주소, 디버깅 정보등이 그것입니다.

.. code-block:: html

    ...
    <div class="table2">
        <div class="col1">1</div><div class="col2">Mary</div>
        <div class="col1">2</div><div class="col2">Peter</div>
        <div class="col1">3</div><div class="col2">Joe</div>
    <!-- Query: SELECT id, name FROM app.users WHERE active='1' -->
    </div>
    ...

테스트를 하다보면 아래와 같은 주석 정보도 찾을 수 있습니다.

.. code-block:: html

    <!-- Use the DB administrator password for testing: f@keP@a$$w0rD -->

유효한 버전 번호 및 DTD(Data Type Definition) URL에 대한 HTML 버전 정보 확인

.. code-block:: html

    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">

- "strict.dtd" -- default strict DTD
- "loose.dtd" -- loose DTD
- "frameset.dtd" -- DTD for frameset documents

일부 메타 태그는 유효 공격 벡터를 제공하지 않지만, 대신 공격자가 응용 프로그램의 프로필을 작성하도록 허용합니다.

.. code-block:: html

    <META name="Author" content="Andrew Muller">

일부 메타 태그는 다음과 같이 메타 요소의 content 속성을 기반으로 HTTP 응답 헤더를 설정하는 http-equiv와 같은 HTTP 응답 헤더를 변경합니다.

.. code-block:: html

    <META http-equiv="Expires" content="Fri, 21 Dec 2012 12:34:56 GMT">

**예상 결과**

.. code-block:: html

    Expires: Fri, 21 Dec 2012 12:34:56 GMT


.. code-block:: html

    <META http-equiv="Cache-Control" content="no-cache">

**예상 결과**

.. code-block:: html

    Cache-Control: no-cache

인젝션 공격(CRLF 공격)을 수행하는 데 사용 할 수 있는지 테스트합니다.

It can also help determine the level of data leakage via the browser cache.
또한 브라우저 캐시를 통해 데이터 유출 수준을 결정하는데도 도움이 됩니다.

.. code-block:: html

    <META http-equiv="Refresh" content="15;URL=https://www.owasp.org/index.html">

메타 태그의 일반적인 용도는 검색 엔진이 검색 결과의 품질을 높이기 위해 사용 할 수 있는 키워드를 지정하는 것입니다.

.. code-block:: html

    <META name="keywords" lang="en-us" content="OWASP, security,sunshine, lollipops">

대부분의 웹 서버는 robots.txt 파일을 통해 검색 엔진 색인 생성을 관리하지만 메타 태그로 관리 할 수도 있습니다.
아래의 태그는 로봇이 태그를 포함하지 않고 해당 태그가 포함 된 HTML 페이지의 링크를 따르지 않도록 조언합니다

.. code-block:: html

    <META name="robots" content="none"> 


|

Gray Box Testing
-----------------------------------------------------------------------------------------

Not applicable.

|

도구
==========================================================================================

- Wget
- Browser "view source" function
- Eyeballs
- Curl

|