============================================================================================
OTG-CONFIG-008 (RIA 크로스 도메인 정책 테스트)
============================================================================================

|

개요
============================================================================================

Rich Internet Applications (RIA) have adopted Adobe’s crossdomain.xml
policy files to allow for controlled cross domain access to
data and service consumption using technologies such as Oracle
Java, Silverlight, and Adobe Flash. Therefore, a domain can grant
remote access to its services from a different domain. However,
often the policy files that describe the access restrictions are
poorly configured. Poor configuration of the policy files enables
Cross-site Request Forgery attacks, and may allow third parties
to access sensitive data meant for the user.

|

cross-domain 정책 파일이 무엇인가?
============================================================================================

A cross-domain policy file specifies the permissions that a web
client such as Java, Adobe Flash, Adobe Reader, etc. use to access
data across different domains. For Silverlight, Microsoft adopted a
subset of the Adobe’s crossdomain.xml, and additionally created
it’s own cross-domain policy file: clientaccesspolicy.xml.

Whenever a web client detects that a resource has to be requested
from other domain, it will first look for a policy file in the target
domain to determine if performing cross-domain requests, including
headers, and socket-based connections are allowed.

Master policy files are located at the domain’s root. A client may
be instructed to load a different policy file but it will always check
the master policy file first to ensure that the master policy file permits
the requested policy file.

|

Crossdomain.xml vs. Clientaccesspolicy.xml
============================================================================================

Most RIA applications support crossdomain.xml. However in the
case of Silverlight, it will only work if the crossdomain.xml specifies
that access is allowed from any domain. For more granular
control with Silverlight, clientaccesspolicy.xml must be used.
Policy files grant several types of permissions:

- Accepted policy files (Master policy files can disable or restrict specific policy files)
- Sockets permissions
- Header permissions
- HTTP/HTTPS access permissions
- Allowing access based on cryptographic credentials

An example of an overly permissive policy file:

.. code-block:: html

    <?xml version="1.0"?>
    <!DOCTYPE cross-domain-policy SYSTEM
    "http://www.adobe.com/xml/dtds/cross-domain-policy.dtd">
    <cross-domain-policy>
        <site-control permitted-cross-domain-policies="all"/>
        <allow-access-from domain="*" secure="false"/>
        <allow-http-request-headers-from domain="*" headers="*" secure="false"/>
    </cross-domain-policy>

|

어떻게 cross domain 정책 파일을 악용할 수 있는가?
-------------------------------------------------------------------------------------------

- Overly permissive cross-domain policies.
- Generating server responses that may be treated as crossdomain policy files.
- Using file upload functionality to upload files that may be treated as cross-domain policy files.

|

cross-domain 액세스 악용 영향
-------------------------------------------------------------------------------------------

- CSRF 보안 파괴.
- Read data restricted or otherwise protected by cross-origin policies.

|

테스트 방법
============================================================================================

RIA 정책 파일 취약점 테스트:

테스터가 RIA 정책 파일 취약점을 테스트하기 위해서는 어플리케이션의 루트 폴더에서 
crossdomain.xml과 clientaccesspolicy.xml 정책 파일을 검색합니다.

예를 들어, 만약 어플리케이션의 URL이 http://www.owasp.org라면,
테스터는 http://www.owasp.org/crossdomain.xml과 http://www.owasp.org/clientaccesspolicy.xml 
파일을 다운로드 합니다.
모든 정책 파일을 검색한 후, 허용된 권한을 최소 권한 원칙에 따라 확인해야 합니다.

Requests should only come from the domains, ports, or protocols that are necessary.

지나치게 관대한 정책은 피해야합니다.

파일에 "*"로 정책이 있는 지 면밀히 검토해야 합니다.

**예제**

.. code-block:: html

    <cross-domain-policy>
        <allow-access-from domain="*" />
    </cross-domain-policy>


예상 결과:

- 정책 파일 리스트 발견
- 정책에 취약한 설정

|

Tools
============================================================================================

- Nikto
- OWASP ZAP
- W3af

|

References
============================================================================================

Whitepapers
-------------------------------------------------------------------------------------------

- UCSD: "Analyzing the Crossdomain Policies of Flash Applications" - http://cseweb.ucsd.edu/~hovav/dist/crossdomain.pdf
- Adobe: "Cross-domain policy file specification" - http://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html
- Adobe: "Cross-domain policy file usage recommendations for Flash Player" - http://www.adobe.com/devnet/flashplayer/articles/cross_domain_policy.html
- Oracle: "Cross-Domain XML Support": http://www.oracle.com/technetwork/java/javase/plugin2-142482.html#CROSSDOMAINXML
- MSDN: "Making a Service Available Across Domain Boundaries" 
- http://msdn.microsoft.com/en-us/library/cc197955(v=vs.95).aspx
- MSDN: "Network Security Access Restrictions in Silverlight" - http://msdn.microsoft.com/en-us/library/cc645032(v=vs.95).aspx
- Stefan Esser: "Poking new holes with Flash Crossdomain Policy Files" http://www.hardened-php.net/library/poking_new_holes_with_flash_crossdomain_policy_files.html
- Jeremiah Grossman: "Crossdomain.xml Invites Cross-site Mayhem" http://jeremiahgrossman.blogspot.com/2008/05/crossdomainxml-invites-cross-site.html
- Google Doctype: "Introduction to Flash security " - http://code.google.com/p/doctype-mirror/wiki/ArticleFlashSecurity

|
