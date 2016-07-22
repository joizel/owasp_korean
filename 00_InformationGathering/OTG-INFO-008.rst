==========================================================================================
OTG-INFO-008 (웹 어플리케이션 프레임워크 핑거프린트)
==========================================================================================

|

개요
==========================================================================================

웹 프레임워크 핑거프린트는 정보 수집 과정 중 중요한 작업 중 하나입니다.

이러한 프레임워크가 침투 테스터에 의해 이미 테스트 되었다면, 프레임워크의 유형을 아는 것은 자동으로 큰 이점을 제공 할 수 있습니다.

It is not only the known vulnerabilities in unpatched versions but specific misconfigurations in the framework and known file structure that makes the fingerprinting process so important.
Several different vendors and versions of web frameworks are widely
used. Information about it significantly helps in the testing process,
and can also help in changing the course of the test. Such information
can be derived by careful analysis of certain common locations. Most
of the web frameworks have several markers in those locations which
help an attacker to spot them. This is basically what all automatic tools
do, they look for a marker from a predefined location and then compare it to the database of known signatures. For better accuracy several markers are usually used.
[*] Please note that this article makes no differentiation between Web
Application Frameworks (WAF) and Content Management Systems
(CMS). This has been done to make it convenient to fingerprint both of
them in one chapter. Furthermore, both categories are referenced as
web frameworks.

|

테스트 목적
==========================================================================================

보안 테스트 방법에 대한 더 나은 이해를 갖기 위해 사용하는 웹 프레임워크 유형을 정의
    
|


테스트 방법
==========================================================================================

|

Black Box Testing
-----------------------------------------------------------------------------------------

현재 프레임워크를 정의하기 위해 찾을 수 있는 가장 일반적인 위치가 있습니다.

- HTTP 헤더
- Cookies
- HTML source code
- 특정 파일과 폴더

|

HTTP 헤더
-----------------------------------------------------------------------------------------

웹 프레임워크를 식별하는 가장 기본적인 형태는 HTTP response 헤더에서 X-Powered-By 필드를 찾는 것입니다.
많은 툴들이 목표에 대한 핑거프린트를 하는데 사용될 수 있습니다. 가장 간단한 것 중 하나가 netcat 유틸리티입니다. 다음 HTTP Request와 Response를 고려해봅니다.

.. code-block:: console

    $ nc 127.0.0.1 80
    HEAD / HTTP/1.0

    HTTP/1.1 200 OK
    Server: nginx/1.0.14
    Date: Sat, 07 Sep 2013 08:19:15 GMT
    Content-Type: text/html;charset=ISO-8859-1
    Connection: close
    Vary: Accept-Encoding
    X-Powered-By: Mono

X-Powered-By 필드로 부터, Mono라는 웹 어플리케이션 프레임워크인 걸 확인하였습니다.
이 방법은 간단하고 빠르지만, 100% 완벽하지는 않습니다.
설정에 의해 쉽게 X-Powered-By 헤더를 비활성화 할 수 있습니다.
또한, HTTP 헤더를 통해 난독화할 수 있는 몇가지 기술이 있습니다.
그래서, 아래 같은 예제에서 테스터는 X-Powered-By 헤더가 누락되거나 다음과 같은 응답을 얻을 수 있습니다.

.. code-block:: console

    HTTP/1.1 200 OK
    Server: nginx/1.0.14
    Date: Sat, 07 Sep 2013 08:19:15 GMT
    Content-Type: text/html;charset=ISO-8859-1
    Connection: close
    Vary: Accept-Encoding
    X-Powered-By: Blood, sweat and tears

때때로는 특정 웹 프레임워크를 가리키는 HTTP 헤더가 있습니다.
다음 예제에서, HTTP 리퀘스트로 부터 정보에 의해 PHP 버전을 포함하는 X-Powered-By 헤더를 볼 수 있습니다.
그러나, X-Generator 헤더는 실제로 사용되는 프레임워크가 Swiftlet를 가리키며, 침투 테스터는 공격 경로를 확장하는 데 도움이 됩니다.
핑거프린팅을 수행할 때, 항상 HTTP 헤더 노출에 대한 검사를 주의깊게 해야합니다.

.. code-block:: console

    HTTP/1.1 200 OK
    Server: nginx/1.4.1
    Date: Sat, 07 Sep 2013 09:22:52 GMT
    Content-Type: text/html
    Connection: keep-alive
    Vary: Accept-Encoding
    X-Powered-By: PHP/5.4.16-1~dotdeb.1
    Expires: Thu, 19 Nov 1981 08:52:00 GMT
    Cache-Control: no-store, no-cache, must-revalidate, postcheck=0, pre-check=0
    Pragma: no-cache
    X-Generator: Swiftlet

|

Cookies
-----------------------------------------------------------------------------------------

현재 웹 프레임워크를 결정하는 또 다른 유사한 방법은 프레임워크의 특정 쿠키 값입니다.

.. code-block:: console

    GET /cake HTTP /1.1
    Host: defcon-moscow.org
    User-Agent: Mozilla75.0 |Macintosh; Intel Mac OS X 10.7; rv:
    Accept: text/html, application/xhtml + xml, application/xml;
    Accept - Language: ru-ru, ru; q=0.8, en-us; q=0.5 , en; q=0 . 3
    Accept - Encoding: gzip, deflate
    DNT: 1
    Cookie: CAKEPHP=rm72kprivgmau5fmjdesbuqi71;
    Connection: Keep-alive
    Cache-Control: max-age=0

프레임워크 사용에 대한 정보를 CAKEPHP 쿠키로 자동 설정됩니다. 

.. code-block:: console

    /**
    * The name of CakePHP`s session cookie.
    *
    * Note the guidelines for Session names states: "The session
    name references
    * the session id in cookies and URLs. It should contain only alphanumeric
    * characters."
    * @link http://php.net/session_name
    */
    Configure::write('Session.cookie', 'CAKEPHP');

However, these changes are less likely to be made than changes
to the X-Powered-By header, so this approach can be considered
as more reliable.

|

HTML source code
-----------------------------------------------------------------------------------------

This technique is based on finding certain patterns in the HTML page source code. 
Often one can find a lot of information which helps a tester to recognize a specific web framework. 
One of the common markers are HTML comments that directly lead to framework disclosure.

More often certain framework-specific paths can be found, i.e. links to framework-specific css and/or js folders. 
Finally, specific script variables might also point to a certain framework.
From the screenshot below one can easily learn the used framework and its version by the mentioned markers. 

The comment, specific paths and script variables can all help an attacker to quickly determine an instance of ZK framework.

More frequently such information is placed between <head></head> tags, in <meta> tags or at the end of the page.
Nevertheless, it is recommended to check the whole document since it can be useful for other purposes such as inspection of other useful comments and hidden fields. 
Sometimes, web developers do not care much about hiding information about the framework used. 
It is still possible to stumble upon something like this at the bottom of the page:

|

특정 파일과 폴더
-----------------------------------------------------------------------------------------

Specific files and folders are different for each specific framework. It is recommended to install the corresponding framework
during penetration tests in order to have better understanding
of what infrastructure is presented and what files might be left
on the server. However, several good file lists already exist and
one good example is FuzzDB wordlists of predictable files/folders
(http://code.google.com/p/fuzzdb/).


|

Tools
==========================================================================================

A list of general and well-known tools is presented below. 
There are also a lot of other utilities, as well as framework-based fingerprinting tools.

WhatWeb
-----------------------------------------------------------------------------------------

http://www.morningstarsecurity.com/research/whatweb

Currently one of the best fingerprinting tools on the market. 
Included in a default Kali Linux build. Language: Ruby Matches for fingerprinting are made with:

- Text strings (case sensitive)
- Regular expressions
- Google Hack Database queries (limited set of keywords)
- MD5 hashes
- URL recognition
- HTML tag patterns
- Custom ruby code for passive and aggressive operations

Sample output is presented on a screenshot below:

|

BlindElephant
-----------------------------------------------------------------------------------------

https://community.qualys.com/community/blindelephant

This great tool works on the principle of static file checksum based
version difference thus providing a very high quality of fingerprinting. Language: Python

Sample output of a successful fingerprint:

.. code-block:: console

    pentester$ python BlindElephant.py http://my_target drupal
    Loaded /Library/Python/2.7/site-packages/blindelephant/
    dbs/drupal.pkl with 145 versions, 478 differentiating paths,
    and 434 version groups.
    Starting BlindElephant fingerprint for version of drupal at
    http://my_target
    
    Hit http://my_target/CHANGELOG.txt
    File produced no match. Error: Retrieved file doesn`t match
    known fingerprint. 527b085a3717bd691d47713dff74acf4
    
    Hit http://my_target/INSTALL.txt
    File produced no match. Error: Retrieved file doesn`t match
    known fingerprint. 14dfc133e4101be6f0ef5c64566da4a4
    
    Hit http://my_target/misc/drupal.js
    Possible versions based on result: 7.12, 7.13, 7.14

    Hit http://my_target/MAINTAINERS.txt
    File produced no match. Error: Retrieved file doesn`t match
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

Wapplyzer is a Firefox Chrome plug-in. It works only on regular expression matching and doesn`t need anything other than the page
to be loaded on browser. It works completely at the browser level
and gives results in the form of icons. Although sometimes it has
false positives, this is very handy to have notion of what technologies were used to construct a target website immediately after
browsing a page.

Sample output of a plug-in is presented on a screenshot below.

|

References
==========================================================================================

Whitepapers
-----------------------------------------------------------------------------------------

- Saumil Shah: "An Introduction to HTTP fingerprinting" - http://www.net-square.com/httprint_paper.html
- Anant Shrivastava : "Web Application Finger Printing" - http://anantshri.info/articles/web_app_finger_printing.html


|

Remediation
==========================================================================================

The general advice is to use several of the tools described above and check logs to better understand what exactly helps an attacker to disclose the web framework. 
By performing multiple scans after changes have been made to hide framework tracks, it`s possible to achieve a better level of security and to make sure of the framework can not be detected by automatic scans. 
Below are some specific recommendations by framework marker location and some additional interesting approaches.

**HTTP headers**

Check the configuration and disable or obfuscate all HTTP-headers that disclose information the technologies used. Here is an
interesting article about HTTP-headers obfuscation using Netscaler:
http://grahamhosking.blogspot.ru/2013/07/obfuscating-http-header-using-netscaler.html

|

**Cookies**

It is recommended to change cookie names by making changes in
the corresponding configuration files.

|

**HTML source code**

Manually check the contents of the HTML code and remove everything that explicitly points to the framework.

General guidelines:

- Make sure there are no visual markers disclosing the framework
- Remove any unnecessary comments (copyrights, bug information, specific framework comments)
- Remove META and generator tags
- Use the companies own css or js files and do not store those in a framework-specific folders
- Do not use default scripts on the page or obfuscate them if they must be used.

|

**Specific files and folders**

General guidelines:

- Remove any unnecessary or unused files on the server. This implies text files disclosing information about versions and installation too.
- Restrict access to other files in order to achieve 404-response when accessing them from outside. This can be done, for example, by modifying htaccess file and adding RewriteCond or RewriteRule there. An example of such restriction for two common WordPress folders is presented below.

.. code-block:: console

    RewriteCond %{REQUEST_URI} /wp-login\.php$ [OR]
    RewriteCond %{REQUEST_URI} /wp-admin/$
    RewriteRule $ /http://your_website [R=404,L]


However, these are not the only ways to restrict access. In order to
automate this process, certain framework-specific plugins exist.
One example for WordPress is StealthLogin (http://wordpress.org/
plugins/stealth-login-page).

|

**Additional approaches**

:General guidelines:

1. Checksum management

The purpose of this approach is to beat checksum-based scanners
and not let them disclose files by their hashes. Generally, there are
two approaches in checksum management:
- Change the location of where those files are placed (i.e. move them to another folder, or rename the existing folder)
- Modify the contents - even slight modification results in a completely different hash sum, so adding a single byte in the end of the file should not be a big problem.

2. Controlled chaos

A funny and effective method that involves adding bogus files and folders from other frameworks in order to fool scanners and confuse an attacker. But be careful not to overwrite existing files and folders and to break the current framework!

|