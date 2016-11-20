============================================================================================
OTG-INPVAL-006 (LDAP 인젝션 테스트)
============================================================================================

|

개요
============================================================================================

Lightweight Directory Access Protocol(LDAP)는 사용자, 호스트, 수많은 오브젝트에 관한
정보를 저장하기 위해 사용됩니다.
LDAP 인젝션은 서버 사이드 공격으로, 공개, 수정, 삽입 할 LDAP 구조에 표시된 사용자와 호스트에 관한 
민감한 정보입니다.

This is done by manipulating input parameters afterwards passed to internal search, add, and modify functions.
A web application could use LDAP in order to let users authenticate
or search other users’ information inside a corporate structure.
The goal of LDAP injection attacks is to inject LDAP search
filters metacharacters in a query which will be executed by the
application.
[Rfc2254] defines a grammar on how to build a search filter on
LDAPv3 and extends [Rfc1960] (LDAPv2).
An LDAP search filter is constructed in Polish notation, also known
as [prefix notation].
This means that a pseudo code condition on a search filter like
this:

.. code-block:: console

    find("cn=John & userPassword=mypass")

위 내용은 아래와 같이 표현될 것입니다.

.. code-block:: console

    find("(&(cn=John)(userPassword=mypass))")


Boolean conditions and group aggregations on an LDAP search filter
could be applied by using the following metacharacters:

More complete examples on how to build a search filter can be
found in the related RFC.
A successful exploitation of an LDAP injection vulnerability could
allow the tester to:

- Access unauthorized content
- Evade application restrictions
- Gather unauthorized informations
- Add or modify Objects inside LDAP tree structure.

|

테스트 방법
============================================================================================

**예제 1: Search Filters**
-------------------------------------------------------------------------------------------

다음과 같은 검색 필터를 사용하는 웹 어플리케이션을 가지고 있다고 가정합시다.

.. code-block:: console

    searchfilter="(cn="+user+")"

이 같은 HTTP 요청에 의해 초기화 됩니다.

.. code-block:: console

    http://www.example.com/ldapsearch?user=John

만약 *John* 값이 별표(*)로 바뀐다면, 요청을 다음과 같이 보냅니다.

.. code-block:: console

    http://www.example.com/ldapsearch?user=*

필터는 아래와 같이 처리할 것입니다.

.. code-block:: console

    searchfilter="(cn=*)"

*cn* 속성을 가진 모든 오브젝트들이 모두 일치되어 매칭됩니다.

만약 어플리케이션이 LDAP 인젝션에 취약하다면, 어플리케이션의 실행 흐름과 
LDAP에 연결된 사용자의 권한에 의존하는 사용자의 속성의 일부 또는 전체를 
보게될 것입니다. 

침투 테스터는 어플리케이션 에러를 체크하기 위하여 
파라미터에 '(', '|', '&', '*' 를 삽입하여 시행 착오를 사용합니다.

|

**예제 2: Login**
-------------------------------------------------------------------------------------------

If a web application uses LDAP to check user credentials during
the login process and it is vulnerable to LDAP injection, it is possible
to bypass the authentication check by injecting an always true
LDAP query (in a similar way to SQL and XPATH injection ).
Let’s suppose a web application uses a filter to match LDAP user/
password pair.

.. code-block:: console

    searchlogin= "(&(uid="+user+")(userPassword={MD5}"+base64(pack("H*",md5(pass)))+"))";

다음 값을 사용

.. code-block:: console

    user=*)(uid=*))(|(uid=*
    pass=password


검색 필터 결과

.. code-block:: console

    searchlogin="(&(uid=*)(uid=*))(|(uid=*)(userPassword={MD5}X03MO1qnZdYdgyfeuILPmQ==))";

which is correct and always true. This way, the tester will gain
logged-in status as the first user in LDAP tree.

|

Tools
============================================================================================

- Softerra LDAP Browser: http://www.ldapadministrator.com/


|

References
============================================================================================

Whitepapers
-------------------------------------------------------------------------------------------

- Sacha Faust: "LDAP Injection: Are Your Applications Vulnerable?": http://www.networkdls.com/articles/ldapinjection.pdf
- RFC 1960: "A String Representation of LDAP Search Filters": http://www.ietf.org/rfc/rfc1960.txt

|