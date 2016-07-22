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

어플리케이션을 더 잘 이해하고 모든 정보 유출을 찾기 위해 웹 페이지 주석과 메타 데이터를 확인

|


테스트 방법
==========================================================================================

HTML 주석은 종종 어플리케이션에 관한 디버깅 정보를 포함하도록 개발자에 의해 사용됩니다.
테스터는 ""을 시작으로 하는 HTML 주석을 찾을 것 입니다.

|

Black Box Testing
-----------------------------------------------------------------------------------------

Check HTML source code for comments containing sensitive information that can help the attacker gain more insight about the application.
It might be SQL code, usernames and passwords, internal IP addresses, or debugging information.

.. code-block:: html

    ...
    <div class="table2">
     <div class="col1">1</div><div class="col2">Mary</div>
     <div class="col1">2</div><div class="col2">Peter</div>
     <div class="col1">3</div><div class="col2">Joe</div>
    <!-- Query: SELECT id, name FROM app.users WHERE active='1' -->
    </div>
    ...

The tester may even find something like this:

.. code-block:: html

    <!-- Use the DB administrator password for testing: f@keP@a$$w0rD -->

Check HTML version information for valid version numbers and Data Type Definition (DTD) URLs

.. code-block:: html

    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">

- "strict.dtd" -- default strict DTD
- "loose.dtd" -- loose DTD
- "frameset.dtd" -- DTD for frameset documents

Some Meta tags do not provide active attack vectors but instead allow
an attacker to profile an application to

.. code-block:: html

    <META name="Author" content="Andrew Muller">

Some Meta tags alter HTTP response headers, such as http-equiv that sets an HTTP response header based on the the content attribute of a meta element, such as:

.. code-block:: html

    <META http-equiv="Expires" content="Fri, 21 Dec 2012 12:34:56 GMT">

which will result in the HTTP header:

.. code-block:: html

    Expires: Fri, 21 Dec 2012 12:34:56 GMT

and

.. code-block:: html

    <META http-equiv="Cache-Control" content="no-cache">

will result in

.. code-block:: html

    Cache-Control: no-cache

Test to see if this can be used to conduct injection attacks (e.g. CRLF
attack). It can also help determine the level of data leakage via the
browser cache.

A common (but not WCAG compliant) Meta tag is the refresh.

.. code-block:: html

    <META http-equiv="Refresh" content="15;URL=https://www.owasp.org/index.html">

A common use for Meta tag is to specify keywords that a search engine
may use to improve the quality of search results.

.. code-block:: html

    <META name="keywords" lang="en-us" content="OWASP, security,sunshine, lollipops">

Although most web servers manage search engine indexing via the
robots.txt file, it can also be managed by Meta tags. The tag below
will advise robots to not index and not follow links on the HTML page
containing the tag

.. code-block:: html

    <META name="robots" content="none"> 


The Platform for Internet Content Selection (PICS) and Protocol for
Web Description Resources (POWDER) provide infrastructure for associating
meta data with Internet content.

|

Gray Box Testing
-----------------------------------------------------------------------------------------

Not applicable.

|


Tools
==========================================================================================

- Wget
- Browser "view source" function
- Eyeballs
- Curl


|

References
==========================================================================================

Whitepapers
-----------------------------------------------------------------------------------------

- http://www.w3.org/TR/1999/REC-html401-19991224 HTML version 4.01
- http://www.w3.org/TR/2010/REC-xhtml-basic-20101123/ XHTML(for small devices)
- http://www.w3.org/TR/html5/ HTML version 5

|