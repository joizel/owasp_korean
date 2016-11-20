============================================================================================
OTG-SESS-008 (세션 퍼즐링 테스트)
============================================================================================

|

개요
============================================================================================

세션 변수 과부하(세션 퍼즐링)는 공격자가 다양한 악성 행위를 실행할 수 있는 어플리케이션 레벨 취약점이 있습니다.

- 효율적인 인증 강화 메카니즘을 우회하여 합법적인 사용자 행세
- 안전하다고 간주되는 환경에서 악의적인 사용자 계정의 권한 상승

- Skip over qualifying phases in multi-phase processes, even if the process includes all the commonly recommended code level restrictions.
- Manipulate server-side values in indirect methods that cannot be predicted or detected.
- Execute traditional attacks in locations that were previously unreachable, or even considered secure.

해당 취약점은 어플리케이션이 하나 이상의 목적을 위해 동일한 세션 변수를 사용할 경우 발생합니다.

An attacker can potentially access pages in an order unanticipated by the developers so that the session variable is set in one context and then used in another.

For example, an attacker could use session variable overloading to bypass authentication enforcement mechanisms of applications that enforce authentication by validating the existence of session variables that contain identity–related values, which are usually stored in the session after a successful authentication process.
This means an attacker first accesses a location in the application that sets session context and then accesses privileged locations that examine this context.
For example - an authentication bypass attack vector could be executed by accessing a publicly accessible entry point (e.g. a password recovery page) that populates the session with an identical session variable, based on fixed values or on user originating input.

|

테스트 방법
============================================================================================

Black Box Testing
----------------------------------------------------------------------------------------

This vulnerability can be detected and exploited by enumerating all of the session variables used by the application and in which context they are valid. 
In particular this is possible by accessing a sequence of entry points and then examining exit points.
In case of black box testing this procedure is difficult and requires some luck since every different sequence could lead to a different result.

**예제**


A very simple example could be the password reset functionality that, in the entry point, could request the user to provide some identifying information such as the username or the e-mail address.
This page might then populate the session with these identifying values, which are received directly from the client side, or obtained from queries or calculations based on the received input.
At this point there may be some pages in the application that show private data based on this session object. 

이러한 방식으로 공격자는 인증 과정을 우회할 수 있습니다.

|

Gray Box testing
----------------------------------------------------------------------------------------

해당 취약점을 탐지하기 위한 가장 효율적인 방법은 소스 코드를 검토하는 것입니다.

|

References
============================================================================================

Whitepapers
----------------------------------------------------------------------------------------

- Session Puzzles: http://puzzlemall.googlecode.com/files/Session%20Puzzles%20-%20Indirect%20Application%20Attack%20Vectors%20-%20May%202011%20-%20Whitepaper.pdf

|

Remediation
============================================================================================

세션 변수는 하나의 일관된 목적으로만 사용되어야 합니다.

|
