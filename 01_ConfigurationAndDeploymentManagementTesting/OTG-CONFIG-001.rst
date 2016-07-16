==========================================================================================
OTG-CONFIG-001
==========================================================================================

|

네트워크 및 인프라 설정 테스트

|

개요
==========================================================================================


The intrinsic complexity of interconnected and heterogeneous web server infrastructure, which can include hundreds of web applications, makes configuration management and review a fundamental step in testing and deploying every single application. 

It takes only a single vulnerability to undermine the security of the entire infrastructure, and even small and seemingly unimportant problems may evolve into severe risks for another application on the same server. In order to address these problems, it is of utmost importance to perform an in-depth review of configuration and known security issues, after having mapped the entire architecture. 

Proper configuration management of the web server infrastructure is very important in order to preserve the security of the application it¡©self. If elements such as the web server software, the back-end data¡©base servers, or the authentication servers are not properly reviewed and secured, they might introduce undesired risks or introduce new vulnerabilities that might compromise the application itself. 

For example, a web server vulnerability that would allow a remote attacker to disclose the source code of the application itself (a vul¡©nerability that has arisen a number of times in both web servers or application servers) could compromise the application, as anonymous users could use the information disclosed in the source code to lever¡©age attacks against the application or its users. 

The following steps need to be taken to test the configuration man¡©agement infrastructure: 

- The different elements that make up the infrastructure need to be determined in order to understand how they interact with a web application and how they affect its security.
- All the elements of the infrastructure need to be reviewed in order to make sure that they don`t contain any known vulnerabilities. 
- A review needs to be made of the administrative tools used to maintain all the different elements. 
- The authentication systems, need to reviewed in order to assure that they serve the needs of the application and that they cannot be manipulated by external users to leverage access. 
- A list of defined ports which are required for the application should be maintained and kept under change control. 

After having mapped the different elements that make up the infra©structure (see Map Network and Application Architecture) it is possible to review the configuration of each element founded and test for any known vulnerabilities. 

|

테스트 방법
==========================================================================================

**알려진 서버 취약점**

어플리케이션 구조의 다른 영역에서 발견되는 취약점은 웹 서버나 백 엔드 데이터베이스에 있는 것으로 애플리케이션 자체를 손상시킬 수 있습니다.
예를 들어, 비인가된 사용자가 원격에서 웹 서버에 파일을 업로드하거나 파일을 교체할 수 있다고 고려해봅시다.
취약점은 어플리케이션 자체를 교체하거나 백 엔드 서버에 영향을 미칠 수 있는 코드를 삽입하여 어플리케이션을 손상시킬 수 있습니다.
서버 취약점 검토는 블라인드 침투 테스트를 통해 해야한다면 어려울 수 있습니다.
이러한 경우 취약점은 일반적으로 자동화된 툴을 이용하여 원격지에서 테스트되어야 합니다.

그러나, 일부 취약점 테스트는 웹 서버에 예기치 않은 결과를 가질 수 있고, 테스트가 성공할 경우 관련된 서비스가 중지될 수 도 있습니다.
일부 자동화 툴은 웹 서버 버전에 기반한 플래그 취약점을 기반으로 합니다.
이것은 진실 혹은 거짓으로 결정하며, 웹 서버 버전이 제거된 경우나 로컬 사이트 관리자가 모호하다면 스캔 툴을 사용하지 않습니다.

On the other hand, if the vendor providing the software does not update the web server version when vulnerabilities are fixed, the scan tool will flag vulnerabilities that do not exist. 

The latter case is actually very common as some operating system vendors back port patches of security vulnerabilities to the software they provide in the operating system, but do not do a full upload to the latest software version. 

This happens in most GNU/Linux distributions such as Debian, Red Hat or SuSE. 

In most cases, vulnerability scanning of an application architecture will only find vulnerabilities associated with the "exposed" elements of the architecture (such as the web server) and will usually be unable to find vulnerabilities associated to elements which are not directly exposed, such as the authentication back ends, the back end database, or reverse proxies in use. 

Finally, not all software vendors disclose vulnerabilities in a public way, and therefore these weaknesses do not become registered within publicly known vulnerability databases[2]. This information is only disclosed to customers or published through fixes that do not have accompanying advisories. This reduces the usefulness of vulnerability scanning tools. 

Typically, vulnerability coverage of these tools will be very good for common products (such as the Apache web server, Microsoft`s Internet Information Server, or IBM`s Lotus Domino) but will be lacking for lesser known products. 

This is why reviewing vulnerabilities is best done when the tester is provided with internal information of the software used, including versions and releases used and patches applied to the software. 

With this information, the tester can retrieve the information from the vendor itself and analyze what vulnerabilities might be present in the architecture and how they can affect the application itself. 

When possible, these vulnerabilities can be tested to determine their real effects and to detect if there might be any external elements (such as intrusion detection or prevention systems) that might reduce or negate the possibility of successful exploitation. Testers might even determine, through a configuration review, that the vulnerability is not even present, since it affects a software component that is not in use. 

It is also worthwhile to note that vendors will sometimes silently fix vulnerabilities and make the fixes available with new software releases. 

Different vendors will have different release cycles that determine the support they might provide for older releases. 

A tester with detailed information of the software versions used by the architecture can analyse the risk associated to the use of old software releases that might be unsupported in the short term or are already unsupported. This is critical, since if a vulnerability were to surface in an old software version that is no longer supported, the systems personnel might not be directly aware of it. No patches will be ever made available for it and advisories might not list that version as vulnerable as it is no longer supported. Even in the event that they are aware that the vulnerability is present and the system is vulnerable, they will need to do a full upgrade to a new software release, which might introduce significant downtime in the application architecture or might force the application to be re-coded due to incompatibilities with the latest software version. 

|

Administrative Tools
==========================================================================================

Any web server infrastructure requires the existence of administrative tools to maintain and update the information used by the application. This information includes static content (web pages, graphic files), application source code, user authentication databases, etc. Administrative tools will differ depending on the site, technology, or software used. For example, some web servers will be managed using administrative interfaces which are, themselves, web servers (such as the iPlanet web server) or will be administrated by plain text configuration files (in the Apache case[3]) or use operating-system GUI tools (when using Microsoft`s IIS server or ASP.Net). 

In most cases the server configuration will be handled using different file maintenance tools used by the web server, which are managed through FTP servers, WebDAV, network file systems (NFS, CIFS) or other mechanisms. Obviously, the operating system of the elements that make up the application architecture will also be managed using other tools. Applications may also have administrative interfaces embedded in them that are used to manage the application data itself (users, content, etc.). 

After having mapped the administrative interfaces used to manage the different parts of the architecture it is important to review them since if an attacker gains access to any of them he can then compromise or damage the application architecture. To do this it is important to: 

- Determine the mechanisms that control access to these interfaces and their associated susceptibilities. This information may be available online. 
- Change the default username and password. 

Some companies choose not to manage all aspects of their web server applications, but may have other parties managing the content delivered by the web application. This external company might either provide only parts of the content (news updates or promotions) or might manage the web server completely (including content and code). It is common to find administrative interfaces available from the Internet in these situations, since using the Internet is cheaper than providing a dedicated line that will connect the external company to the application infrastructure through a management-only interface. In this situation, it is very important to test if the administrative interfaces can be vulnerable to attacks. 

|

References
==========================================================================================

- WebSEAL, also known as Tivoli Authentication Manager, is a reverse proxy from IBM which is part of the Tivoli framework. 
- Such as Symantec`s Bugtraq, ISS` X-Force, or NIST`s National Vulnerability Database (NVD). 
- There are some GUI-based administration tools for Apache (like NetLoony) but they are not in widespread use yet. 

|