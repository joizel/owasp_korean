==========================================================================================
OTG-AUTHN-003
==========================================================================================

|

취약한 잠금 매카니즘에 대한 테스트

|

개요
==========================================================================================

Account lockout mechanisms are used to mitigate brute force password guessing attacks. 
Accounts are typically locked after 3 to 5 unsuccessful login attempts and can only be unlocked after a predetermined period of time, via a self-service unlock mechanism, or intervention by an administrator. 
Account lockout mechanisms require a balance between protecting accounts from unauthorized access and protecting users from being denied authorized access.
Note that this test should cover all aspects of authentication where lockout mechanisms would be appropriate, e.g. when the user is presented with security questions during forgotten password mechanisms (see Testing for Weak security question/answer (OTG-AUTHN-008)).
Without a strong lockout mechanism, the application may be susceptible to brute force attacks. After a successful brute force attack, a malicious user could have access to:

- Confidential information or data: Private sections of a web
application could disclose confidential documents, users’ profile
data, financial information, bank details, users’ relationships, etc.
- Administration panels: These sections are used by webmasters
to manage (modify, delete, add) web application content, manage
user provisioning, assign different privileges to the users, etc.
- Opportunities for further attacks: authenticated sections of a
web application could contain vulnerabilities that are not present
in the public section of the web application and could contain
advanced functionality that is not available to public users.

|

테스트 목적
==========================================================================================

- 브루트 포스 패스워드 게싱을 방어하기 위한 계정 잠금 메커니즘 평가
- 인증되지 않은 계정 잠금 해제로 해제 메커니즘 방어 평가

|

테스트 방법
==========================================================================================

Typically, to test the strength of lockout mechanisms, you will
need access to an account that you are willing or can afford to lock.
If you have only one account with which you can log on to the web
application, perform this test at the end of you test plan to avoid
that you cannot continue your testing due to a locked account.
To evaluate the account lockout mechanism’s ability to mitigate
brute force password guessing, attempt an invalid log in by using
the incorrect password a number of times, before using the correct
password to verify that the account was locked out. An example
test may be as follows:

1. 잘못된 패스워드로 3번 로그인 시도
2. 올바른 패스워드로 성공적으로 로그인
이에 따라 3번 잘못된 인증 시도 이 후 잠금 메커니즘이 실행되지 않음을 보여줍니다.
3. 잘못된 패스워드로 4번 로그인 시도
4. 올바른 패스워드로 성공적으로 로그인
이에 따라 4번 잘못된 인증 시도 이 후 잠금 메커니즘이 실행되지 않음을 보여줍니다.
5. 잘못된 패스워드로 5번 로그인 시도
6. 올바른 패스워드로 성공적으로 로그인
The application returns “Your account is locked out.”, thereby confirming that the account is locked out after 5 incorrect authentication attempts.
7. Attempt to log in with the correct password 5 minutes later. 
The application returns “Your account is locked out.”, thereby showing that the lockout mechanism does not automatically unlock after 5 minutes.
8. Attempt to log in with the correct password 10 minutes later.
The application returns “Your account is locked out.”, thereby showing that the lockout mechanism does not automatically unlock after 10 minutes.
9. Successfully log in with the correct password 15 minutes later,thereby showing that the lockout mechanism automatically unlocks after a 10 to 15 minute period.

A CAPTCHA may hinder brute force attacks, but they can come
with their own set of weaknesses (see Testing for CAPTCHA), and
should not replace a lockout mechanism.

To evaluate the unlock mechanism’s resistance to unauthorized
account unlocking, initiate the unlock mechanism and look for
weaknesses.
Typical unlock mechanisms may involve secret questions or an
emailed unlock link. The unlock link should be a unique one-time
link, to stop an attacker from guessing or replaying the link and
performing brute force attacks in batches. Secret questions and
answers should be strong (see Testing for Weak Security Question/Answer).
Note that an unlock mechanism should only be used for unlocking
accounts. It is not the same as a password recovery mechanism.
Factors to consider when implementing an account lockout mechanism:

- What is the risk of brute force password guessing against the application?
- Is a CAPTCHA sufficient to mitigate this risk?
- Number of unsuccessful log in attempts before lockout. If the
lockout threshold is to low then valid users may be locked out too
often. If the lockout threshold is to high then the more attempts
an attacker can make to brute force the account before it will be
locked. Depending on the application’s purpose, a range of 5 to 10
unsuccessful attempts is typical lockout threshold.
- How will accounts be unlocked?

• Manually by an administrator: this is the most secure lockout
method, but may cause inconvenience to users and take up the
administrator’s “valuable” time.
- Note that the administrator should also have a recovery method
in case his account gets locked.
- This unlock mechanism may lead to a denial-of-service attack
if an attacker’s goal is to lock the accounts of all users of the web
application.
• After a period of time: What is the lockout duration?
Is this sufficient for the application being protected? E.g. a 5 to
30 minute lockout duration may be a good compromise between
mitigating brute force attacks and inconveniencing valid users.
• Via a self-service mechanism: As stated before, this self-service
mechanism must be secure enough to avoid that the attacker can
unlock accounts himself.

|

References
==========================================================================================

See the OWASP article on Brute Force Attacks.

|

Remediation
==========================================================================================

Apply account unlock mechanisms depending on the risk level. In order from lowest to highest assurance:

- Time-based lockout and unlock.
- Self-service unlock (sends unlock email to registered email address).
- Manual administrator unlock.
- Manual administrator unlock with positive user identification.

|