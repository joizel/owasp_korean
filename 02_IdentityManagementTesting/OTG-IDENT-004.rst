============================================================================================
OTG-IDENT-004 (계정 리스트 및 추측 가능한 사용자 테스트)
============================================================================================

|

개요
============================================================================================

The scope of this test is to verify if it is possible to collect a set of valid usernames by interacting with the authentication mechanism of the application. This test will be useful for brute force testing, in which the tester verifies if, given a valid username, it is possible to find the corresponding password. 

Often, web applications reveal when a username exists on system, either as a consequence of mis-configuration or as a design decision. For example, sometimes, when we submit wrong credentials, we receive a message that states that either the user-name is present on the system or the provided password is wrong. The information obtained can be used by an attacker to gain a list of users on system. This information can be used to attack the web application, for example, through a brute force or default username and password attack. 

The tester should interact with the authentication mechanism of the application to understand if sending particular requests causes the application to answer in different manners. This issue exists because the information released from web application or web server when the user provide a valid username is different than when they use an invalid one. 

In some cases, a message is received that reveals if the provided credentials are wrong because an invalid username or an invalid password was used. Sometimes, testers can enumerate the existing users by sending a username and an empty password. 

|

테스트 방법
============================================================================================

|

Black Box testing
-----------------------------------------------------------------------------------------

테스터는 특정 어플리케이션, 사용자 이름, 어플리케이션 로직, 로그 페이지에서 오류 메시지, 또는 비밀 번호 복구 과정에 대해 아무것도 모른다.
만약 어플리케이션이 취약하다면, 테스터는 응답 메시지를 통해 사용자 리스트에 대한 유용한 정보를 받을 수 있습니다.

|

HTTP 응답 메시지를 통한 사용자 리스팅
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**유효한 사용자 및 올바른 패스워드 테스트**

유효한 사용자 ID와 유효한 패스워드를 입력할 때 서버 응답 저장

**결과**

WebScarab를 사용하면, 성공적인 인증에 대한 정보를 알 수 있습니다.
(HTTP 200 Response, response 길이)

|

**유효한 사용자 및 잘못된 패스워드 테스트**

이제, 테스터는 유효한 사용자 ID와 잘못된 암호를 삽입하고 어플리케이션에 의해 생성된 오류 메시지를 저장하려고 합니다.

**결과**

브라우저는 다음과 유사한 메시지를 보여줄 것입니다.

.. code-block:: html

    Authentication failed

    Return to Login page

또는 다음과 같은 메시지를 보여줄 것입니다.

.. code-block:: html

    No configuration found.
    Contact your system administrator

    Return to Login page

사용자 존재를 보여주는 메시지는 아래와 같습니다.

.. code-block:: html

    Login for User foo: invalid password 
    
WebScarab를 사용하면, 인증 실패 시도에 대한 검색 정보를 알 수 있습니다.
(HTTP 200 Response, response 길이) 

|

**존재하지 않는 사용자명 테스트**

이제, 테스터는 잘못된 ID와 password를 삽입한 뒤, 서버 응답을 저장할 것입니다.

**결과**

만약 테스터가 존재하지 않는 ID를 입력했다면, 다음과 유사한 메시지를 보낼 것입니다. 

.. code-block:: html

    Login failed for User foo: invalid Account 

일반적으로 어플리케이션은 잘못된 요청에 대해 에러 메시지와 길이로 응답합니다.
만약 응답이 같지 않다면, 테스터는 두 응답의 차이를 발생시키는 키를 찾고 조사해야합니다.


예제: 

- 클라이언트 요청: 유효한 사용자/잘못된 패스워드 --> 서버응답:'The password is not correct' 
- 클라이언트 요청: 잘못된 사용자/잘못된 패스워드 --> 서버응답:'User not recognized' 

The above responses let the client understand that for the first request they have a valid user name. So they can interact with the application requesting a set of possible user IDs and observing the answer. 

Looking at the second server response, the tester understand in the same way that they don't hold a valid username. So they can interact in the same manner and create a list of valid user ID looking at the server answers. 

|

사용자 리스팅 또다른 방법
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

테스터는 다음과 같이 여러가지 방법으로 사용자 리스팅을 할 수 있습니다.

**로그인 페이지에서 받는 에러 코드 분석**

일부 웹 어플리케이션은 지정한 에러 코드 또는 분석할 수 있는 메시지를 배포합니다.

|

**URL과 리다이렉트 URL을 분석**

예제:

.. code-block:: html

    http://www.foo.com/err.jsp?User=baduser&Error=0 
    http://www.foo.com/err.jsp?User=gooduser&Error=2 

As is seen above, when a tester provides a user ID and password to the web application, they see a message indication that an error has occurred in the URL. In the first case they have provided a bad user ID and bad password. In the second, a good user ID and a bad password, so they can identify a valid user ID. 

**URI Probing**

Sometimes a web server responds differently if it receives a request for an existing directory or not. For instance in some portals every user is associated with a directory. If testers try to access an existing directory they could receive a web server error. 

A very common error that is received from web server is: 

.. code-block:: html

    403 Forbidden error code 

and 

.. code-block:: html

    404 Not found error code 

예제

.. code-block:: html

    http://www.foo.com/account1 - we receive from web server:
    403 Forbidden 
    http://www.foo.com/account2 - we receive from web server: 
    404 file Not Found 

In the first case the user exists, but the tester cannot view the web page, in second case instead the user "account2" does not exist. By collecting this information testers can enumerate the users. 

**웹 페이지 타이틀 분석**

Testers can receive useful information on Title of web page, where they can obtain a specific error code or messages that reveal if the problems are with the username or password. 

For instance, if a user cannot authenticate to an application and receives a web page whose title is similar to: 

.. code-block:: html

    Invalid user 
    Invalid authentication 

**복원 장비로 부터 수신된 메시지 분석**

When we use a recovery facility (i.e. a forgotten password function) a vulnerable application might return a message that reveals if a username exists or not. 

For example, message similar to the following: 

.. code-block:: html

    Invalid username: e-mail address is not valid or the specified user was not found. 

.. code-block:: html

    Valid username: Your password has been successfully sent to the email address you registered with. 

**Friendly 404 Error Message**

When we request a user within the directory that does not exist, we don't always receive 404 error code. Instead, we may receive "200 ok" with an image, in this case we can assume that when we receive the specific image the user does not exist. This logic can be applied to other web server response; the trick is a good analysis of web server and web application messages. 

|

사용자 추측
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In some cases the user IDs are created with specific policies of administrator or company. For example we can view a user with a user ID created in sequential order: CN000100 CN000101 ¡¦. Sometimes the usernames are created with a REALM alias and then a sequential numbers: R1001 . user 001 for REALM1 R2001 . user 001 for REALM2 
In the above sample we can create simple shell scripts that compose user IDs and submit a request with tool like wget to automate a web query to discern valid user IDs. To create a script we can also use Perl and CURL. 
Other possibilities are: - user IDs associated with credit card numbers, or in general numbers with a pattern. - user IDs associated with real names, e.g. if Freddie Mercury has a user ID of "fmercury", then you might guess Roger Taylor to have the user ID of "rtaylor". 
Again, we can guess a username from the information received from an LDAP query or from Google information gathering, for example, from a specific domain. Google can help to find domain users through specific queries or through a simple shell script or tool. 
Attention: by enumerating user accounts, you risk locking out accounts after a predefined number of failed probes (based on application policy). Also, sometimes, your IP address can be banned by dynamic rules on the application firewall or Intrusion Prevention System. 


Gray Box testing 
-----------------------------------------------------------------------------------------

**인증 에러 메시지 테스트**

Verify that the application answers in the same manner for every client request that produces a failed authentication. For this issue the Black Box testing and Gray Box testing have the same concept based on the analysis of messages or error codes received from web application. 

**결과**

The application should answer in the same manner for every failed attempt of authentication. 

예제: 

Credentials submitted are not valid 

|

Tools 
============================================================================================

- WebScarab: OWASP_WebScarab_Project 
- CURL: http://curl.haxx.se/ 
- PERL: http://www.perl.org 
- Sun Java Access & Identity Manager users enumeration tool: http://www.aboutsecurity.net 

|

References 
============================================================================================

- Marco Mella, Sun Java Access & Identity Manager Users enumeration: http://www.aboutsecurity.net 
- Username Enumeration Vulnerabilities: http://www.gnucitizen.org/blog/username-enumeration-vulnerabilities 

|

Remediation 
============================================================================================

Ensure the application returns consistent generic error messages in response to invalid account name, password or other user credentials entered during the log in process. 

Ensure default system accounts and test accounts are deleted prior to releasing the system into production (or exposing it to an untrusted network). 