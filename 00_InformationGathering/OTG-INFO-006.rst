==========================================================================================
OTG-INFO-006 (어플리케이션 엔트리 포인트 식별)
==========================================================================================

|

개요
==========================================================================================

Enumerating the application and its attack surface is a key precursor
before any thorough testing can be undertaken, as it allows the tester
to identify likely areas of weakness. This section aims to help identify
and map out areas within the application that should be investigated
once enumeration and mapping have been completed.
    

|

테스트 목적
==========================================================================================

리퀘스트가 어떻게 형성되는 지와 어플리케이션으로 부터 전형적인 응답 이해
    
|


테스트 방법
==========================================================================================

|

Before any testing begins, the tester should always get a good understanding
of the application and how the user and browser communicates
with it. As the tester walks through the application, they should
pay special attention to all HTTP requests (GET and POST Methods,
also known as Verbs), as well as every parameter and form field that
is passed to the application. In addition, they should pay attention to
when GET requests are used and when POST requests are used to
pass parameters to the application. It is very common that GET requests
are used, but when sensitive information is passed, it is often
done within the body of a POST request.

Note that to see the parameters sent in a POST request, the tester will
need to use a tool such as an intercepting proxy (for example, OWASP:
Zed Attack Proxy (ZAP)) or a browser plug-in. Within the POST request,
the tester should also make special note of any hidden form fields that
are being passed to the application, as these usually contain sensitive
information, such as state information, quantity of items, the price of
items, that the developer never intended for you to see or change.

In the author’s experience, it has been very useful to use an intercepting
proxy and a spreadsheet for this stage of the testing. The proxy
will keep track of every request and response between the tester and
the application as they u walk through it. Additionally, at this point,
testers usually trap every request and response so that they can
see exactly every header, parameter, etc. that is being passed to the
application and what is being returned. This can be quite tedious at
times, especially on large interactive sites (think of a banking application).
However, experience will show what to look for and this phase
can be significantly reduced.

As the tester walks through the application, they should take note
of any interesting parameters in the URL, custom headers, or body
of the requests/responses, and save them in a spreadsheet. The
spreadsheet should include the page requested (it might be good to
also add the request number from the proxy, for future reference),
the interesting parameters, the type of request (POST/GET), if access
is authenticated/unauthenticated, if SSL is used, if it’s part of
a multi-step process, and any other relevant notes. Once they have
every area of the application mapped out, then they can go through
the application and test each of the areas that they have identified
and make notes for what worked and what didn’t work. The rest of
this guide will identify how to test each of these areas of interest, but
this section must be undertaken before any of the actual testing can
commence.

Below are some points of interests for all requests and responses.
Within the requests section, focus on the GET and POST methods,
as these appear the majority of the requests. Note that other methods,
such as PUT and DELETE, can be used. Often, these more rare
requests, if allowed, can expose vulnerabilities. There is a special section
in this guide dedicated for testing these HTTP methods.

|

Requests
-----------------------------------------------------------------------------------------

- Identify where GETs are used and where POSTs are used.
- Identify all parameters used in a POST request (these are in the body of the request).
- Within the POST request, pay special attention to any hidden parameters. When a POST is sent all the form fields (including hidden parameters) will be sent in the body of the HTTP message to the application. These typically aren’t seen unless a proxy or view the HTML source code is used. In addition, the next page shown, its data, and the level of access can all be different depending on the value of the hidden parameter(s).
- Identify all parameters used in a GET request (i.e., URL), in particular the query string (usually after a ? mark).
- Identify all the parameters of the query string. These usually are in a pair format, such as foo=bar. Also note that many parameters can be in one query string such as separated by a &, ~, :, or any other special character or encoding.
- A special note when it comes to identifying multiple parameters in one string or within a POST request is that some or all of the parameters will be needed to execute the attacks. The tester needs to identify all of the parameters (even if encoded or encrypted) and identify which ones are processed by the application. Later sections of the guide will identify how to test these parameters. At this point, just make sure each one of them is identified.
- Also pay attention to any additional or custom type headers not typically seen (such as debug=False).

|

Responses
-----------------------------------------------------------------------------------------

- Identify where new cookies are set (Set-Cookie header), modified, or added to.
- Identify where there are any redirects (3xx HTTP status code), 400 status codes, in particular 403 Forbidden, and 500 internal server errors during normal responses (i.e., unmodified requests).
- Also note where any interesting headers are used. For example, "Server: BIG-IP" indicates that the site is load balanced. Thus, if a site is load balanced and one server is incorrectly configured, then the tester might have to make multiple requests to access the vulnerable server, depending on the type of load balancing used.

|

Black Box Testing
-----------------------------------------------------------------------------------------

어플리케이션 엔트리 포인트 테스트:

어플리케이션 엔트리 포인트를 체크하는 방법은 다음 두가지 예가 있습니다.

EXAMPLE 1
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

이 예제는 온라인 쇼핑 어플리케이션으로 부터 아이템을 구매하는 GET 리퀘스트를 보여줍니다.

.. code-block:: console

    GET https://x.x.x.x/shoppingApp/buyme.asp?CUSTOMERID=100&ITEM=z101a&PRICE=62.50&IP=x.x.x.x

    Host: x.x.x.x
    Cookie: SESSIONID=Z29vZCBqb2IgcGFkYXdhIG15IHVzZXJuYW1lIGlzIGZvbyBhbmQgcGFzc3dvcmQgaXMgYmFy

:Result Expected:

여기 테스터는 리퀘스트의 파라미터 모두를 기록해야 합니다.
(CUSTOMERID, ITEM, PRICE, IP, Cookie)

|

EXAMPLE 2
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

이 예제는 어플리케이션으로 로그인하는 POST 리퀘스트를 보여줍니다.

.. code-block:: console

    POST https://x.x.x.x/KevinNotSoGoodApp/authenticate.asp?-service=login
    Host: x.x.x.x
    Cookie: SESSIONID=dGhpcyBpcyBhIGJhZCBhcHAgdGhhdCBzZXRzIHByZWRpY3RhYmxlIGNvb2tpZXMgYW5kIG1pbmUgaXMgMTIzNA==
    CustomCookie=00my00trusted00ip00is00x.x.x.x00

:POST Body:

.. code-block:: console

    user=admin&pass=pass123&debug=true&fromtrustIP=true

:Result Expected:

이 예제에서 테스터는 POST Body 상에 모든 파라미터를 기록해야 합니다.
또한, Cookie값과 CustomCookie값 역시 기록합니다.

|

Gray Box Testing
==========================================================================================

Testing for application entry points via a Gray Box methodology
would consist of everything already identified above with one addition.
In cases where there are external sources from which the application
receives data and processes it (such as SNMP traps, syslog
messages, SMTP, or SOAP messages from other servers) a meeting
with the application developers could identify any functions that
would accept or expect user input and how they are formatted. For
example, the developer could help in understanding how to formulate
a correct SOAP request that the application would accept and
where the web service resides (if the web service or any other function
hasn’t already been identified during the black box testing).

|

Tools
==========================================================================================

Intercepting Proxy
-----------------------------------------------------------------------------------------

- OWASP: Zed Attack Proxy (ZAP)
- OWASP: WebScarab
- Burp Suite
- CAT

Browser Plug-in
-----------------------------------------------------------------------------------------

- Internet Explorer: TamperIE 
- Firefox: Tamper Data 

|

References
==========================================================================================

Whitepapers
-----------------------------------------------------------------------------------------

- RFC 2616 – Hypertext Transfer Protocol – HTTP 1.1 - http://tools.ietf.org/html/rfc2616

|
