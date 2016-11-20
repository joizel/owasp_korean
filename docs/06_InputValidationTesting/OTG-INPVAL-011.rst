============================================================================================
OTG-INPVAL-011 (IMAP/SMTP 인젝션 침투 테스트)
============================================================================================

|

개요
============================================================================================

This threat affects all applications that communicate with mail
servers (IMAP/SMTP), generally webmail applications. The aim of
this test is to verify the capacity to inject arbitrary IMAP/SMTP
commands into the mail servers, due to input data not being properly
sanitized.
The IMAP/SMTP Injection technique is more effective if the mail
server is not directly accessible from Internet. Where full communication
with the backend mail server is possible, it is recommended
to conduct direct testing.
An IMAP/SMTP Injection makes it possible to access a mail server
which otherwise would not be directly accessible from the Internet.
In some cases, these internal systems do not have the same
level of infrastructure security and hardening that is applied to the 
front-end web servers.Therefore, mail server results may be more
vulnerable to attacks by end users (see the scheme presented in
Figure 1).


Figure 1 depicts the flow of traffic generally seen when using
webmail technologies. Step 1 and 2 is the user interacting with
the webmail client, whereas step 2 is the tester bypassing the
webmail client and interacting with the back-end mail servers
directly.
This technique allows a wide variety of actions and attacks. The
possibilities depend on the type and scope of injection and the
mail server technology being tested.
Some examples of attacks using the IMAP/SMTP Injection technique
are:

- Exploitation of vulnerabilities in the IMAP/SMTP protocol
- Application restrictions evasion
- Anti-automation process evasion
- Information leaks
- Relay/SPAM

|

테스트 방법
============================================================================================

기본 공격 패턴

- 취약한 파라미터 확인
- 데이터 흐름 및 클라이언트 배포 구조 이해
- IMAP/SMTP 명령 인젝션

|

취약한 파라미터 확인
-----------------------------------------------------------------------------------------

취약한 파라미터를 탐지하기 위해서, 테스터는 입력시 어플리케이션이 
어떻게 처리되는 지 확인합니다.
입력 검증 테스트는 서버에 위조, 또는 악의적인 요청을 보냈을 때 
응답 내용을 분석합니다.

In a secure application, the response should be an error 
with some corresponding action telling the client that something has gone wrong. 

취약한 어플리케이션에서, 악의적인 요청은 "HTTP 200 ok" 응답 메시지로 
답한 백엔드 어플리케이션에서 처리될 것입니다.


It is important to note that the requests being sent should match
the technology being tested. 

Sending SQL injection strings for Microsoft SQL server when a MySQL 
server is being used will result in false positive responses. 

In this case, sending malicious IMAP commands is modus operandi since 
IMAP is the underlying protocol being tested.

사용할 수 있는 IMAP 특별한 파라미터

.. csv-table::

    "On the IMAP server", "On the SMTP server" 
    "Authentication", "Emissor e-mail"
    "operations with mail boxes", "Destination e-mail"
    "(list, read, create, delete,rename)", ""
    "operations with messages", "Subject"
    "(read, copy, move, delete)",""
    "Disconnection", "Message body"
    "","Attached files"


이번 예제에서, "mailbox" 파라미터는 모든 요청을 조작하는 데 사용됩니다.

.. code-block:: html

    http://<webmail>/src/read_body.php?mailbox=INBOX&
    passed_id=46106&startMessage=1

다음 예제를 사용할 수 있습니다.

- 파라미터에 null 값 할당

.. code-block:: html

    http://<webmail>/src/read_body.php?mailbox=&
    passed_id=46106&startMessage=1

- 랜덤한 다른 값으로 값 대체

.. code-block:: html

    http://<webmail>/src/read_body.php?mailbox=NOTEXIST&
    passed_id=46106&startMessage=1


- 파라미터에 다른 값 추가

.. code-block:: html

    http://<webmail>/src/read_body.php?mailbox=INBOX 
    PARAMETER2&passed_id=46106&startMessage=1

- 특수 문자 추가

.. code-block:: html

    http://<webmail>/src/read_body.php?mailbox=INBOX"& 
    passed_id=46106&startMessage=1

- 파라미터 제거

.. code-block:: html

    http://<webmail>/src/read_body.php?passed_id=46106&startMessage=1

위 테스트 쵣종 결과는 테스터에게 세가지 상황을 줍니다.

S1 - 어플리케이션은 에러 코드 및 메세지를 리턴
S2 - 어플리케이션은 에러 코드 및 메시지를 리턴하지 않지만, 요청한 조작을 실행하지 않습니다.
S3 - 어플리케이션은 에러 코드 및 메시지를 리턴하지 않고, 요청한 조작을 실행합니다.

S1과 S2 상황은 성공적인 IMAP/SMTP 인젝션을 의미합니다.

An attacker’s aim is receiving the S1 response, as it is an indicator
that the application is vulnerable to injection and further
manipulation.

Let’s suppose that a user retrieves the email headers using the
following HTTP request:

.. code-block:: html

    http://<webmail>/src/view_header.php?mailbox=INBOX&-
    passed_id=46105&passed_ent_id=0

An attacker might modify the value of the parameter INBOX by
injecting the character " (%22 using URL encoding):

.. code-block:: html

    http://<webmail>/src/view_header.php?mailbox=INBOX-
    %22&passed_id=46105&passed_ent_id=0

In this case, the application answer may be:

.. code-block:: html

    ERROR: Bad or malformed request.
    Query: SELECT "INBOX""
    Server responded: Unexpected extra arguments to Select

The situation S2 is harder to test successfully. The tester needs
to use blind command injection in order to determine if the server
is vulnerable.
On the other hand, the last situation (S3) is not revelant in this
paragraph.

**예상 결과**

- 취약한 파라미터 리스트
- 영향 받는 기능
- 인젝션 가능 형식 (IMAP/SMTP)

|

데이터 흐름 및 클라이언트 배포 구조 이해
-----------------------------------------------------------------------------------------

모든 취약한 파라미터를 확인한 후에, 테스터는 가능한 인젝션 부분을 정의하고 
어플리케이션을 공격하기 위한 테스트 계획을 설계해야 합니다.

이번 테스트에서는 어플리케이션의 "passed_id" 파라미터가 취약하고 
다음 요청을 사용해야 합니다.

.. code-block:: html

    http://<webmail>/src/read_body.php?mailbox=INBOX&
    passed_id=46225&startMessage=1

다음 테스트 진행

.. code-block:: html

    http://<webmail>/src/read_body.php?mailbox=INBOX&
    passed_id=test&startMessage=1

다음 에러 메시지 생성

.. code-block:: html
    
    ERROR : Bad or malformed request.
    Query: FETCH test:test BODY[HEADER]
    Server responded: Error in IMAP command received by
    server

이번 예제에서는 에러 메시지에 실행 명령 이름과 관련 파라미터를 리턴합니다.

In other situations, the error message ("not controlled" by the
application) contains the name of the executed command, but
reading the suitable RFC (see "Reference" paragraph) allows the
tester to understand what other possible commands can be executed.

If the application does not return descriptive error messages, the
tester needs to analyze the affected functionality to deduce all
the possible commands (and parameters) associated with the
above mentioned functionality.

For example, if a vulnerable parameter has been detected in the
create mailbox functionality, it is logical to assume that the affected
IMAP command is "CREATE".

According to the RFC, the
CREATE command accepts one parameter which specifies the
name of the mailbox to create.

**예상 결과**

- IMAP/SMTP 명령 영향 목록
- Type, value, and number of parameters expected by the affected IMAP/SMTP commands

|

IMAP/SMTP 명령 인젝션
-----------------------------------------------------------------------------------------

Once the tester has identified vulnerable parameters and has
analyzed the context in which they are executed, the next stage
is exploiting the functionality.

This stage has two possible outcomes:

1. The injection is possible in an unauthenticated state:
the affected functionality does not require the user to be
authenticated. The injected (IMAP) commands available are
limited to: CAPABILITY, NOOP, AUTHENTICATE, LOGIN, and
LOGOUT.

2. The injection is only possible in an authenticated state:
the successful exploitation requires the user to be fully
authenticated before testing can continue.
In any case, the typical structure of an IMAP/SMTP Injection is
as follows:

- Header: ending of the expected command;
- Body: injection of the new command;
- Footer: beginning of the expected command.

It is important to remember that, in order to execute an IMAP/
SMTP command, the previous command must be terminated
with the CRLF (%0d%0a) sequence.

Let’s suppose that in the stage 1 ("Identifying vulnerable parameters"),
the attacker detects that the parameter "message_id" in
the following request is vulnerable:

.. code-block:: html

    http://<webmail>/read_email.php?message_id=4791

Let’s suppose also that the outcome of the analysis performed
in the stage 2 ("Understanding the data flow and deployment
structure of the client") has identified the command and arguments
associated with this parameter as:

.. code-block:: html

    FETCH 4791 BODY[HEADER]


In this scenario, the IMAP injection structure would be:

.. code-block:: html

    http://<webmail>/read_email.php?message_id=4791
    BODY[HEADER]%0d%0aV100 CAPABILITY%0d%0aV101
    FETCH 4791

Which would generate the following commands:

.. code-block:: html

    ???? FETCH 4791 BODY[HEADER]
    V100 CAPABILITY
    V101 FETCH 4791 BODY[HEADER]

where:

.. code-block:: html

    Header = 4791 BODY[HEADER]
    Body = %0d%0aV100 CAPABILITY%0d%0a
    Footer = V101 FETCH 4791 


**예상 결과**

- 임의의 IMAP/SMTP 명령 인젝션

|

References
============================================================================================

Whitepapers
-----------------------------------------------------------------------------------------

- RFC 0821 "Simple Mail Transfer Protocol".
- RFC 3501 "Internet Message Access Protocol - Version 4rev1".
- Vicente Aguilera Díaz: "MX Injection: Capturing and Exploiting Hidden Mail Servers" - http://www.webappsec.org/projects/articles/121106.pdf

|