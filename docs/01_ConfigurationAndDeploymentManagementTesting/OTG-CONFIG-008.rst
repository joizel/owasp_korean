============================================================================================
OTG-CONFIG-008 (RIA 크로스 도메인 정책 테스트)
============================================================================================

|

개요
============================================================================================

RIA (Rich Internet Application)는 Adobe Java, Silverlight 및 Adobe Flash와 같은 기술을 사용하여 데이터 및 서비스 소비에 대한 제어 된 상호 도메인 액세스를 허용하기 위해 Adobe의 crossdomain.xml 정책 파일을 채택했습니다. 
따라서 도메인은 다른 도메인의 서비스에 대한 원격 액세스 권한을 부여 할 수 있습니다. 
그러나 종종 액세스 제한을 설명하는 정책 파일은 잘못 구성됩니다. 
정책 파일을 잘못 구성하면 교차 사이트 요청 위조 공격이 가능하고 제 3자가 사용자를위한 중요한 데이터에 액세스 할 수 있습니다.

|

**cross-domain 정책 파일이 무엇인가?**

cross-domain 정책 파일은 Java, Adobe Flash, Adobe Reader 등과 같은 웹 클라이언트가 다른 도메인의 데이터에 액세스하는 데 사용하는 권한을 지정합니다. 
Silverlight의 경우 Microsoft는 Adobe의 crossdomain.xml 하위 집합을 채택하고 자체적으로 cross-domain 정책 파일 인 clientaccesspolicy.xml을 추가로 만들었습니다.

웹 클라이언트에서 리소스가 다른 도메인에서 요청되어야 한다는 것을 감지 할 때마다 대상 도메인의 정책 파일을 먼저 찾고 헤더를 포함한 도메인 간 요청 수행 및 소켓 기반 연결이 허용되는지 확인합니다.

마스터 정책 파일은 도메인 루트에 있습니다. 클라이언트는 다른 정책 파일을 로드하도록 지시받을 수 있지만 항상 마스터 정책 파일을 검사하여 마스터 정책 파일이 요청 된 정책 파일을 허용하는지 확인합니다.

|

**Crossdomain.xml vs. Clientaccesspolicy.xml**

대부분의 RIA 응용 프로그램은 crossdomain.xml을 지원합니다. 
그러나 Silverlight의 경우 crossdomain.xml이 모든 도메인에서 액세스가 허용되도록 지정한 경우에만 작동합니다. 
Silverlight를 사용하여 보다 세밀하게 제어하려면 clientaccesspolicy.xml을 사용해야 합니다.

- 수락된 정책 파일 (마스터 정책 파일은 특정 정책 파일을 비활성화하거나 제한 할 수 있음)
- 소켓 사용 권한
- 헤더 사용 권한
- HTTP / HTTPS 액세스 권한
- 암호화 자격 증명을 기반으로 액세스 허용

지나치게 관대한 정책 파일의 예 :

.. code-block:: html

    <?xml version="1.0"?>
    <!DOCTYPE cross-domain-policy SYSTEM "http://www.adobe.com/xml/dtds/cross-domain-policy.dtd">
    <cross-domain-policy>
        <site-control permitted-cross-domain-policies="all"/>
        <allow-access-from domain="*" secure="false"/>
        <allow-http-request-headers-from domain="*" headers="*" secure="false"/>
    </cross-domain-policy>

|

**어떻게 cross domain 정책 파일을 악용할 수 있는가?**

- 지나치게 허용되는 교차 도메인 정책.
- 도메인 간 정책 파일로 취급 될 수 있는 서버 응답 생성
- 파일 업로드 기능을 사용하여 도메인 간 정책 파일로 간주 될 수 있는 파일을 업로드합니다.

|

**cross-domain 액세스 악용 영향**

- CSRF 보호를 무력화
- 원본이 아닌 정책으로 제한되거나 보호되는 데이터를 읽습니다.

|

테스트 방법
============================================================================================

RIA 정책 파일 취약점 테스트:

테스터가 RIA 정책 파일 취약점을 테스트하기 위해서는 어플리케이션의 루트 폴더에서 crossdomain.xml과 clientaccesspolicy.xml 정책 파일을 검색합니다.

예를 들어, 만약 어플리케이션의 URL이 http://www.owasp.org라면, 테스터는 http://www.owasp.org/crossdomain.xml과 http://www.owasp.org/clientaccesspolicy.xml 파일을 다운로드 합니다.
모든 정책 파일을 검색한 후, 허용된 권한을 최소 권한 원칙에 따라 확인해야 합니다.
요청은 필요한 도메인, 포트 또는 프로토콜에서만 발생해야합니다.
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

도구
============================================================================================

- Nikto
- OWASP ZAP
- W3af

|

참고 문헌
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
