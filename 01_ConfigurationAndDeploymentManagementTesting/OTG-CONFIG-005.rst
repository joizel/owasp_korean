============================================================================================
OTG-CONFIG-005 (인프라와 어플리케이션 관리자 인터페이스 확인)
============================================================================================

|

개요
==========================================================================================

Administrator interfaces may be present in the application or on the
application server to allow certain users to undertake privileged activities
on the site. Tests should be undertaken to reveal if and how
this privileged functionality can be accessed by an unauthorized or
standard user.

An application may require an administrator interface to enable a privileged
user to access functionality that may make changes to how the
site functions. Such changes may include:

- user account provisioning
- site design and layout
- data manipulation
- configuration changes

In many instances, such interfaces do not have sufficient controls to
protect them from unauthorized access. Testing is aimed at discovering
these administrator interfaces and accessing functionality intended
for the privileged users.

|

테스트 방법
==========================================================================================

|

Black Box Testing
-----------------------------------------------------------------------------------------

이번 섹션은 관리 인터페이스 존재를 테스트 하기 위해
사용되는 벡터를 설명합니다.

이 기술들은 권한 상승 관련 문제를 테스트하는데 사용될 수 있습니다.
(인증 스키마 우회 테스트(OTG-AUTHZ-002)와 안전하지 않은 직접 객체 참조 테스트(OTG-AUTHZ-004) 참고)

- 디렉토리와 파일 리스팅.
Google dorks 등의 내용을 기반으로 관리 인터페이스의 경로 추측한다.
- 서버 컨텐츠를 브루트 포싱을 할 수 있는 툴이 많이 존재합니다.
테스터는 관리 페이지의 파일명도 식별해야합니다.
- 소스 코드에 있는 주석과 링크.
대부분의 사이트는 모든 사이트 사용자들에게 로드하기 위해 공통 코드를 사용합니다.
클라이언트로 전송된 모든 소스를 검사하다보면, 관리자 기능에 대한 링크가 발견될 수 있으니 검사해야합니다.
- 서버와 어플리케이션 문서 검토
만약 어플리케이션 서버 또는 어플리케이션이 기본 설정으로 배포된 것이라면, 
설정 또는 도움말 문서에 설명된 정보를 사용하여 관리 인터페이스에 저액세스할 수 있습니다.
관리 인터페이스가 발견되어 자격 증명이 요구되면, 기본 패스워드 리스트를 참고해야 합니다.
- 공개 이용 정보
워드프레스와 같은 어플리케이션 대부분이 기본 관리인터페이스를 가지고 있습니다.
- 서버 포트 대체
관리 인터페이스는 주요 어플리케이션과 다른 포트를 사용하기도 합니다.
예를 들어, 아파치 톰캣의 관리 인터페이스는 8080 포트를 사용합니다.
- 파라미터 침투. 
GET/POST 파라미터, 또는 cookie 변수를 통해 기능적으로 관리자를 활성화할 수도 있습니다.

.. code-block:: html

    <input type="hidden" name="admin" value="no">

또는, cookie에서

.. code-block:: html

    Cookie: session_cookie; useradmin=0

관리 인터페이스가 발견되면, 인증 우회를 하기 위해 위의 기술들을 조합하여 사용할 수 있습니다.
만약 실패하면, 테스터는 브루트 포스 공격을 시도해야 합니다.

그러한 경우 테스터는 관리 계정 잠금에 대한 가능성도 알고 있어야 합니다.

|

Gray Box Testing
-----------------------------------------------------------------------------------------

A more detailed examination of the server and application components
should be undertaken to ensure hardening (i.e. administrator
pages are not accessible to everyone through the use of IP filtering
or other controls), and where applicable, verification that all components
do not use default credentials or configurations.

Source code should be reviewed to ensure that the authorization and
authentication model ensures clear separation of duties between
normal users and site administrators. User interface functions shared
between normal and administrator users should be reviewed to ensure
clear separation between the drawing of such components and
information leakage from such shared functionality.

|

Tools
==========================================================================================

- Dirbuster
- THC-HYDRA
- brute forcer

|

References
==========================================================================================

- 기본 패스워드 리스트: http://www.governmentsecurity.org/articles/DefaultLoginsandPasswordsforNetworkedDevices.php
- 기본 패스워드 리스트: http://www.cirt.net/passwords

|



























