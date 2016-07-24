==========================================================================================
OTG-AUTHN-003 (취약한 잠금 매카니즘 테스트)
==========================================================================================

|

개요
==========================================================================================

계정 잠금 메카니즘은 패스워드 추측 무작위 공격에 대한 보안으로 사용됩니다.
계정은 일반적으로 3번에서 5번 로그인 시도가 성공하지 못하면 잠기고 일정 시간 후에 자체 해제 메카니즘 또는 관리자 개입을 통해 풀릴 수 있습니다.
계정 잠금 메카니즘은 비인가 접근 사용자와 거부된 인가 접근 사용자 사이에 균형이 필요합니다.

테스트할 때 잠금 메카니즘은 인증의 모든 측면을 포함하고 있어야 합니다.
예, 사용자가 패스워드를 잃어버렸을 때 보안 질문으로 복구하는 메카니즘
(취약한 보안 질의응답 테스트 (OTG-AUTHN-008)).

어플리케이션은 강력한 잠금 메카니즘이 없이는 무작위 공격에 대해 취약할 수 있습니다.
무작위 공격이 성공하게 되면 악의적인 사용자는 접근이 가능하게 됩니다.

- 기밀 정보 또는 데이터: 

Private sections of a web application could disclose confidential documents, users’ profile data, financial information, bank details, users’ relationships, etc.

- 관리자 페이지: 
These sections are used by webmasters to manage (modify, delete, add) web application content, manage user provisioning, assign different privileges to the users, etc.

- 추가 공격 기회: 
authenticated sections of a web application could contain vulnerabilities that are not present in the public section of the web application and could contain advanced functionality that is not available to public users.

|

테스트 목적
==========================================================================================

- 브루트 포스 패스워드 게싱을 방어하기 위한 계정 잠금 메커니즘 평가
- 인증되지 않은 계정 잠금 해제로 해제 메커니즘 방어 평가

|

테스트 방법
==========================================================================================

일반적으로, 잠금 메카니즘의 강도를 테스트하는 것은 잠겨도 상관없는 계정에 대한 
접근이 필요합니다.
만약 웹 어플리케이션으로 로그온 한 계정 뿐이라면, 잠긴 계정 때문에 테스트를 계속할 수 없는 것을 피하기 위해 테스트 마지막에 수행하도록 합니다.

잠금 메카니즘 능력 평가를 위해서는 잘못된 패스워드를 여러 번 사용하여 잘못된 로그인 시도를 해야합니다.
예를 들어 아래 테스트와 같습니다.

1. 잘못된 패스워드로 3번 로그인 시도
2. 올바른 패스워드로 성공적으로 로그인
이에 따라 3번 잘못된 인증 시도 이 후 잠금 메커니즘이 실행되지 않음을 보여줍니다.
3. 잘못된 패스워드로 4번 로그인 시도
4. 올바른 패스워드로 성공적으로 로그인
이에 따라 4번 잘못된 인증 시도 이 후 잠금 메커니즘이 실행되지 않음을 보여줍니다.
5. 잘못된 패스워드로 5번 로그인 시도
6. 올바른 패스워드로 성공적으로 로그인
어플리케이션은 "Your account is locked out."을 리턴, 이에 따라 계정은 5회 잘못된 인증 시도 잠금되는 걸 확인
7. 5분 후 올바른 패스워드로 로그인 시도
어플리케이션은 "Your account is locked out."을 리턴, 이에 따라 잠금 메카니즘은 5분 후 자동으로 해제되지 않는 것을 확인
8. 10분 후 올바른 패스워드로 로그인 시도
어플리케이션은 "Your account is locked out."을 리턴, 이에 따라 잠금 메카니즘은 10분 후 자동으로 해제되지 않는 것을 확인
9. 15분 후에 올바른 패스워드로 로그인 성공, 이에 따라 잠금 메카니즘은 10분에서 15분 사이에 자동으로 해제되는 것을 확인

A CAPTCHA may hinder brute force attacks, but they can come with their own set of weaknesses (see Testing for CAPTCHA), and should not replace a lockout mechanism.

To evaluate the unlock mechanism’s resistance to unauthorized account unlocking, initiate the unlock mechanism and look for weaknesses.
Typical unlock mechanisms may involve secret questions or an emailed unlock link. 
The unlock link should be a unique one-time link, to stop an attacker from guessing or replaying the link and performing brute force attacks in batches. 
Secret questions and answers should be strong (see Testing for Weak Security Question/Answer).

Note that an unlock mechanism should only be used for unlocking accounts. 
It is not the same as a password recovery mechanism.
Factors to consider when implementing an account lockout mechanism:

- What is the risk of brute force password guessing against the application?
- Is a CAPTCHA sufficient to mitigate this risk?
- Number of unsuccessful log in attempts before lockout. If the lockout threshold is to low then valid users may be locked out too often. 
If the lockout threshold is to high then the more attempts an attacker can make to brute force the account before it will be locked. 
Depending on the application’s purpose, a range of 5 to 10 unsuccessful attempts is typical lockout threshold.
- How will accounts be unlocked?

- Manually by an administrator: this is the most secure lockout method, but may cause inconvenience to users and take up the administrator’s “valuable” time.
- Note that the administrator should also have a recovery method in case his account gets locked.
- This unlock mechanism may lead to a denial-of-service attack if an attacker’s goal is to lock the accounts of all users of the web application.
- After a period of time: What is the lockout duration? Is this sufficient for the application being protected? E.g. a 5 to 30 minute lockout duration may be a good compromise between mitigating brute force attacks and inconveniencing valid users.
- Via a self-service mechanism: As stated before, this self-service mechanism must be secure enough to avoid that the attacker can unlock accounts himself.

|

References
==========================================================================================

무작위 공격에 대한 OWASP 자료 확인

|

Remediation
==========================================================================================

위험 수준에 따라 계정 잠금 해제 메커니즘을 적용합니다.

- 시간 기반 잠금과 잠금 해제
- 셀프 서비스 잠금 해제 (등록된 이메일 주소로 잠금 해제 메일 전송).
- 관리자가 수동으로 잠금 해제
- 관리자가 수동으로 정상 사용자 식별하여 잠금 해제

|