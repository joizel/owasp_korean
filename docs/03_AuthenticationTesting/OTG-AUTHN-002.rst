==========================================================================================
OTG-AUTHN-002 (기본 자격 증명 테스트)
==========================================================================================

|


개요
==========================================================================================

오늘날 웹 어플리케이션은 서버 관리자에 의해 최소한의 구성/사용자 정의로 서버에 설치할 수 있는 유명 오픈 소스 나 상용 소프트웨어를 사용합니다.
또한, 수 많은 하드웨어 제품(네트워크 라우터/데이터베이스 서버)이 웹 기반 설정 및 관리 인터페이스를 제공합니다.
한번 설치된 응용 프로그램은 대부분 제대로 구성하지 않고, 초기 인증 및 설정에 제공되는 기본 자격 증명을 변경하지 않습니다.
또한, 어플리케이션에서 새로운 계정을 생성할 때 기본 패스워드가 생성됩니다.
만약 패스워드가 예측 가능하고 사용자가 첫번째 접속시 바꾸지 않을 경우, 공격자는 어플리케이션의 비인가 접근을 할 수 있습니다.

해당 문제의 근본 원인을 다음과 같이 식별할 수 있습니다.

- Inexperienced IT personnel, who are unaware of the importance of changing default passwords on installed infrastructure components, or leave the password as default for "ease of maintenance".
- Programmers who leave back doors to easily access and test their application and later forget to remove them.
- Applications with built-in non-removable default accounts with a preset username and password.
- Applications that do not force the user to change the default credentials after the first log in.

|

테스트 방법
==========================================================================================

|

**일반 어플리케이션의 기본 자격 증명 테스트**

In black box testing the tester knows nothing about the application and its underlying infrastructure. 
In reality this is often not true, and some information about the application is known. 
We suppose that you have identified, through the use of the techniques described in this Testing Guide under the chapter Information Gathering, at least one or more common applications that may contain accessible administrative interfaces.
When you have identified an application interface, for example
a Cisco router web interface or a Weblogic administrator portal,
check that the known usernames and passwords for these devices
do not result in successful authentication. 
To do this you can consult the manufacturer`s documentation or, in a much simpler way, you can find common credentials using a search engine or by using one of the sites or tools listed in the Reference section.
When facing applications where we do not have a list of default
and common user accounts (for example due to the fact that the
application is not wide spread) we can attempt to guess valid default
credentials. Note that the application being tested may have
an account lockout policy enabled, and multiple password guess
attempts with a known username may cause the account to be
locked. If it is possible to lock the administrator account, it may be
troublesome for the system administrator to reset it.
Many applications have verbose error messages that inform the
site users as to the validity of entered usernames. This information
will be helpful when testing for default or guessable user accounts.
Such functionality can be found, for example, on the log
in page, password reset and forgotten password page, and sign
up page. Once you have found a default username you could also
start guessing passwords for this account.
More information about this procedure can be found in the section
Testing for User Enumeration and Guessable User Account and in
the section Testing for Weak password policy.
Since these types of default credentials are often bound to administrative
accounts you can proceed in this manner:

• Try the following usernames - "admin", "administrator", "root",
"system", "guest", "operator", or "super".
These are popular among system administrators and are often
used. Additionally you could try "qa", "test", "test1", "testing" and
similar names. Attempt any combination of the above in both the
username and the password fields. If the application is vulnerable
to username enumeration, and you manage to successfully
identify any of the above usernames, attempt passwords in a
similar manner. In addition try an empty password or one of
the following "password", "pass123", "password123", "admin",
or "guest" with the above accounts or any other enumerated
accounts.
Further permutations of the above can also be attempted. If
these passwords fail, it may be worth using a common username
and password list and attempting multiple requests against the
application. This can, of course, be scripted to save time.
• Application administrative users are often named after the
application or organization.
This means if you are testing an application named "Obscurity",
try using obscurity/obscurity or any other similar combination as
the username and password.
• When performing a test for a customer, attempt using names
of contacts you have received as usernames with any common
passwords. Customer email addresses mail reveal the user
accounts naming convention: if employee John Doe has the email
address jdoe@example.com, you can try to find the names of
system administrators on social media and guess their username
by applying the same naming convention to their name.
• Attempt using all the above usernames with blank passwords.
• Review the page source and JavaScript either through a proxy
or by viewing the source. Look for any references to users and
passwords in the source.
For example "If username='admin' then starturl=/admin.asp else /index.asp" (for a successful log in versus a failed log in).
Also, if you have a valid account, then log in and view every
request and response for a valid log in versus an invalid log in,
such as additional hidden parameters, interesting GET request
(login=yes), etc.
• Look for account names and passwords written in comments
in the source code. Also look in backup directories for source
code (or backups of source code) that may contain interesting
comments and code.

**새로운 계정 기존 패스워드 테스트**

It can also occur that when a new account is created in an application
the account is assigned a default password. This password
could have some standard characteristics making it predictable. If
the user does not change it on first usage (this often happens if
the user is not forced to change it) or if the user has not yet logged
on to the application, this can lead an attacker to gain unauthorized
access to the application.
The advice given before about a possible lockout policy and verbose
error messages are also applicable here when testing for
default passwords.
The following steps can be applied to test for these types of default
credentials:
• Looking at the User Registration page may help to determine the
expected format and minimum or maximum length of the 
application usernames and passwords. If a user registration page
does not exist, determine if the organization uses a standard
naming convention for user names such as their email address or
the name before the "@" in the email.
• Try to extrapolate from the application how usernames are
generated.
For example, can a user choose his/her own username or does
the system generate an account name for the user based on
some personal information or by using a predictable sequence? If
the application does generate the account names in a predictable
sequence, such as user7811, try fuzzing all possible accounts
recursively.
If you can identify a different response from the application when
using a valid username and a wrong password, then you can try a
brute force attack on the valid username (or quickly try any of the
identified common passwords above or in the reference section).
• Try to determine if the system generated password is predictable.
To do this, create many new accounts quickly after one another
so that you can compare and determine if the passwords
are predictable. If predictable, try to correlate these with the
usernames, or any enumerated accounts, and use them as a
basis for a brute force attack.
• If you have identified the correct naming convention for the user
name, try to "brute force" passwords with some common
predictable sequence like for example dates of birth.
• Attempt using all the above usernames with blank passwords or
using the username also as password value.

|

Gray Box testing
---------------------------------------------------------------------------------------

The following steps rely on an entirely Gray Box approach. If only
some of this information is available to you, refer to black box
testing to fill the gaps.
• Talk to the IT personnel to determine which passwords they
use for administrative access and how administration of the
application is undertaken.
• Ask IT personnel if default passwords are changed and if default
user accounts are disabled.
• Examine the user database for default credentials as described
in the Black Box testing section. Also check for empty password
fields.
• Examine the code for hard coded usernames and passwords.
• Check for configuration files that contain usernames
and passwords.
• Examine the password policy and, if the application generates its
own passwords for new users, check the policy in use for this
procedure.

|

Tools
========================================================================================

- Burp Intruder: http://portswigger.net/burp/intruder.html
- THC Hydra: http://www.thc.org/thc-hydra/
- Brutus: http://www.hoobie.net/brutus/
- Nikto 2: http://www.cirt.net/nikto2

|

References
========================================================================================

Whitepapers
---------------------------------------------------------------------------------------

- CIRT http://www.cirt.net/passwords
- Government Security - Default Logins and Passwords for Networked Devices http://www.governmentsecurity.org/articles/DefaultLoginsandPasswordsforNetworkedDevices.php
- Virus.org http://www.virus.org/default-password/