============================================================================================
OTG-SESS-001 (세션 관리 스키마 우회 테스트)
============================================================================================

|

개요
============================================================================================

각각의 웹 페이지 또는 서비스에 계속적인 인증을 피하기 위해,
웹 어플리케이션은 정의된 시간 범위 동안 자격 증명을 저장하고 유효하게 하기 위해
다양한 메카니즘을 구현합니다.

These mechanisms are known as Session Management and while they are important in order to increase the ease of use and user-friendliness of the application, they can be exploited by a penetration tester to gain access to a user account, without the need to provide correct credentials.
In this test, the tester wants to check that cookies and other session tokens are created in a secure and unpredictable way. 
An attacker who is able to predict and forge a weak cookie can easily hijack the sessions of legitimate users.
Cookies are used to implement session management and are described
in detail in RFC 2965. 
In a nutshell, when a user accesses an
application which needs to keep track of the actions and identity of
that user across multiple requests, a cookie (or cookies) is generated
by the server and sent to the client. The client will then send the
cookie back to the server in all following connections until the cookie
expires or is destroyed. The data stored in the cookie can provide
to the server a large spectrum of information about who the user is,
what actions he has performed so far, what his preferences are, etc.
therefore providing a state to a stateless protocol like HTTP.
A typical example is provided by an online shopping cart. Throughout
the session of a user, the application must keep track of his identity,
his profile, the products that he has chosen to buy, the quantity, the
individual prices, the discounts, etc. Cookies are an efficient way to
store and pass this information back and forth (other methods are
URL parameters and hidden fields).
Due to the importance of the data that they store, cookies are there
fore vital in the overall security of the application. Being able to tamper
with cookies may result in hijacking the sessions of legitimate
users, gaining higher privileges in an active session, and in general
influencing the operations of the application in an unauthorized way.
In this test the tester has to check whether the cookies issued to clients
can resist a wide range of attacks aimed to interfere with the
sessions of legitimate users and with the application itself. The overall
goal is to be able to forge a cookie that will be considered valid
by the application and that will provide some kind of unauthorized
access (session hijacking, privilege escalation, ...).
Usually the main steps of the attack pattern are the following:

- cookie collection: collection of a sufficient number of cookie samples;
- cookie reverse engineering: analysis of the cookie generation algorithm;
- cookie manipulation: forging of a valid cookie in order to perform the attack. 

This last step might require a large number of attempts,
depending on how the cookie is created (cookie brute-force attack).
Another pattern of attack consists of overflowing a cookie. Strictly
speaking, this attack has a different nature, since here testers are not
trying to recreate a perfectly valid cookie. Instead, the goal is to overflow
a memory area, thereby interfering with the correct behavior of
the application and possibly injecting (and remotely executing) malicious
code.

|

테스트 방법
============================================================================================

Black Box Testing and Examples
------------------------------------------------------------------------------------------

All interaction between the client and application should be tested at
least against the following criteria:

- Are all Set-Cookie directives tagged as Secure?
- Do any Cookie operations take place over unencrypted transport?
- Can the Cookie be forced over unencrypted transport?
- If so, how does the application maintain security?
- Are any Cookies persistent?
- What Expires= times are used on persistent cookies, and are they reasonable?
- Are cookies that are expected to be transient configured as such?
- What HTTP/1.1 Cache-Control settings are used to protect Cookies?
- What HTTP/1.0 Cache-Control settings are used to protect Cookies?

|

쿠키 수집
------------------------------------------------------------------------------------------

쿠키 조작을 해야하는 첫번째 단계는 어플리케이션이 쿠키를 생성하고 관리하는 방법을 이해해야 합니다.
이 작업을 위해, 테스터는 다음 질문에 대답해야 합니다.

- 어플리케이션에서 얼마나 많은 쿠키가 사용됩니까?
어플리케이션 서핑. 쿠키가 생성될 때 기록. 
수신한 쿠키, 쿠키들이 설정된 페이지, 쿠키가 유효한 도메인, 쿠키값, 쿠키 특징 리스트 작성
- 어플리케이션의 어떤 부분이 쿠키가 생성되는지 수정되는지?
어플리케이션 서핑, 쿠키가 계속 남아있는 것과 수정되는 것을 찾기.
- 어플리케이션의 어떤 부분이 접근 및 활용하기 위해 쿠키로 필요한지?
어플리케이션의 어떤 부분이 쿠키가 필요한지 찾습니다.
페이지에 접근하고, 쿠키없이 다시 접근해보거나 그 값을 수정하여 접근해봅니다.
여기서 사용되는 쿠키를 매핑해야 합니다.
해당 어플리케이션 부분과 연관 정보에 각 쿠키를 매핑하는 스프레드 시트는 과정의 중요한 출력을 할 수 있습니다.

|

세션 분석
------------------------------------------------------------------------------------------

The session tokens (Cookie, SessionID or Hidden Field) themselves should be examined to ensure their quality from a security perspective.
They should be tested against criteria such as their randomness, uniqueness, resistance to statistical and cryptographic analysis and information leakage.

- Token Structure & Information Leakage

The first stage is to examine the structure and content of a Session ID
provided by the application. A common mistake is to include specific
data in the Token instead of issuing a generic value and referencing
real data at the server side.

If the Session ID is clear-text, the structure and pertinent data may be
immediately obvious as the following:

.. code-block:: html

    http://foo.bar/showImage?img=img00011

If part or the entire token appears to be encoded or hashed, it should
be compared to various techniques to check for obvious obfuscation.
For example the string "192.168.100.1:owaspuser:password:15:58"
is represented in Hex, Base64 and as an MD5 hash:

.. code-block:: html

    Hex 3139322E3136382E3130302E313A6F77617370757
    365723A70617373776F72643A31353A3538
    Base64 MTkyLjE2OC4xMDAuMTpvd2FzcHVzZXI6c
    GFzc3dvcmQ6MTU6NTg=
    MD5 01c2fc4f0a817afd8366689bd29dd40a

Having identified the type of obfuscation, it may be possible to decode back to the original data. 
In most cases, however, this is unlikely. 
Even so, it may be useful to enumerate the encoding in place from the format of the message. 
Furthermore, if both the format and obfuscation technique can be deduced, automated brute-force attacks could be devised.
Hybrid tokens may include information such as IP address or User ID together with an encoded portion, as the following:

.. code-block:: html

    owaspuser:192.168.100.1:
    a7656fafe94dae72b1e1487670148412

Having analyzed a single session token, the representative sample should be examined. 
A simple analysis of the tokens should immediately reveal any obvious patterns. 
For example, a 32 bit token may include 16 bits of static data and 16 bits of variable data. 
This may indicate that the first 16 bits represent a fixed attribute of the user 

– e.g. the username or IP address. If the second 

16 bit chunk is incrementing at a regular rate, it may indicate a
sequential or even time-based element to the token generation.
See examples.
If static elements to the Tokens are identified, further samples
should be gathered, varying one potential input element at a time.
For example, log in attempts through a different user account or
from a different IP address may yield a variance in the previously
static portion of the session token.
The following areas should be addressed during the single and
multiple Session ID structure testing:

- What parts of the Session ID are static?
- What clear-text confidential information is stored in the Session
D? E.g. usernames/UID, IP addresses
- What easily decoded confidential information is stored?
- What information can be deduced from the structure of the
Session ID?
- What portions of the Session ID are static for the same log in
conditions?
- What obvious patterns are present in the Session ID as a whole,
or individual portions?

|

Session ID 예측 가능성과 랜덤성
------------------------------------------------------------------------------------------

Analysis of the variable areas (if any) of the Session ID should be
undertaken to establish the existence of any recognizable or predictable
patterns. These analyses may be performed manually and
with bespoke or OTS statistical or cryptanalytic tools to deduce
any patterns in the Session ID content. Manual checks should include
comparisons of Session IDs issued for the same login conditions
– e.g., the same username, password, and IP address.
Time is an important factor which must also be controlled. High
numbers of simultaneous connections should be made in order to
gather samples in the same time window and keep that variable
constant. Even a quantization of 50ms or less may be too coarse
and a sample taken in this way may reveal time-based components
that would otherwise be missed.
Variable elements should be analyzed over time to determine
whether they are incremental in nature. Where they are incremental,
patterns relating to absolute or elapsed time should be investigated.
Many systems use time as a seed for their pseudo-random
elements. Where the patterns are seemingly random, one-way
hashes of time or other environmental variations should be considered
as a possibility. Typically, the result of a cryptographic
hash is a decimal or hexadecimal number so should be identifiable.
In analyzing Session ID sequences, patterns or cycles, static elements
and client dependencies should all be considered as possible
contributing elements to the structure and function of the
application.

- Are the Session IDs provably random in nature? Can the resulting
values be reproduced?
- Do the same input conditions produce the same ID on a
subsequent run?
- Are the Session IDs provably resistant to statistical or
cryptanalysis?
- What elements of the Session IDs are time-linked?
- What portions of the Session IDs are predictable?
- Can the next ID be deduced, given full knowledge of the
generation algorithm and previous IDs?

|

Cookie reverse engineering
------------------------------------------------------------------------------------------

Now that the tester has enumerated the cookies and has a general
idea of their use, it is time to have a deeper look at cookies
that seem interesting. Which cookies is the tester interested in?
A cookie, in order to provide a secure method of session management,
must combine several characteristics, each of which is
aimed at protecting the cookie from a different class of attacks.
These characteristics are summarized below:
[1] Unpredictability: a cookie must contain some amount of hardto-guess
data. The harder it is to forge a valid cookie, the harder is
to break into legitimate user’s session. If an attacker can guess the
cookie used in an active session of a legitimate user, they will be
able to fully impersonate that user (session hijacking). In order to
make a cookie unpredictable, random values and/or cryptography
can be used.
[2] Tamper resistance: a cookie must resist malicious attempts
of modification. If the tester receives a cookie like IsAdmin=No,
it is trivial to modify it to get administrative rights, unless the application
performs a double check (for instance, appending to the
cookie an encrypted hash of its value)
[3] Expiration: a critical cookie must be valid only for an appropriate
period of time and must be deleted from the disk or memory
afterwards to avoid the risk of being replayed. This does not apply
to cookies that store non-critical data that needs to be remembered
across sessions (e.g., site look-and-feel).
[4] "Secure" flag: a cookie whose value is critical for the integrity
of the session should have this flag enabled in order to allow its
transmission only in an encrypted channel to deter eavesdropping.
The approach here is to collect a sufficient number of instances
of a cookie and start looking for patterns in their value. The exact
meaning of "sufficient" can vary from a handful of samples,
if the cookie generation method is very easy to break, to several
thousands, if the tester needs to proceed with some mathematical
analysis (e.g., chi-squares, attractors. See later for more information).
It is important to pay particular attention to the workflow of the
application, as the state of a session can have a heavy impact on
collected cookies. A cookie collected before being authenticated
can be very different from a cookie obtained after the authentication.
Another aspect to keep into consideration is time. Always record
the exact time when a cookie has been obtained, when there is
the possibility that time plays a role in the value of the cookie (the
server could use a time stamp as part of the cookie value). The
time recorded could be the local time or the server’s time stamp
included in the HTTP response (or both).
When analyzing the collected values, the tester should try to figure
out all variables that could have influenced the cookie value and
try to vary them one at the time. Passing to the server modified
versions of the same cookie can be very helpful in understanding
how the application reads and processes the cookie.

Examples of checks to be performed at this stage include:
• What character set is used in the cookie? Has the cookie a
numeric value? alphanumeric? hexadecimal? What happens if
the tester inserts in a cookie characters that do not belong to the
expected charset?
• Is the cookie composed of different sub-parts carrying different
pieces of information? How are the different parts separated?
With which delimiters? Some parts of the cookie could have a
higher variance, others might be constant, others could assume
only a limited set of values. Breaking down the cookie to its base
components is the first and fundamental step.
An example of an easy-to-spot structured cookie is the following:

.. code-block:: html

    ID=5a0acfc7ffeb919:CR=1:TM=1120514521:LM=11205145
    21:S=j3am5KzC4v01ba3q

This example shows 5 different fields, carrying different types of data:

.. code-block:: html

    ID – hexadecimal
    CR – small integer
    TM and LM – large integer. (And curiously they hold the
    same value. Worth to see what happens modifying one of
    them)
    S – alphanumeric

Even when no delimiters are used, having enough samples can help.
As an example, let’s look at the following series:

.. code-block:: html

    0123456789abcdef

|

Brute Force Attacks
------------------------------------------------------------------------------------------

Brute force attacks inevitably lead on from questions relating to
predictability and randomness. The variance within the Session
IDs must be considered together with application session duration
and timeouts. If the variation within the Session IDs is relatively
small, and Session ID validity is long, the likelihood of a successful
brute-force attack is much higher.
A long Session ID (or rather one with a great deal of variance) and
a shorter validity period would make it far harder to succeed in a
brute force attack.

- How long would a brute-force attack on all possible Session IDs
take?
- Is the Session ID space large enough to prevent brute forcing? For
example, is the length of the key sufficient when compared to the
valid life-span?
- Do delays between connection attempts with different Session IDs
mitigate the risk of this attack?

|

Gray Box testing and example
------------------------------------------------------------------------------------------

만약 테스터가 세션 관리 스키마 구현에 접근된다면, 다음을 확인할 수 있습니다.

- Random Session Token

The Session ID or Cookie issued to the client should not be easily pre dictable 
(don’t use linear algorithms based on predictable variables such as the client IP address). 
The use of cryptographic algorithms with key length of 256 bits is encouraged (like AES).

- Token length

Session ID will be at least 50 characters length.

- Session Time-out

Session token should have a defined time-out (it depends on the criticality
of the application managed data)

- Cookie configuration:
- non-persistent: only RAM memory
- secure (set only on HTTPS channel):

Set Cookie: cookie=data; path=/; domain=.aaa.it; secure

- HTTPOnly (not readable by a script):
Set Cookie: cookie=data; path=/; domain=.aaa.it; HTTPOnly
More information here: Testing for cookies attributes

|

Tools
============================================================================================

- OWASP Zed Attack Proxy Project (ZAP)
- Burp Sequencer: http://www.portswigger.net/suite/sequencer.html
- Foundstone CookieDigger: http://www.mcafee.com/us/downloads/free-tools/cookiedigger.aspx
- YEHG’s JHijack: https://www.owasp.org/index.php/JHijack

|

References
============================================================================================

Whitepapers
----------------------------------------------------------------------------------------

- RFC 2965 "HTTP State Management Mechanism"
- RFC 1750 "Randomness Recommendations for Security"
- Michal Zalewski: "Strange Attractors and TCP/IP Sequence Number Analysis" (2001): http://lcamtuf.coredump.cx/oldtcp/tcpseq.html
- Michal Zalewski: "Strange Attractors and TCP/IP Sequence Number Analysis - One Year Later" (2002): http://lcamtuf.coredump.cx/newtcp/
- Correlation Coefficient: http://mathworld.wolfram.com/CorrelationCoefficient.html
- Darrin Barrall: "Automated Cookie Analysis": http://www.spidynamics.com/assets/documents/SPIcookies.pdf
- ENT: http://fourmilab.ch/random/
- http://seclists.org/lists/fulldisclosure/2005/Jun/0188.html
- Gunter Ollmann: "Web Based Session Management": http://www.technicalinfo.net
- Matteo Meucci:"MMS Spoofing": http://www.owasp.org/images/7/72/MMS_Spoofing.ppt

|

Videos
----------------------------------------------------------------------------------------

- Session Hijacking in Webgoat Lesson: http://yehg.net/lab/pr0js/training/view/owasp/webgoat/WebGoat_SessionMan_SessionHijackingWithJHijack/

|

Related Security Activities
----------------------------------------------------------------------------------------

Description of Session Management Vulnerabilities

See the OWASP articles on Session Management Vulnerabilities.

Description of Session Management Countermeasures
See the OWASP articles on Session Management Countermeasures.

How to Avoid Session Management Vulnerabilities
See the OWASP Development Guide article on how to Avoid Session
Management Vulnerabilities.

How to Review Code for Session Management| Vulnerabilities
See the OWASP Code Review Guide article on how to Review Code
for Session Management Vulnerabilities.

|