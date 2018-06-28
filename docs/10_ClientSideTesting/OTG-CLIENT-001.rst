============================================================================================
OTG-CLIENT-001 (DOM 기반 크로스 사이트 스크립팅 침투 테스트)
============================================================================================

|

개요
============================================================================================

DOM 기반 크로스 사이트 스크립팅은 사실상 XSS 버그로 페이지의 JavaScript에서 사용자 입력을 얻은 다음 안전하지 않은 작업을 수행하여 삽입된 코드가 실행되게 합니다.
이 문서에서는 XSS로 이어지는 JavaScript 버그에 대해서만 설명합니다.

DOM(Documet Object Model)은 브라우져에서 문서를 나타내는 데 사용되는 구조 형식입니다.
DOM은 Javascript와 같은 동적 스크립트를 사용하여 form 필드 나 session cookie와 같은 문서의 구성 요소를 참조할 수 있습니다.
DOM은 다른 도메인의 스크립트가 다른 도메인의 session cookie를 가져오지 못하도록 제한하는 등의 보안을 위해서도 브라우저에서 사용됩니다.
공격자가 조작할 수 있는 DOM 요소와 같이 특수하게 조작된 요청으로 인해 JavaScript 기능과 같은 활성 콘텐츠가 수정되면 DOM 기반 XSS 취약점이 발생할 수 있습니다.
이 주제에 관한 논문은 거의 발표되지 않았기 때문에, 그 의미와 형식화된 테스트의 표준화가 거의 존재하지 않습니다.

|

테스트 방법
============================================================================================

모든 XSS 버그가 서버에서 반환된 콘텐츠를 공격자가 제어하는 것은 아니지만, 동일한 결과를 얻기 위해 잘못된 JavaScript 코딩 방법을 남용할 수 있습니다.
그 결과는 일반적인 XSS 결함과 동일하며 전달 방법만 다릅니다.

검증되지 않은 파라미터가 서버에서 전달되고 사용자에게 반환되어 사용자의 브라우저 컨텍스트에서 실행되는 다른 크로스 사이트 스크립팅 취약점 (reflected와 stored XSS)과 비교할 때 DOM 기반 XSS 취약점은 흐름을 변경하기 위해 공격자가 만든 코드와 함께 DOM(Document Object Model)의 요소를 사용하여 코드를 작성하십시오.
본래 DOM 기반 XSS 취약점은 서버가 실제로 무엇이 실행되고 있는지를 결정하지 않고 많은 경우에 실행될 수 있습니다. 
이로 인해 많은 일반적인 XSS 필터링 및 탐지 기술이 이러한 공격에 무력화 될 수 있습니다.

첫 번째 가상 예제는 다음 클라이언트 사이드를 사용합니다

.. code-block:: html

    <script>
    document.write("Site is at: " + document.location.href + ".");
    </script>


공격자는 영향을 받은 페이지 URL에 #<script>alert('xss')</script>를 추가할 수 있으며. 이 경우 실행될 때 경고 상자가 표시됩니다.
이 경우 #문자가 브라우저에서 쿼리의 일부로 처리되지 않고 조각으로 처리된 이 후 추가된 코드가 서버로 전송되지 않습니다.
이 예제에서는 코드가 즉시 실행되고 "xss"라는 경고가 페이지에 표시됩니다.

코드가 서버로 전송된 다음 다시 브라우저로 돌아오는 일반적인 크로스 사이트 스크립팅 유형(reflected와 stored XSS)과 달리 이는 서버와의 접촉없이 사용자의 브라우저에서 직접 실행됩니다.
DOM 기반 XSS 취약점 결과는 쿠키 검색, 추가 악의적인 스크립트 삽입 등을 포함하여 잘 알려진 형태 XSS와 마찬가지로 광범위합니다. 따라서 동일한 심각도로 조치되어야 합니다.

|

Black Box testing
----------------------------------------------------------------------------------------

DOM-Based XSS에 대한 블랙박스 테스팅은 대개 수행되지 않습니다. 
소스 코드에 대한 액세스는 실행될 클라이언트로 전송되어 항상 사용 가능하기 때문입니다.

|

Gray Box testing
----------------------------------------------------------------------------------------

DOM 기반 XSS 취약점 테스팅

Javascript 애플리케이션은 서버에 의해 동적으로 생성되는 경우가 많기 때문에 다른 유형의 애플리케이션과 다르며, 실행 중인 코드를 파악하기 위해 테스트중인 웹 사이트를 크롤링하여 사용자 입력을 받거나 실행 중인 Javascript의 모든 인스턴스를 확인해야합니다. 

많은 웹 사이트는 수십만 줄의 코드로 확장되고, 사내에서 개발되지 않은 수많은 함수 라이브러리에 의존합니다.
이러한 경우 top-down 테스트는 많은 최하위 레벨 기능이 사용되지 않으므로 실제로 가능한 유일한 옵션이 되며, 싱크가 무엇인지 결정하기 위해 이를 분석하여 자주 사용 가능한 것보다 더 많은 시간을 소비하게 됩니다.

입력 또는 그 부족이 처음부터 확인되지 않은 경우에도 top-down 테스트를 수행할 수 있습니다.
사용자 입력은 두 가지 기본 형식으로 제공됩니다.

- Input written to the page by the server in a way that does not allow direct XSS
- Input obtained from client-side JavaScript objects

Here are two examples of how the server may insert data into JavaScript:

.. code-block:: javascript

    var data = "<escaped data from the server>";
    var result = someFunction("<escaped data from the server>");


And here are two examples of input from client-side JavaScript objects:

.. code-block:: javascript

    var data = window.location;
    var result = someFunction(window.referer);

While there is little difference to the JavaScript code in how they are retrieved, it is important to note that when input is received via the server, the server can apply any permutations to the data that it desires, whereas the permutations performed by JavaScript objects are fairly well understood and documented, and so if someFunction in the above example were a sink, then the exploitability of the former would depend on the filtering done by the server, whereas the latter would depend on the encoding done by the browser on the window.referer object.
Stefano Di Paulo has written an excellent article on what browsers return when asked for the various elements of a URL using the document. and location. attributes.
Additionally, JavaScript is often executed outside of <script> blocks, as evidenced by the many vectors which have led to XSS filter bypasses in the past, and so, when crawling the application, it is important to note the use of scripts in places such as event handlers and CSS blocks with expression attributes.
Also, note that any off-site CSS or script objects will need to be assessed to determine what code is being executed.
Automated testing has only very limited success at identifying and validating DOM-based XSS as it usually identifies XSS by sending a specific payload and attempts to observe it in the server response. 
This may work fine for the simple example provided below, where the message parameter is reflected back to the user:


.. code-block:: html

    <script>
    var pos=document.URL.indexOf("message=")+5;
    document.write(document.URL.substring(pos,document.URL.length));
    </script>

but may not be detected in the following contrived case:

.. code-block:: html

    <script>
    var navAgt = navigator.userAgent;
     
    if (navAgt.indexOf("MSIE")!=-1) {
         document.write("You are using IE as a browser and visiting site: " + document.location.href + ".");
    }
    else
    {
        document.write("You are using an unknown browser.");
    }
    </script>

For this reason, automated testing will not detect areas that may be susceptible to DOM-based XSS unless the testing tool can perform addition analysis of the client side code.
이러한 이유로 자동화된 테스트는 테스트 도구가 클라이언트 측 코드에 대한 추가 분석을 수행할 수 없는 경우 DOM 기반 XSS의 영향을 받을 수 있는 영역을 감지하지 못합니다.


Manual testing should therefore be undertaken and can be done by examining areas in the code where parameters are referred to that may be useful to an attacker. 
Examples of such areas include places where code is dynamically written to the page and elsewhere where the DOM is modified or even where scripts are directly executed. 
Further examples are described in the excellent DOM XSS article by Amit Klein, referenced at the end of this section.

|

References
============================================================================================

OWASP Resources
-------------------------------------------------------------------------------------------

- DOM based XSS Prevention Cheat Sheet

|

Whitepapers
-------------------------------------------------------------------------------------------

- Document Object Model (DOM) 
    - http://en.wikipedia.org/wiki/Document_Object_Model
- DOM Based Cross Site Scripting or XSS of the Third Kind 
    - Amit Klein: http://www.webappsec.org/projects/articles/071105.shtml
- Browser location/document URI/URL Sources 
    - https://code.google.com/p/domxsswiki/wiki/LocationSources
- i.e., what is returned when the user asks the browser for things like document.URL, document.baseURI, location, location.href, etc.

|