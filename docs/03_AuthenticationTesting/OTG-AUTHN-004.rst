==========================================================================================
OTG-AUTHN-004 (인증 스키마를 우회하기 위한 테스트)
==========================================================================================

|

개요
==========================================================================================

While most applications require authentication to gain access to
private information or to execute tasks, not every authentication
method is able to provide adequate security. Negligence, ignorance,
or simple understatement of security threats often result
in authentication schemes that can be bypassed by simply skipping
the log in page and directly calling an internal page that is
supposed to be accessed only after authentication has been performed.
In addition, it is often possible to bypass authentication measures
by tampering with requests and tricking the application into
thinking that the user is already authenticated. This can be accomplished
either by modifying the given URL parameter, by manipulating
the form, or by counterfeiting sessions.
Problems related to the authentication schema can be found at
different stages of the software development life cycle (SDLC), like
the design, development, and deployment phases:
• In the design phase errors can include a wrong definition of
application sections to be protected, the choice of not applying
strong encryption protocols for securing the transmission of
credentials, and many more.
• In the development phase errors can include the incorrect
 implementation of input validation functionality or not following
the security best practices for the specific language.
• In the application deployment phase, there may be issues during
 the application setup (installation and configuration activities)
due to a lack in required technical skills or due to the lack of good
documentation.

|

테스트 방법
==========================================================================================

|

Black Box testing
-----------------------------------------------------------------------------------------

웹 어플리케이션에서 사용되는 인증 스키마를 우회하는 방법은 여러가지 있습니다.

- Direct page request (forced browsing)
- Parameter modification
- Session ID prediction
- SQL injection

**Direct page request**

만약 웹 어플리케이션이 페이지에서의 로그인만 접근 제어를 구현해두었다면, 인증 스키마는 
우회될 것입니다.
예를 들어, 사용자가 직접 강제 브라우징을 통해 다른 페이지를 요청하면, 해당 페이지는 접속 권한을 부여하기 전에 사용자의 자격 증명을 확인하지 않을 수 있습니다.
이 방법을 사용한 테스트는 브라우저에 주소 창을 통해 보호된 페이지를 직접 접근하는 시도입니다.


**Parameter Modification**

Another problem related to authentication design is when the application
verifies a successful log in on the basis of a fixed value
parameters. A user could modify these parameters to gain access
to the protected areas without providing valid credentials. In the
example below, the “authenticated” parameter is changed to a
value of “yes”, which allows the user to gain access. In this example,
the parameter is in the URL, but a proxy could also be used to
modify the parameter, especially when the parameters are sent
as form elements in a POST request or when the parameters are
stored in a cookie. 

.. code-block:: console

    http://www.site.com/page.asp?authenticated=no

.. code-block:: console

    raven@blackbox /home $nc www.site.com 80
    GET /page.asp?authenticated=yes HTTP/1.0

    HTTP/1.1 200 OK
    Date: Sat, 11 Nov 2006 10:22:44 GMT
    Server: Apache
    Connection: close
    Content-Type: text/html; charset=iso-8859-1
    <!DOCTYPE HTML PUBLIC “-//IETF//DTD HTML 2.0//EN”>
    <HTML><HEAD>
    </HEAD><BODY>
    <H1>You Are Authenticated</H1>
    </BODY></HTML>

**Session ID Prediction**

Many web applications manage authentication by using session
identifiers (session IDs). Therefore, if session ID generation is
predictable, a malicious user could be able to find a valid session ID
and gain unauthorized access to the application, impersonating a
previously authenticated user.
In the following figure, values inside cookies increase linearly, so it
could be easy for an attacker to guess a valid session ID.

n the following figure, values inside cookies change only partially, so
it’s possible to restrict a brute force attack to the defined fields shown
below.

**SQL Injection (HTML Form Authentication)**

SQL Injection is a widely known attack technique. This section is not
going to describe this technique in detail as there are several sections
in this guide that explain injection techniques beyond the scope of
this section.

The following figure shows that with a simple SQL injection attack,
it is sometimes possible to bypass the authentication form.

|

Gray Box Testing
-----------------------------------------------------------------------------------------

If an attacker has been able to retrieve the application source code
by exploiting a previously discovered vulnerability (e.g., directory
traversal), or from a web repository (Open Source Applications),
it could be possible to perform refined attacks against the
implementation of the authentication process.
In the following example (PHPBB 2.0.13 - Authentication Bypass
Vulnerability), at line 5 the unserialize() function parses a user
supplied cookie and sets values inside the $row array. At line
10 the user’s MD5 password hash stored inside the back end
database is compared to the one supplied.
In PHP, a comparison between a string value and a boolean value 

.. code-block:: html

    1. if ( isset($HTTP_COOKIE_VARS[$cookiename . ‘_sid’]) ||
    2. {
    3. $sessiondata = isset( $HTTP_COOKIE_VARS[$cookiename
    . ‘_data’] ) ?
    4.
    5. unserialize(stripslashes($HTTP_COOKIE_VARS[$cookiename
    . ‘_data’])) : array();
    6.
    7. $sessionmethod = SESSION_METHOD_COOKIE;
    8. }
    9.
    10. if( md5($password) == $row[‘user_password’] &&
    $row[‘user_active’] )
    11.
    12. {
    13. $autologin = ( isset($HTTP_POST_VARS[‘autologin’]) ) ?
    TRUE : 0;
    14. }

(1 - “TRUE”) is always “TRUE”, so by supplying the following string
(the important part is “b:1”) to the unserialize() function, it is
possible to bypass the authentication control:

.. code-block:: html

    a:2:{s:11:”autologinid”;b:1;s:6:”userid”;s:1:”2”;}


|

Tools
==========================================================================================

- WebScarab
- WebGoat
- OWASP Zed Attack Proxy (ZAP)

|

References
==========================================================================================

Whitepapers
-----------------------------------------------------------------------------------------

• Mark Roxberry: “PHPBB 2.0.13 vulnerability”
• David Endler: “Session ID Brute Force Exploitation and Prediction”
- http://www.cgisecurity.com/lib/SessionIDs.pdf


|