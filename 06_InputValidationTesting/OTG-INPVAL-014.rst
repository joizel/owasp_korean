============================================================================================
OTG-INPVAL-014 (Buffer Overflow 침투 테스트)
============================================================================================

|

개요
============================================================================================

To find out more about buffer overflow vulnerabilities, please go
to Buffer Overflow pages.
See the OWASP article on Buffer Overflow Attacks.
See the OWASP article on Buffer Overflow Vulnerabilities.

|

테스트 방법
============================================================================================

Different types of buffer overflow vulnerabilities have different
testing methods. Here are the testing methods for the common
types of buffer overflow vulnerabilities.

- 힙 오버플로우 취약점 테스트
- 스택 오버플로우 취약점 테스트
- 포맷 스트링 취약점 테스트

Code Review
-----------------------------------------------------------------------------------------

See the OWASP Code Review Guide article on how to Review
Code for Buffer Overruns and Overflows Vulnerabilities.

|

Remediation
============================================================================================

See the OWASP Development Guide article on how to Avoid Buffer
Overflow Vulnerabilities.

|

|

============================================================================================
힙 오버플로우 취약점 테스트
============================================================================================

개요
============================================================================================

In this test the penetration tester checks whether a they can
make a Heap overflow that exploits a memory segment.

Heap is a memory segment that is used for storing dynamically
allocated data and global variables. Each chunk of memory in
heap consists of boundary tags that contain memory management
information.

When a heap-based buffer is overflowed the control information 
in these tags is overwritten. When the heap management routine
frees the buffer, a memory address overwrite takes place
leading to an access violation. When the overflow is executed in a
controlled fashion, the vulnerability would allow an adversary to
overwrite a desired memory location with a user-controlled value.
In practice, an attacker would be able to overwrite function
pointers and various addresses stored in structures like GOT,
.dtors or TEB with the address of a malicious payload.
There are numerous variants of the heap overflow (heap corruption)
vulnerability that can allow anything from overwriting
function pointers to exploiting memory management structures
for arbitrary code execution. Locating heap overflows requires
closer examination in comparison to stack overflows, since there
are certain conditions that need to exist in the code for these
vulnerabilities to be exploitable.

|

테스트 방법
============================================================================================

|

Black Box testing
-----------------------------------------------------------------------------------------

The principles of black box testing for heap overflows remain the
same as stack overflows. The key is to supply as input strings
that are longer than expected. Although the test process remains
the same, the results that are visible in a debugger are
significantly different. While in the case of a stack overflow, an
instruction pointer or SEH overwrite would be apparent, this
does not hold true for a heap overflow condition. When debugging
a windows program, a heap overflow can appear in several
different forms, the most common one being a pointer exchange
taking place after the heap management routine comes into action.
Shown below is a scenario that illustrates a heap overflow
vulnerability

The two registers shown, EAX and ECX, can be populated with
user supplied addresses which are a part of the data that is used
to overflow the heap buffer. One of the addresses can point to a
function pointer which needs to be overwritten, for example UEF
(Unhandled Exception filter), and the other can be the address of
user supplied code that needs to be executed.

When the MOV instructions shown in the left pane are executed,
the overwrite takes place and, when the function is called,
user supplied code gets executed. As mentioned previously, other
methods of testing such vulnerabilities include reverse engineering
the application binaries, which is a complex and tedious

process, and using fuzzing techniques.

|

Gray Box testing
-----------------------------------------------------------------------------------------

When reviewing code, one must realize that there are several
avenues where heap related vulnerabilities may arise. Code that
seems innocuous at the first glance can actually be vulnerable
under certain conditions. Since there are several variants of this
vulnerability, we will cover only the issues that are predominant.

Most of the time, heap buffers are considered safe by a lot of developers
who do not hesitate to perform insecure operations like
strcpy( ) on them. The myth that a stack overflow and instruction
pointer overwrite are the only means to execute arbitrary code
proves to be hazardous in case of code shown below:-

.. code-block:: c

    int main(int argc, char *argv[])
    {
     ……
     vulnerable(argv[1]);
     return 0;
    }
    int vulnerable(char *buf)
    {

     HANDLE hp = HeapCreate(0, 0, 0);

     HLOCAL chunk = HeapAlloc(hp, 0, 260);
     strcpy(chunk, buf); ‘’’ Vulnerability’’’

     ……..
     return 0;
    }

In this case, if buf exceeds 260 bytes, it will overwrite pointers in
the adjacent boundary tag, facilitating the overwrite of an arbitrary
memory location with 4 bytes of data once the heap management
routine kicks in.

Lately, several products, especially anti-virus libraries, have
been affected by variants that are combinations of an integer
overflow and copy operations to a heap buffer. As an example,
consider a vulnerable code snippet, a part of code responsible for
processing TNEF filetypes, from Clam Anti Virus 0.86.1, source
file tnef.c and function tnef_message( ):

.. code-block:: c

    string = cli_malloc(length + 1); ‘’’ Vulnerability’’’
    if(fread(string, 1, length, fp) != length) {‘’’ Vulnerability’’’
    free(string);
    return -1;
    }


The malloc in line 1 allocates memory based on the value of
length, which happens to be a 32 bit integer. In this particular example,
length is user-controllable and a malicious TNEF file can
be crafted to set length to ‘-1’, which would result in malloc( 0 ).
Therefore, this malloc would allocate a small heap buffer, which
would be 16 bytes on most 32 bit platforms (as indicated in malloc.h).
And now, in line 2, a heap overflow occurs in the call to fread(
). The 3rd argument, in this case length, is expected to be a
size_t variable. But if it’s going to be ‘-1’, the argument wraps to
0xFFFFFFFF, thus copying 0xFFFFFFFF bytes into the 16 byte
buffer.

Static code analysis tools can also help in locating heap related
vulnerabilities such as “double free” etc. A variety of tools like
RATS, Flawfinder and ITS4 are available for analyzing C-style
languages.

|

Tools
============================================================================================

- OllyDbg: “A windows based debugger used for analyzing buffer overflow vulnerabilities”: http://www.ollydbg.de
- Spike, A fuzzer framework that can be used to explore vulnerabilities and perform length testing: http://www.immunitysec.com/downloads/SPIKE2.9.tgz
- Brute Force Binary Tester (BFB), A proactive binary checker: http://bfbtester.sourceforge.net
- Metasploit, A rapid exploit development and Testing frame work - http://www.metasploit.com

|

References
============================================================================================

Whitepapers
-------------------------------------------------------------------------------------------

- w00w00: “Heap Overflow Tutorial”: http://www.cgsecurity.org/exploit/heaptut.txt
- David Litchfield: “Windows Heap Overflows”: http://www.blackhat.com/presentations/win-usa-04/bhwin-04-litchfield/bh-win-04-litchfield.ppt

|

|

============================================================================================
스택 오버플로우 취약점 테스트
============================================================================================

개요
============================================================================================

Stack overflows occur when variable size data is copied into fixed
length buffers located on the program stack without any bounds
checking. Vulnerabilities of this class are generally considered to
be of high severity since their exploitation would mostly permit
arbitrary code execution or Denial of Service. Rarely found in interpreted
platforms, code written in C and similar languages is
often ridden with instances of this vulnerability. In fact almost
every platform is vulnerable to stack overflows with the following
notable exceptions:

- J2EE – as long as native methods or system calls are not invoked
- .NET – as long as /unsafe or unmanaged code is not invoked (such as the use of P/Invoke or COM Interop)
- PHP – as long as external programs and vulnerable PHP

extensions written in C or C++ are not called can suffer from
stack overflow issues.
Stack overflow vulnerabilities often allow an attacker to directly
take control of the instruction pointer and, therefore, alter the
execution of the program and execute arbitrary code. Besides
overwriting the instruction pointer, similar results can also be
obtained by overwriting other variables and structures, like Exception
Handlers, which are located on the stack.

|

테스트 방법
============================================================================================

|

Black Box testing
-----------------------------------------------------------------------------------------

The key to testing an application for stack overflow vulnerabilities
is supplying overly large input data as compared to what is
expected. However, subjecting the application to arbitrarily large
data is not sufficient. It becomes necessary to inspect the application’s
execution flow and responses to ascertain whether an
overflow has actually been triggered or not. Therefore, the steps
required to locate and validate stack overflows would be to attach
a debugger to the target application or process, generate
malformed input for the application, subject the application to
malformed input, and inspect responses in a debugger. The debugger
allows the tester to view the execution flow and the state
of the registers when the vulnerability gets triggered.
On the other hand, a more passive form of testing can be employed,
which involves inspecting assembly code of the application
by using disassemblers. In this case, various sections are
scanned for signatures of vulnerable assembly fragments. This
is often termed as reverse engineering and is a tedious process.
As a simple example, consider the following technique employed
while testing an executable “sample.exe” for stack overflows:

.. code-block:: c

    #include<stdio.h>
    int main(int argc, char *argv[])
    {
     char buff[20];
     printf(“copying into buffer”);
     strcpy(buff,argv[1]);
     return 0;
    }

File sample.exe is launched in a debugger, in our case OllyDbg.

Since the application is expecting command line arguments, a
large sequence of characters such as ‘A’, can be supplied in the
argument field shown above.

On opening the executable with the supplied arguments and
continuing execution the following results are obtained.

As shown in the registers window of the debugger, the EIP or Extended
Instruction Pointer, which points to the next instruction
to be executed, contains the value ‘41414141’. ‘41’ is a hexadecimal
representation for the character ‘A’ and therefore the string
‘AAAA’ translates to 41414141.
This clearly demonstrates how input data can be used to overwrite
the instruction pointer with user-supplied values and control
program execution. A stack overflow can also allow overwriting
of stack-based structures like SEH (Structured Exception
Handler) to control code execution and bypass certain stack protection
mechanisms.
As mentioned previously, other methods of testing such vulnerabilities
include reverse engineering the application binaries,
which is a complex and tedious process, and using fuzzing techniques.

|

Gray Box testing
-----------------------------------------------------------------------------------------

When reviewing code for stack overflows, it is advisable to
search for calls to insecure library functions like gets(), strcpy(),
strcat() etc which do not validate the length of source strings and
blindly copy data into fixed size buffers.
For example consider the following function:-

.. code-block:: c

    void log_create(int severity, char *inpt) {
    char b[1024];
    if (severity == 1)
    {
    strcat(b,”Error occurred on”);
    strcat(b,”:”);
    strcat(b,inpt);
    FILE *fd = fopen (“logfile.log”, “a”);
    fprintf(fd, “%s”, b);
    fclose(fd);
    . . . . . .
    }

From above, the line strcat(b,inpt) will result in a stack overflow
if inpt exceeds 1024 bytes. Not only does this demonstrate an
insecure usage of strcat, it also shows how important it is to
examine the length of strings referenced by a character pointer
that is passed as an argument to a function; In this case the
length of string referenced by char *inpt. Therefore it is always
a good idea to trace back the source of function arguments and
ascertain string lengths while reviewing code.
Usage of the relatively safer strncpy() can also lead to stack
overflows since it only restricts the number of bytes copied into
the destination buffer. If the size argument that is used to accomplish
this is generated dynamically based on user input or
calculated inaccurately within loops, it is possible to overflow
stack buffers. For example:-    

.. code-block:: c

    void func(char *source)
    {
    Char dest[40];
    …
    size=strlen(source)+1
    ….
    strncpy(dest,source,size)
    }

where source is user controllable data. A good example would be
the samba trans2open stack overflow vulnerability (http://www.
securityfocus.com/archive/1/317615).
Vulnerabilities can also appear in URL and address parsing code.
In such cases, a function like memccpy() is usually employed
which copies data into a destination buffer from source until a
specified character is not encountered. Consider the function:

.. code-block:: c

    void func(char *path)
    {
    char servaddr[40];
    …
    memccpy(servaddr,path,’\’);
    ….
    }


In this case the information contained in path could be greater
than 40 bytes before ‘\’ can be encountered. If so it will cause a
stack overflow. A similar vulnerability was located in Windows
RPCSS subsystem (MS03-026). The vulnerable code copied
server names from UNC paths into a fixed size buffer until a ‘\’
was encountered. The length of the server name in this case was
controllable by users.
Apart from manually reviewing code for stack overflows, static
code analysis tools can also be of great assistance. Although
they tend to generate a lot of false positives and would barely be
able to locate a small portion of defects, they certainly help in reducing
the overhead associated with finding low hanging fruits,
like strcpy() and sprintf() bugs.
A variety of tools like RATS, Flawfinder and ITS4 are available for
analyzing C-style languages.

|

Tools
============================================================================================

- OllyDbg: “A windows based debugger used for analyzing buffer overflow vulnerabilities” - http://www.ollydbg.de
- Spike, A fuzzer framework that can be used to explore vulnerabilities and perform length testing - http://www.immunitysec.com/downloads/SPIKE2.9.tgz
- Brute Force Binary Tester (BFB), A proactive binary checker: http://bfbtester.sourceforge.net/
- Metasploit, A rapid exploit development and Testing frame work - http://www.metasploit.com

|

References
============================================================================================

Whitepapers
--------------------------------------------------------------------------------------------

- Aleph One: “Smashing the Stack for Fun and Profit”: http://insecure.org/stf/smashstack.html
- The Samba trans2open stack overflow vulnerability: http://www.securityfocus.com/archive/1/317615
- Windows RPC DCOM vulnerability details: http://www.xfocus.org/documents/200307/2.html

|

|

============================================================================================
포맷 스트링 취약점 테스트
============================================================================================

개요
============================================================================================

This section describes how to test for format string attacks that can
be used to crash a program or to execute harmful code. The problem
stems from the use of unfiltered user input as the format string
parameter in certain C functions that perform formatting, such as
printf().
Various C-Style languages provision formatting of output by means
of functions like printf( ), fprintf( ) etc. Formatting is governed by a
parameter to these functions termed as format type specifier, typically
%s, %c etc. The vulnerability arises when format functions are
called with inadequate parameters validation and user controlled
data.
A simple example would be printf(argv[1]). In this case the type specifier
has not been explicitly declared, allowing a user to pass characters
such as %s, %n, %x to the application by means of command line
argument argv[1].
This situation tends to become precarious since a user who can supply
format specifiers can perform the following malicious actions:

Enumerate Process Stack: This allows an adversary to view stack
organization of the vulnerable process by supplying format strings,
such as %x or %p, which can lead to leakage of sensitive information.
It can also be used to extract canary values when the application is
protected with a stack protection mechanism. Coupled with a stack
overflow, this information can be used to bypass the stack protector.
Control Execution Flow: This vulnerability can also facilitate arbitrary
code execution since it allows writing 4 bytes of data to an address
supplied by the adversary. The specifier %n comes handy for
overwriting various function pointers in memory with address of the
malicious payload. When these overwritten function pointers get
called, execution passes to the malicious code.
Denial of Service: If the adversary is not in a position to supply malicious
code for execution, the vulnerable application can be crashed
by supplying a sequence of %x followed by %n.

|

테스트 방법
============================================================================================

|

Black Box testing
--------------------------------------------------------------------------------------------

The key to testing format string vulnerabilities is supplying format
type specifiers in application input.
For example, consider an application that processes the URL string
http://xyzhost.com/html/en/index.htm or accepts inputs from
forms. If a format string vulnerability exists in one of the routines
processing this information, supplying a URL like http://xyzhost.
com/html/en/index.htm%n%n%n or passing %n in one of the form
fields might crash the application creating a core dump in the hosting
folder.

Format string vulnerabilities manifest mainly in web servers, application
servers, or web applications utilizing C/C++ based code or CGI
scripts written in C. In most of these cases, an error reporting or logging
function like syslog( ) has been called insecurely.
When testing CGI scripts for format string vulnerabilities, the input
parameters can be manipulated to include %x or %n type specifiers.
For example a legitimate request like

.. code-block:: html

    http://hostname/cgi-bin/query.cgi?name=john&code=45765 

.. code-block:: html

    http://hostname/cgi-bin/query.cgi?name=john%x.%x.%x
    &code=45765%x.%x

If a format string vulnerability exists in the routine processing this
request, the tester will be able to see stack data being printed out
to browser.
If code is unavailable, the process of reviewing assembly fragments
(also known as reverse engineering binaries) would yield substantial
information about format string bugs.
Take the instance of code (1) :

.. code-block:: html

    int main(int argc, char **argv)
    {
    printf(“The string entered is\n”);
    printf(“%s”,argv[1]);
    return 0;
    }

when the disassembly is examined using IDA Pro, the address of a
format type specifier being pushed on the stack is clearly visible before
a call to printf is made.

On the other hand, when the same code is compiled without “%s” as
an argument , the variation in assembly is apparent. As seen below,
there is no offset being pushed on the stack before calling printf.

|

Gray Box testing
--------------------------------------------------------------------------------------------

While performing code reviews, nearly all format string vulnerabilities
can be detected by use of static code analysis tools. Subjecting
the code shown in (1) to ITS4, which is a static code analysis tool,
gives the following output. 

The functions that are primarily responsible for format string vulnerabilities
are ones that treat format specifiers as optional. Therefore
when manually reviewing code, emphasis can be given to functions
such as:

.. code-block:: html

    printf
    fprintf
    sprintf
    snprintf
    vfprintf
    vprintf
    vsprintf
    vsnprintf

There can be several formatting functions that are specific to the
development platform. These should also be reviewed for absence
of format strings once their argument usage has been understood.

|

Tools
============================================================================================

- ITS4: “A static code analysis tool for identifying format string
vulnerabilities using source code” - http://www.cigital.com/its4
- An exploit string builder for format bugs - http://seclists.org/
lists/pen-test/2001/Aug/0014.html

|

References
============================================================================================

Whitepapers
--------------------------------------------------------------------------------------------

- Format functions manual page: http://www.die.net/doc/linux/man/man3/fprintf.3.html
- Tim Newsham: “A paper on format string attacks”: http://comsec.theclerk.com/CISSP/FormatString.pdf
- Team Teso: “Exploiting Format String Vulnerabilities”: http://www.cs.ucsb.edu/~jzhou/security/formats-teso.html
- Analysis of format string bugs: http://julianor.tripod.com/format-bug-analysis.pdf

|