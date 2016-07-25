============================================================================================
OTG-CRYPST-001 (취약한 SSL/TLS 암호, 불충분한 전송 계층 보호 침투 테스트)
============================================================================================

|

개요
==========================================================================================

민감한 데이터는 네트워크를 통해 전송될 때 보호되어야 하기 때문에, 
사용자 자격 증명과 신용 키를 포함하고 있어야 합니다. 
데이터가 저장될 때 보호되어야 하는 경우, 전송 시에도 역시 보호되어야 합니다.

HTTP는 평문 프로토콜으로 일반적으로 SSL/TLS 터널을 통해 보안되며, 이를 HTTPS 트래픽이라고 합니다.
HTTPS 프로토콜 사용은 기밀성 뿐만 아니라 인증 역시 보장합니다. 
서버가 디지털 인증서를 사용하여 인증되고, 상호 인증을 위해 클라이언트 인증을 사용하여야 합니다.

오늘날 일반적으로 사용되고 지원하는 고급 암호 메커니즘인 경우에도, 서버에 일부 잘못된 설정이 
공격자 접속을 허용하는 잘못된 암호 메커니즘 사용으로 강제 사용될 수 있습니다.
일부 잘못된 설정은 서비스 거부 공격에 사용될 수 있습니다.

|

일반적인 문제
-------------------------------------------------------------------------------------------

취약점은 민감한 정보 전송 시 HTTP 프로토콜을 사용하면 발생할 수 있습니다.

SSL/TLS 서비스가 존재하는 경우에는 양호하지만, 공격 표면을 증가시키고 다음 취약점이 존재할 수 있습니다.

- SSL/TLS 프로토콜, Ciphers, Keys 그리고 재협상이 제대로 설정되어 있어야합니다.
- 인증서 유효 기간이 보장되어야합니다.

이와 관련한 또 다른 취약점

- 노출된 소프트웨어이기에 취약점이 알려진 경우 업데이트 해야 합니다.
- Session 쿠키를 위해 Secure 플래그 사용
- HTTP 엄격한 전송보안(HSTS) 사용
- 트래픽을 차단하는 데 사용할 수 있는 HTTP 및 HTTPS 존재
- 정보 누출에 사용되 수 있는 동일한 페이지에 혼합 HTTPS 및 HTTP 컨텐츠의 존재

|

민감한 데이터 평문 전송
-------------------------------------------------------------------------------------------

어플리케이션은 암호화되지 않은 채널을 통해 민감한 정보를 전송하지 말아야합니다.
일반적으로 HTTP를 통해 기본 인증을 사용할 경우, HTTP를 통해 전송한 입력 비밀번호 또는 세션 쿠키와 규정, 법률 또는 조직 정책에 의해 고려될 일반 정보 등을 발견 할 수 있습니다.

|

취약한 SSL/TLS Ciphers/Protocols/Keys
-------------------------------------------------------------------------------------------

역사적으로, 암호 시스템은 최대 40 비트 크기 이하의 키라면 파괴 될 수 있고, 통신의 암호 해독을 허용되기 때문에 미국 정부에서 설정한 제한이 있었습니다.
이 후 암호화는 최대 키 크기가 128 비트로 규제가 완화되어 출력되었습니다.

It is important to check the SSL configuration being used to avoid putting in place cryptographic support which could be easily defeated. To reach this goal SSL-based services should not offer the possibility to choose weak cipher suite. A cipher suite is specified by an encryption protocol (e.g. DES, RC4, AES), the encryption key length (e.g. 40, 56, or 128 bits), and a hash algorithm (e.g. SHA, MD5) used for integrity checking. 
Briefly, the key points for the cipher suite determination are the following: 

1. The client sends to the server a ClientHello message specifying, among other information, the protocol and the cipher suites that it is able to handle. Note that a client is usually a web browser (most popular SSL client nowadays), but not necessarily, since it can be any SSL-enabled application; the same holds for the server, which needs not to be a web server, though this is the most common case [9]. 
2. The server responds with a ServerHello message, containing the chosen protocol and cipher suite that will be used for that session (in general the server selects the strongest protocol and cipher suite supported by both the client and server). 

It is possible (for example, by means of configuration directives) to specify which cipher suites the server will honor. In this way you may control whether or not conversations with clients will support 40-bit encryption only. 

1. The server sends its Certificate message and, if client authentication is required, also sends a CertificateRequest message to the client. 
2. The server sends a ServerHelloDone message and waits for a client response. 
3. Upon receipt of the ServerHelloDone message, the client verifies the validity of the server's digital certificate. 

|

SSL 인증 유효성 테스트 - 클라이언트와 서버
-------------------------------------------------------------------------------------------

When accessing a web application via the HTTPS protocol, a secure channel is established between the client and the server. The identity of one (the server) or both parties (client and server) is then established by means of digital certificates. So, once the cipher suite is determined, the "SSL Handshake" continues with the exchange of the certificates: 

1. The server sends its Certificate message and, if client authentication is required, also sends a CertificateRequest message to the client. 
2. The server sends a ServerHelloDone message and waits for a client response. 
3. Upon receipt of the ServerHelloDone message, the client verifies the validity of the server's digital certificate. 

In order for the communication to be set up, a number of checks on the certificates must be passed. While discussing SSL and certificate based authentication is beyond the scope of this guide, this section will focus on the main criteria involved in ascertaining certificate validity: 
 
- Checking if the Certificate Authority (CA) is a known one (meaning one considered trusted); 
- Checking that the certificate is currently valid; 
- Checking that the name of the site and the name reported in the certificate match. 

Let's examine each check more in detail. 

- Each browser comes with a pre-loaded list of trusted CAs, against which the certificate signing CA is compared (this list can be customized and expanded at will). During the initial negotiations with an HTTPS server, if the server certificate relates to a CA unknown to the browser, a warning is usually raised. This happens most often because a web application relies on a certificate signed by a self-established CA. Whether this is to be considered a concern depends on several factors. For example, this may be fine for an Intranet environment (think of corporate web email being provided via HTTPS; here, obviously all users recognize the internal CA as a trusted CA). When a service is provided to the general public via the Internet, however (i.e. when it is important to positively verify the identity of the server we are talking to), it is usually imperative to rely on a trusted CA, one which is recognized by all the user base (and here we stop with our considerations; we won't delve deeper in the implications of the trust model being used by digital certificates). 
- Certificates have an associated period of validity, therefore they may expire. Again, we are warned by the browser about this. A public service needs a temporally valid certificate; otherwise, it means we are talking with a server whose certificate was issued by someone we trust, but has expired without being renewed. 
- What if the name on the certificate and the name of the server do not match? If this happens, it might sound suspicious. For a number of reasons, this is not so rare to see. A system may host a number of name-based virtual hosts, which share the same IP address and are identified by means of the HTTP 1.1 Host: header information. In this case, since the SSL handshake checks the server certificate before the HTTP request is processed, it is not possible to assign different certificates to each virtual server. Therefore, if the name of the site and the name reported in the certificate do not match, we have a condition which is typically signaled by the browser. To avoid this, IP-based virtual servers must be used. [33] and [34] describe techniques to deal with this problem and allow name-based virtual hosts to be correctly referenced. 

|

또 다른 취약점
-------------------------------------------------------------------------------------------

The presence of a new service, listening in a separate tcp port may introduce vulnerabilities such as infrastructure vulnerabilities if the software is not up to date [4]. Furthermore, for the correct protection of data during transmission the Session Cookie must use the Secure flag [5] and some directives should be sent to the browser to accept only secure traffic (e.g. HSTS [6], CSP). 
Also there are some attacks that can be used to intercept traffic if the web server exposes the application on both HTTP and HTTPS [6], [7] or in case of mixed HTTP and HTTPS resources in the same page. 

|

테스트 방법
==========================================================================================

|

민감한 데이터를 평문으로 전송하는 테스트
-----------------------------------------------------------------------------------------

보안되어야 하는 다양한 정보들은 평문으로 전송될 수 있습니다.
정보들이 HTTPS 대신 HTTP로 전송되는지 확인해야 합니다.

|

예제 1. HTTP를 통해 기본 인증
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


A typical example is the usage of Basic Authentication over HTTP because with Basic Authentication, after log in, credentials are encoded - and not encrypted - into HTTP Headers. 

로그인 이후 기본 인증으로 인코딩되어 있기 때문에 일반적인 예는 HTTP를 통한 기본 인증의 사용입니다.

.. code-block:: console

    $ curl -kis http://example.com/restricted/ 
    HTTP/1.1 401 Authorization Required 
    Date: Fri, 01 Aug 2013 00:00:00 GMT 
    WWW-Authenticate: Basic realm="Restricted Area" 
    Accept-Ranges: bytes 
    Vary: Accept-Encoding 
    Content-Length: 162 
    Content-Type: text/html 

    <html><head><title>401 Authorization Required</title></ 
    head> 
    <body bgcolor=white> 
    <h1>401 Authorization Required</h1> 

    Invalid login credentials! 
    </body></html> 

|    

취약한 SSL/TLS Ciphers/Protocols/Keys 테스트
-----------------------------------------------------------------------------------------

The large number of available cipher suites and quick progress in cryptanalysis makes testing an SSL server a non-trivial task. 

이러한 기준은 아래의 최소한의 체크리스트로 인식됩니다.
 
- 취약한 ciphers를 사용해서는 안됩니다. 
(예제. less than 128 bits [10]; no NULL ciphers suite, due to no encryption used; no Anonymous Diffie-Hellmann, due to not provides authentication). 

- 취약한 protocols은 비활성화 해야합니다. 
(예제. SSLv2 must be disabled, due to known weaknesses in protocol design [11]). 

- 재협상이 제대로 구성되어야 합니다.
(예제. Insecure Renegotiation must be disabled, due to MiTM attacks [12] and Client-initiated Renegotiation must be disabled, due to Denial of Service vulnerability [13]). 

- No Export (EXP) level cipher suites, due to can be easly broken [10]. 

- X.509 인증 키 길이는 강해야합니다.
(예제. if RSA or DSA is used the key must be at least 1024 bits). 

- X.509 인증은 보안 해쉬 알고리즘으로만 서명해야합니다.
(예제. not signed using MD5 hash, due to known collision attacks on this hash). 

- 키는 적절한 엔트로피로 생성되어야 합니다.
(예제. Weak Key Generated with Debian) [14]. 

더 완벽한 체크리스트는 다음과 같습니다.

- Secure Renegotiation should be enabled. 
- MD5 should not be used, due to known collision attacks. [35] 
- RC4 should not be used, due to crypto-analytical attacks [15]. 
- Server should be protected from BEAST Attack [16]. 
- Server should be protected from CRIME attack, TLS compres sion must be disabled [17]. 
- Server should support Forward Secrecy [18]. 

The following standards can be used as reference while assessing SSL servers: 

- PCI-DSS v2.0 in point 4.1 requires compliant parties to use "strong cryptography" without precisely defining key lengths and algorithms. Common interpretation, partially based on previous versions of the standard, is that at least 128 bit key cipher, no export strength algorithms and no SSLv2 should be used [19]. 
- Qualys SSL Labs Server Rating Guide [14], Depoloyment best practice [10] and SSL Threat Model [20] has been proposed to standardize SSL server assessment and configuration. But is less updated than the SSL Server tool [21]. 
- OWASP has a lot of resources about SSL/TLS Security [22], [23], [24], [25]. [26]. 

Some tools and scanners both free (e.g. SSLAudit [28] or SSLScan [29]) and commercial (e.g. Tenable Nessus [27]), can be used to assess SSL/TLS vulnerabilities. But due to evolution of these vulnerabilities a good way to test is to check them manually with openssl [30] or use the tool's output as an input for manual evaluation using the references. 

Sometimes the SSL/TLS enabled service is not directly accessible and the tester can access it only via a HTTP proxy using CONNECT method [36]. Most of the tools will try to connect to desired tcp port to start SSL/TLS handshake. This will not work since desired port is accessible only via HTTP proxy. The tester can easily circumvent this by using relaying software such as socat [37]. 

|

예제 2. nmap을 통해 SSL 서비스 인식
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

우선적으로 SSL/TLS 서비스를 하는 포트를 식별해야합니다. 
일반적으로 tcp 포트 443(https), 465(ssmtp), 585(imap4-ssl), 993(imaps), 995(ssl-pop)을 사용합니다.

이번 예제에서는 nmap에서 "-sV" 옵션으로 SSL 서비스를 찾습니다.

.. code-block:: console

    $ nmap -sV --reason -PN -n --top-ports 100 www.example.com 

    Starting Nmap 6.25 ( http://nmap.org ) at 2013-01-01 00:00 
    CEST 
    Nmap scan report for www.example.com (127.0.0.1) 
    Host is up, received user-set (0.20s latency). 
    Not shown: 89 filtered ports 
    Reason: 89 no-responses 
    PORT  STATE SERVICE  REASON  VERSION 
    21/tcp open ftp syn-ack Pure-FTPd 
    22/tcp open ssh syn-ack OpenSSH 5.3 (protocol 2.0) 
    25/tcp open smtp  syn-ack Exim smtpd 4.80 
    26/tcp open smtp  syn-ack Exim smtpd 4.80 
    80/tcp open http  syn-ack 
    110/tcp open pop3 syn-ack Dovecot pop3d 
    143/tcp open imap syn-ack Dovecot imapd 
    443/tcp open ssl/http syn-ack Apache 
    465/tcp open ssl/smtp syn-ack Exim smtpd 4.80 
    993/tcp open ssl/imap syn-ack Dovecot imapd 
    995/tcp open ssl/pop3 syn-ack Dovecot pop3d 
    Service Info: Hosts: example.com 
    Service detection performed. Please report any incorrect results 
    at http://nmap.org/submit/ . 
    Nmap done: 1 IP address (1 host up) scanned in 131.38 seconds 

|

예제 3. nmap을 통해 Ciphers, SSLv2, Certificate 정보 확인
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Nmap은 인증서 정보 체크 및 취약한 암호와 SSLv2(sl-cert,ssl-enum-ciphers)를 위한 
두가지 스크립트를 가지고 있습니다.

.. code-block:: console

    $ nmap --script ssl-cert,ssl-enum-ciphers -p 443,465,993,995 www.example.com 

    Starting Nmap 6.25 ( http://nmap.org ) at 2013-01-01 00:00 
    CEST 
    Nmap scan report for www.example.com (127.0.0.1) 
    Host is up (0.090s latency). 
    rDNS record for 127.0.0.1: www.example.com 
    PORT  STATE SERVICE 
    443/tcp open https 
    | ssl-cert: Subject: commonName=www.example.org 
    | Issuer: commonName=******* 
    | Public Key type: rsa 
    | Public Key bits: 1024 
    | Not valid before: 2010-01-23T00:00:00+00:00 
    | Not valid after:  2020-02-28T23:59:59+00:00 
    | MD5: ******* 
    |_SHA-1: ******* 
    | ssl-enum-ciphers: 
    | SSLv3: 
    | ciphers: 
    | TLS_RSA_WITH_CAMELLIA_128_CBC_SHA - strong 
    | TLS_RSA_WITH_CAMELLIA_256_CBC_SHA - strong 
    | TLS_RSA_WITH_RC4_128_SHA - strong 
    | compressors: 
    | NULL 
    | TLSv1.0: 
    | ciphers: 
    | TLS_RSA_WITH_CAMELLIA_128_CBC_SHA - strong 
    | TLS_RSA_WITH_CAMELLIA_256_CBC_SHA - strong 
    | TLS_RSA_WITH_RC4_128_SHA - strong 
    | compressors: 
    | NULL 
    |_ least strength: strong 
    465/tcp open smtps 
    | ssl-cert: Subject: commonName=*.exapmple.com 
    | Issuer: commonName=******* 
    | Public Key type: rsa 
    | Public Key bits: 2048 
    | Not valid before: 2010-01-23T00:00:00+00:00 
    | Not valid after:  2020-02-28T23:59:59+00:00 
    | MD5: ******* 
    |_SHA-1: ******* 
    | ssl-enum-ciphers: 
    | SSLv3: 
    | ciphers: 
    | TLS_RSA_WITH_CAMELLIA_128_CBC_SHA - strong 
    | TLS_RSA_WITH_CAMELLIA_256_CBC_SHA - strong 
    | TLS_RSA_WITH_RC4_128_SHA - strong 
    | compressors: 
    | NULL 
    | TLSv1.0: 
    | ciphers: 
    | TLS_RSA_WITH_CAMELLIA_128_CBC_SHA - strong | TLS_RSA_WITH_CAMELLIA_256_CBC_SHA - strong | TLS_RSA_WITH_RC4_128_SHA - strong | compressors: | NULL |_ least strength: strong 993/tcp open imaps | ssl-cert: Subject: commonName=*.exapmple.com | Issuer: commonName=******* | Public Key type: rsa | Public Key bits: 2048 | Not valid before: 2010-01-23T00:00:00+00:00 | Not valid after:  2020-02-28T23:59:59+00:00 | MD5: ******* |_SHA-1: ******* | ssl-enum-ciphers: | SSLv3: | ciphers: | TLS_RSA_WITH_CAMELLIA_128_CBC_SHA - strong | TLS_RSA_WITH_CAMELLIA_256_CBC_SHA - strong | TLS_RSA_WITH_RC4_128_SHA - strong | compressors: | NULL | TLSv1.0: | ciphers: | TLS_RSA_WITH_CAMELLIA_128_CBC_SHA - strong | TLS_RSA_WITH_CAMELLIA_256_CBC_SHA - strong | TLS_RSA_WITH_RC4_128_SHA - strong | compressors: | NULL |_ least strength: strong 995/tcp open pop3s | ssl-cert: Subject: commonName=*.exapmple.com | Issuer: commonName=******* | Public Key type: rsa | Public Key bits: 2048 | Not valid before: 2010-01-23T00:00:00+00:00 | Not valid after:  2020-02-28T23:59:59+00:00 | MD5: ******* |_SHA-1: ******* | ssl-enum-ciphers: | SSLv3: | ciphers: | TLS_RSA_WITH_CAMELLIA_128_CBC_SHA - strong | TLS_RSA_WITH_CAMELLIA_256_CBC_SHA - strong | TLS_RSA_WITH_RC4_128_SHA - strong | compressors: | NULL | TLSv1.0: | ciphers: | TLS_RSA_WITH_CAMELLIA_128_CBC_SHA - strong | TLS_RSA_WITH_CAMELLIA_256_CBC_SHA - strong | TLS_RSA_WITH_RC4_128_SHA - strong | compressors: | NULL |_ least strength: strong Nmap done: 1 IP address (1 host up) scanned in 8.64 seconds 

|

예제 4. openssl을 통해 Client-initiated Renegotiation과 Secure Renegotiation 테스트
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Openssl은 수동으로 SSL/TLS를 테스트할 수 있습니다. 이번 예제에서 테스터는 openssl으로 서버에 연결된
클라이언트에 재협상을 초기화하려 합니다.
HTTP 요청의 첫 라인을 쓰고나서, 새로운 라인에 "R"을 입력합니다.
그리고나서 재협상을 기다리고 완벽한 HTTP 요청과 보안 재협상이 서버 출력으로 지원되는지 확인합니다.

Using manual requests it is also possible to see if Compression is enabled for TLS and to check for CRIME [13], for ciphers and for other vulnerabilities. 

.. code-block:: console

    $ openssl s_client -connect www2.example.com:443 
    CONNECTED(00000003) 
    depth=2 ****** 
    verify error:num=20:unable to get local issuer certificate 
    verify return:0 
    Certificate chain
     0 s:******
     i:******
     1 s:******
     i:******
     2 s:******
     i:****** 
    Server certificate 
    -----BEGIN CERTIFICATE----
    ****** 
    -----END CERTIFICATE----
    subject=****** 
    issuer=****** 
    No client certificate CA names sent 
    SSL handshake has read 3558 bytes and written 640 bytes 
    New, TLSv1/SSLv3, Cipher is DES-CBC3-SHA 
    Server public key is 2048 bit 
    Secure Renegotiation IS NOT supported 
    Compression: NONE 
    Expansion: NONE 
    SSL-Session:
        Protocol  : TLSv1
     Cipher : DES-CBC3-SHA
     Session-ID: ******
     Session-ID-ctx: 
        Master-Key: ******
        Key-Arg  : None
        PSK identity: None
        PSK identity hint: None
        SRP username: None
        Start Time: ******
     Timeout : 300 (sec)
        Verify return code: 20 (unable to get local issuer certificate) 


이제 테스터는 아래와 같이 HTTP 요청 첫 줄과 새로운 줄에 R을 입력합니다.

.. code-block:: console

    HEAD / HTTP/1.1 
    R 

서버는 renegotiating 됩니다.

.. code-block:: console

    RENEGOTIATING 
    depth=2 C****** 
    verify error:num=20:unable to get local issuer certificate 
    verify return:0 

그리고 테스터는 완벽한 요청을 하여 응답을 체크할 수 있습니다.
만약 HEAD가 지원되지 않더라도, Client-intiated renegotiation은 지원됩니다.

.. code-block:: console

    HEAD / HTTP/1.1 
    
    HTTP/1.1 403 Forbidden ( The server denies the specified Uni
    form Resource Locator (URL). Contact the server administrator.  ) 
    Connection: close 
    Pragma: no-cache 
    Cache-Control: no-cache 
    Content-Type: text/html 
    Content-Length: 1792 

    read:errno=0 

|

예제 5. TestSSLServer를 통해 암호화 방식, BEAST, CRIME 공격 테스트
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

TestSSLServer는 암호화 방식과 BEAST, CRIME 공격을 확인하기 위해 테스터가 사용할 수 있는 스크립트입니다.

BEAST(Browser Exploit Against SSL/TLS)은 TLS 1.0에 CBC 취약점을 공격합니다.

CRIME (Compression Ratio Info-leak Made Easy)은 TLS 압축 취약점을 공격하고, 그것을 비활성화합니다.
재밌는 것은 BEAST를 방어할 수 있는 것은 RC4를 사용하는 것이지만, RC4에서는 암호화 분석 공격 때문에 
권장하지 않습니다.

이 공격을 확인하기 위한 온라인 툴은 SSL 연구소에 있지만, 인터넷과 연결된 서버에만 이용할 수 있습니다.
또한, 확인 결과가 SSL 연구소 서버에 저장될 것이기에 온라인 툴 사용은 고려되어야 합니다.

.. code-block:: console

    $ java -jar TestSSLServer.jar www3.example.com 443 
    Supported versions: SSLv3 TLSv1.0 TLSv1.1 TLSv1.2 
    Deflate compression: no 
    Supported cipher suites (ORDER IS NOT SIGNIFICANT):

      SSLv3
         RSA_WITH_RC4_128_SHA
         RSA_WITH_3DES_EDE_CBC_SHA
         DHE_RSA_WITH_3DES_EDE_CBC_SHA
         RSA_WITH_AES_128_CBC_SHA
         DHE_RSA_WITH_AES_128_CBC_SHA 



         RSA_WITH_AES_256_CBC_SHA
         DHE_RSA_WITH_AES_256_CBC_SHA
         RSA_WITH_CAMELLIA_128_CBC_SHA
         DHE_RSA_WITH_CAMELLIA_128_CBC_SHA
         RSA_WITH_CAMELLIA_256_CBC_SHA
         DHE_RSA_WITH_CAMELLIA_256_CBC_SHA
         TLS_RSA_WITH_SEED_CBC_SHA
         TLS_DHE_RSA_WITH_SEED_CBC_SHA

      (TLSv1.0: idem)
      (TLSv1.1: idem)
      TLSv1.2

         RSA_WITH_RC4_128_SHA
         RSA_WITH_3DES_EDE_CBC_SHA
         DHE_RSA_WITH_3DES_EDE_CBC_SHA
         RSA_WITH_AES_128_CBC_SHA
         DHE_RSA_WITH_AES_128_CBC_SHA
         RSA_WITH_AES_256_CBC_SHA
         DHE_RSA_WITH_AES_256_CBC_SHA
         RSA_WITH_AES_128_CBC_SHA256
         RSA_WITH_AES_256_CBC_SHA256
         RSA_WITH_CAMELLIA_128_CBC_SHA
         DHE_RSA_WITH_CAMELLIA_128_CBC_SHA
         DHE_RSA_WITH_AES_128_CBC_SHA256
         DHE_RSA_WITH_AES_256_CBC_SHA256
         RSA_WITH_CAMELLIA_256_CBC_SHA
         DHE_RSA_WITH_CAMELLIA_256_CBC_SHA
         TLS_RSA_WITH_SEED_CBC_SHA
         TLS_DHE_RSA_WITH_SEED_CBC_SHA
         TLS_RSA_WITH_AES_128_GCM_SHA256
         TLS_RSA_WITH_AES_256_GCM_SHA384
         TLS_DHE_RSA_WITH_AES_128_GCM_SHA256
         TLS_DHE_RSA_WITH_AES_256_GCM_SHA384 
    ----------------------
    Server certificate(s):
     ****** 
    ----------------------
    Minimal encryption strength:  strong encryption (96-bit or 
    more) 
    Achievable encryption strength:  strong encryption (96-bit or 
    more) 
    BEAST status: vulnerable 
    CRIME status: protected 

|

예제 6. sslyze 테스트
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Sslyze는 대량 스캐닝과 XML 출력을 수행할 수 있는 파이썬 스크립트입니다.
일반적인 스캔 에제는 아래와 같습니다.

.. code-block:: console

    ./sslyze.py --regular example.com:443
     REGISTERING AVAILABLE PLUGINS
     ----------------------------PluginHSTS
      PluginSessionRenegotiation
      PluginCertInfo
      PluginSessionResumption
      PluginOpenSSLCipherSuites
      PluginCompression
     CHECKING HOST(S) AVAILABILITY
     ----------------------------
      example.com:443  => 127.0.0.1:443
     SCAN RESULTS FOR EXAMPLE.COM:443 - 127.0.0.1:443 --------------------------------------------------
    *
     Compression :
            Compression Support:  Disabled


     *
     Session Renegotiation :
          Client-initiated Renegotiations:  Rejected
          Secure Renegotiation:  Supported


     *
     Certificate :      Validation w/ Mozilla's CA Store:  Certificate is NOT Trust


    ed: unable to get local issuer certificate      Hostname Validation:  MISMATCH                                 SHA1 Fingerprint:  ******
          Common Name:  www.example.com 
    Issuer: ******
     Serial Number: **** 
          Not Before:  Sep 26 00:00:00 2010 GMT 
          Not After:  Sep 26 23:59:59 2020 GMT 
          Signature Algorithm:  sha1WithRSAEncryption 
          Key Size:  1024 bit 
          X509v3 Subject Alternative Name:  {'othername': ['<unsupported>'], 'DNS': ['www.example.com']}
     *
     OCSP Stapling : 
    Server did not send back an OCSP response.                                   


    *
     Session Resumption : With Session IDs: Supported (5 successful, 0 failed, 


    0 errors, 5 total attempts).      With TLS Session Tickets:  Supported
     * SSLV2 Cipher Suites :
          Rejected Cipher Suite(s): Hidden       Preferred Cipher Suite: None           Accepted Cipher Suite(s): None         Undefined - An unexpected error happened: None 


    * SSLV3 Cipher Suites :
          Rejected Cipher Suite(s): Hidden 
          Preferred Cipher Suite:                  RC4-SHA  128 bits HTTP 200 OK 
          Accepted Cipher Suite(s):                CAMELLIA256-SHA  256 bits HTTP 200 OK         RC4-SHA  128 bits HTTP 200 OK         CAMELLIA128-SHA  128 bits HTTP 200 OK 
          Undefined - An unexpected error happened: None 
    * TLSV1_1 Cipher Suites :      Rejected Cipher Suite(s): Hidden       Preferred Cipher Suite: None           Accepted Cipher Suite(s): None         Undefined - An unexpected error happened: 
            ECDH-RSA-AES256-SHA  out         ECDH-ECDSA-AES256-SHA  out  socket.timeout - timed socket.timeout - timed  
      * TLSV1_2 Cipher Suites : 

          Rejected Cipher Suite(s): Hidden 
          Preferred Cipher Suite: None     
          Accepted Cipher Suite(s): None   
          Undefined - An unexpected error happened:         ECDH-RSA-AES256-GCM-SHA384  socket.timeout - timed out         ECDH-ECDSA-AES256-GCM-SHA384  socket.timeout 
    -timed out 
    * TLSV1 Cipher Suites :      Rejected Cipher Suite(s): Hidden       Preferred Cipher Suite:          
            RC4-SHA  128 bits Timeout on HTTP GET 
          Accepted Cipher Suite(s):                CAMELLIA256-SHA  256 bits HTTP 200 OK         RC4-SHA  128 bits HTTP 200 OK         CAMELLIA128-SHA  128 bits HTTP 200 OK 
          Undefined - An unexpected error happened:         ADH-CAMELLIA256-SHA  socket.timeout - timed out 
     SCAN COMPLETED IN 9.68 S
     -----------------------

|


예제 7. testssl.sh 테스트
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Testssl.sh는 리눅스 쉘 스크립트로, 웹 서버 확인 뿐만 아니라, 다른 포트 서비스 및 STARTTLS, SNI, SPDY 그리고 HTTP 헤더에 대한 몇가지 검사를 지원합니다.

아래 간단한 출력 예제입니다.

.. code-block:: console

    user@myhost: % testssl.sh owasp.org      

    ############################################# 
    ########### 
    testssl.sh v2.0rc3  (https://testssl.sh) 
    ($Id: testssl.sh,v 1.97 2014/04/15 21:54:29 dirkw Exp $)

       This program is free software. Redistribution + 
       modification under GPLv2 is permitted. 
       USAGE w/o ANY WARRANTY. USE IT AT YOUR OWN RISK!

     Note you can only check the server against what is available (ciphers/protocols) locally on your machine ############################################# ########### 
    Using "OpenSSL 1.0.2-beta1 24 Feb 2014" on
          "myhost:/<mypath>/bin/openssl64" 

    Testing now (2014-04-17 15:06) ---> owasp.org:443 <--("owasp.org" resolves to "192.237.166.62 / 2001:4801:7821:77:cd2c:d9de:ff10:170e") 
    --> Testing Protocols
     SSLv2  NOT offered (ok) 
     SSLv3  offered 



     TLSv1  offered (ok) 
     TLSv1.1  offered (ok) 
     TLSv1.2  offered (ok) 

     SPDY/NPN  not offered 

    --> Testing standard cipher lists

     Null Cipher NOT offered (ok) 
     Anonymous NULL Cipher  NOT offered (ok) 
     Anonymous DH Cipher  NOT offered (ok) 
     40 Bit encryption  NOT offered (ok) 
     56 Bit encryption  NOT offered (ok) 
    Export Cipher (general) NOT offered (ok) 
     Low (<=64 Bit)  NOT offered (ok) 
     DES Cipher  NOT offered (ok) 
     Triple DES Cipher  offered
     Medium grade encryption  offered
     High grade encryption  offered (ok) 

    --> Testing server defaults (Server Hello)

     Negotiated protocol  TLSv1.2 
     Negotiated cipher  AES128-GCM-SHA256 

     Server key size  2048 bit
     TLS server extensions:  server name, renegotiation info, 
    session ticket, heartbeat
     Session Tickets RFC 5077  300 seconds 

    --> Testing specific vulnerabilities

     Heartbleed (CVE-2014-0160), experimental  NOT vulnerable 
    (ok) 
     Renegotiation (CVE 2009-3555)  NOT vulnerable (ok) 
     CRIME, TLS (CVE-2012-4929)  NOT vulnerable (ok)  

    --> Checking RC4 Ciphers 

    RC4 seems generally available. Now testing specific ciphers...

     Hexcode  Cipher Name KeyExch.  Encryption Bits 

    [0x05] RC4-SHA  RSA  RC4 128 
    RC4 is kind of broken, for e.g. IE6 consider 0x13 or 0x0a 
    --> Testing HTTP Header response 
    HSTS no  Server  Apache Application (None) --> Testing (Perfect) Forward Secrecy  (P)FS) 
    no PFS available 
    Done now (2014-04-17 15:07) ---> owasp.org:443 <--
    user@myhost: %    

STARTTLS는 testssl.sh -t smtp.gmail.com:587 smtp를 통해 테스트할 수 있고,
각 암호문은 testssl -e <target>로,
각 프로토콜별 암호문은 testssl -E <target>으로 테스트할 수 있습니다.
openssl을 통해 설치된 암호문이 어떤 것이 있는지 보기 위해서는 testssl -V 명령 입력 

For a thorough check it is best to dump the supplied OpenSSL binaries in the path or the one of testssl.sh. 
The interesting thing is if a tester looks at the sources they learn how features are tested, see e.g. 

Example 4. What is even better is that it does the whole handshake for heartbleed in pure /bin/bash with /dev/tcp sockets -- no piggyback perl/python/you name it. 
Additionally it provides a prototype (via "testssl.sh -V") of mapping to RFC cipher suite names to OpenSSL ones. 
The tester needs the file mapping-rfc.txt in same directory. 

|

예제 8. SSL Breacher 테스트
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

이 툴은 여러 가지 다른 툴들 중 가장 포괄적인 SSL 테스트를 보완한 툴입니다. 
다음과 같은 검사를 지원합니다.

- HeartBleed 
- ChangeCipherSpec 인젝션
- BREACH 
- BEAST 
- Forward Secrecy support 
- RC4 support 
- CRIME & TIME (If CRIME is detected, TIME will also be reported) 
- Lucky13 
- HSTS: Check for implementation of HSTS header 
- HSTS: Reasonable duration of MAX-AGE 
- HSTS: Check for SubDomains support 
- Certificate expiration 
- Insufficient public key-length 
- Host-name mismatch 
- Weak Insecure Hashing Algorithm (MD2, MD4, MD5) 
- SSLv2 support 
- Weak ciphers check 
- Null Prefix in certificate 
- HTTPS Stripping 
- Surf Jacking 
- Non-SSL elements/contents embedded in SSL page 
- Cache-Control

.. code-block:: console

    pentester@r00ting: % breacher.sh https://localhost/login.php 

    Host Info: 
    ============== 
    Host : localhost 
    Port : 443 
    Path : /login.php 

    Certificate Info: 
    ================== 
    Type: Domain Validation Certificate (i.e. NON-Extended Validation Certificate) 
    Expiration Date: Sat Nov 09 07:48:47 SGT 2019 
    Signature Hash Algorithm: SHA1withRSA 
    Public key: Sun RSA public key, 1024 bits

     modulus: 13563296484355500991016409816100408625 
    9135236815846778903941582882908611097021488277 
    5657328517128950572278496563648868981962399018 
    7956963565986177085092024117822268667016231814 
    7175328086853962427921575656093414000691131757 
    0996633223696567560900301903699230503066687785 
    34926124693591013220754558036175189121517

      public exponent: 65537 
    Signed for: CN=localhost 
    Signed by: CN=localhost 
    Total certificate chain: 1 

    (Use -Djavax.net.debug=ssl:handshake:verbose for debugged 
    output.) 

    ===================================== 

    Certificate Validation: 
    =============================== 
    [!] Signed using Insufficient public key length 1024 bits

        (Refer to http://www.keylength.com/ for details) [!] Certificate Signer: Self-signed/Untrusted CA  - verified with 
    Firefox & Java ROOT CAs. 
    ===================================== 
    Loading module: Hut3 Cardiac Arrest ... 
    Checking localhost:443 for Heartbleed bug (CVE-2014-0160) ... 
    [-] Connecting to 127.0.0.1:443 using SSLv3 [-] Sending ClientHello [-] ServerHello received [-] Sending Heartbeat [Vulnerable] Heartbeat response was 16384 bytes instead of 3! 127.0.0.1:443 is vulnerable over SSLv3 [-] Displaying response (lines consisting entirely of null bytes are removed):
     0000: 02 FF FF 08 03 00 53 48 73 F0 7C CA C1 D9 02 04 ...... SHs.|.....
     0010: F2 1D 2D 49 F5 12 BF 40 1B 94 D9 93 E4 C4 F4 F0 ..I...@........
     0020: D0 42 CD 44 A2 59 00 02 96 00 00 00 01 00 02 00 .B.D.Y..........
     0060: 1B 00 1C 00 1D 00 1E 00 1F 00 20 00 21 00 22 00 .......... .!.".
     0070: 23 00 24 00 25 00 26 00 27 00 28 00 29 00 2A 00 #.$.%.&.'.(.).*.
     0080: 2B 00 2C 00 2D 00 2E 00 2F 00 30 00 31 00 32 00 +.,..../.0.1.2.
     0090: 33 00 34 00 35 00 36 00 37 00 38 00 39 00 3A 00 3.4.5.6.7.8.9.:.
     00a0: 3B 00 3C 00 3D 00 3E 00 3F 00 40 00 41 00 42 00 ;.<.=.>.?.@.A.B.
     00b0: 43 00 44 00 45 00 46 00 60 00 61 00 62 00 63 00 C.D.E.F.`.a.b.c.
     00c0: 64 00 65 00 66 00 67 00 68 00 69 00 6A 00 6B 00 
    d.e.f.g.h.i.j.k. 00d0: 6C 00 6D 00 80 00 81 00 82 00 83 00 84 00 85 00 
    l.m............. 01a0: 20 C0 21 C0 22 C0 23 C0 24 C0 25 C0 26 C0 27 C0 
    .!.".#.$.%.&.'.
     01b0: 28 C0 29 C0 2A C0 2B C0 2C C0 2D C0 2E C0 2F C0 (.).*.+.,.-.../.
     01c0: 30 C0 31 C0 32 C0 33 C0 34 C0 35 C0 36 C0 37 C0 
    0.1.2.3.4.5.6.7. 01d0: 38 C0 39 C0 3A C0 3B C0 3C C0 3D C0 3E C0 3F C0 8.9.:.;.<.=.>.?. 01e0: 40 C0 41 C0 42 C0 43 C0 44 C0 45 C0 46 C0 47 C0 
    @.A.B.C.D.E.F.G. 01f0: 48 C0 49 C0 4A C0 4B C0 4C C0 4D C0 4E C0 4F C0 
    H.I.J.K.L.M.N.O. 0200: 50 C0 51 C0 52 C0 53 C0 54 C0 55 C0 56 C0 57 C0 
    P.Q.R.S.T.U.V.W. 0210: 58 C0 59 C0 5A C0 5B C0 5C C0 5D C0 5E C0 5F C0 X.Y.Z.[.\.].^._. 0220: 60 C0 61 C0 62 C0 63 C0 64 C0 65 C0 66 C0 67 C0 
    `.a.b.c.d.e.f.g. 0230: 68 C0 69 C0 6A C0 6B C0 6C C0 6D C0 6E C0 6F C0 
    h.i.j.k.l.m.n.o. 0240: 70 C0 71 C0 72 C0 73 C0 74 C0 75 C0 76 C0 77 C0 
    p.q.r.s.t.u.v.w. 0250: 78 C0 79 C0 7A C0 7B C0 7C C0 7D C0 7E C0 7F C0 x.y.z.{.|.}.~... 02c0: 00 00 49 00 0B 00 04 03 00 01 02 00 0A 00 34 00 ..I...........4. 02d0: 32 00 0E 00 0D 00 19 00 0B 00 0C 00 18 00 09 00 2............... 0300: 10 00 11 00 23 00 00 00 0F 00 01 01 00 00 00 00 ....#...........
     0bd0: 00 00 00 00 00 00 00 00 00 12 7D 01 00 10 00 02 ..........}..... 
    [-] Closing connection 
    [-] Connecting to 127.0.0.1:443 using TLSv1.0 [-] Sending ClientHello [-] ServerHello received [-] Sending Heartbeat [Vulnerable] Heartbeat response was 16384 bytes instead of 3! 


    127.0.0.1:443 is vulnerable over TLSv1.0 [-] Displaying response (lines consisting entirely of null bytes are removed):
     0000: 02 FF FF 08 03 01 53 48 73 F0 7C CA C1 D9 02 04 ...... SHs.|.....
     0010: F2 1D 2D 49 F5 12 BF 40 1B 94 D9 93 E4 C4 F4 F0 ..I...@........
     0020: D0 42 CD 44 A2 59 00 02 96 00 00 00 01 00 02 00 .B.D.Y..........
     0060: 1B 00 1C 00 1D 00 1E 00 1F 00 20 00 21 00 22 00 .......... .!.".
     0070: 23 00 24 00 25 00 26 00 27 00 28 00 29 00 2A 00 #.$.%.&.'.(.).*.
     0080: 2B 00 2C 00 2D 00 2E 00 2F 00 30 00 31 00 32 00 +.,..../.0.1.2.
     0090: 33 00 34 00 35 00 36 00 37 00 38 00 39 00 3A 00 3.4.5.6.7.8.9.:.
     00a0: 3B 00 3C 00 3D 00 3E 00 3F 00 40 00 41 00 42 00 ;.<.=.>.?.@.A.B.
     00b0: 43 00 44 00 45 00 46 00 60 00 61 00 62 00 63 00 C.D.E.F.`.a.b.c.
     00c0: 64 00 65 00 66 00 67 00 68 00 69 00 6A 00 6B 00 
    d.e.f.g.h.i.j.k. 00d0: 6C 00 6D 00 80 00 81 00 82 00 83 00 84 00 85 00 
    l.m............. 01a0: 20 C0 21 C0 22 C0 23 C0 24 C0 25 C0 26 C0 27 C0 
    .!.".#.$.%.&.'.
     01b0: 28 C0 29 C0 2A C0 2B C0 2C C0 2D C0 2E C0 2F C0 (.).*.+.,.-.../.
     01c0: 30 C0 31 C0 32 C0 33 C0 34 C0 35 C0 36 C0 37 C0 
    0.1.2.3.4.5.6.7. 01d0: 38 C0 39 C0 3A C0 3B C0 3C C0 3D C0 3E C0 3F C0 8.9.:.;.<.=.>.?. 01e0: 40 C0 41 C0 42 C0 43 C0 44 C0 45 C0 46 C0 47 C0 
    @.A.B.C.D.E.F.G. 01f0: 48 C0 49 C0 4A C0 4B C0 4C C0 4D C0 4E C0 4F C0 
    H.I.J.K.L.M.N.O. 0200: 50 C0 51 C0 52 C0 53 C0 54 C0 55 C0 56 C0 57 C0 
    P.Q.R.S.T.U.V.W. 0210: 58 C0 59 C0 5A C0 5B C0 5C C0 5D C0 5E C0 5F C0 X.Y.Z.[.\.].^._. 0220: 60 C0 61 C0 62 C0 63 C0 64 C0 65 C0 66 C0 67 C0 
    `.a.b.c.d.e.f.g. 0230: 68 C0 69 C0 6A C0 6B C0 6C C0 6D C0 6E C0 6F C0 
    h.i.j.k.l.m.n.o. 0240: 70 C0 71 C0 72 C0 73 C0 74 C0 75 C0 76 C0 77 C0 
    p.q.r.s.t.u.v.w. 0250: 78 C0 79 C0 7A C0 7B C0 7C C0 7D C0 7E C0 7F C0 x.y.z.{.|.}.~... 02c0: 00 00 49 00 0B 00 04 03 00 01 02 00 0A 00 34 00 ..I...........4. 02d0: 32 00 0E 00 0D 00 19 00 0B 00 0C 00 18 00 09 00 2...............
     0300: 10 00 11 00 23 00 00 00 0F 00 01 01 00 00 00 00 ....#...........
     0bd0: 00 00 00 00 00 00 00 00 00 12 7D 01 00 10 00 02 ..........}..... 
    [-] Closing connection 
    [-] Connecting to 127.0.0.1:443 using TLSv1.1 [-] Sending ClientHello [-] ServerHello received [-] Sending Heartbeat [Vulnerable] Heartbeat response was 16384 bytes instead of 3! 
    127.0.0.1:443 is vulnerable over TLSv1.1 [-] Displaying response (lines consisting entirely of null bytes are removed):
     0000: 02 FF FF 08 03 02 53 48 73 F0 7C CA C1 D9 02 04 ...... SHs.|.....
     0010: F2 1D 2D 49 F5 12 BF 40 1B 94 D9 93 E4 C4 F4 F0 ..I...@........
     0020: D0 42 CD 44 A2 59 00 02 96 00 00 00 01 00 02 00 .B.D.Y..........
     0060: 1B 00 1C 00 1D 00 1E 00 1F 00 20 00 21 00 22 00 .......... .!.".
     0070: 23 00 24 00 25 00 26 00 27 00 28 00 29 00 2A 00 #.$.%.&.'.(.).*.
     0080: 2B 00 2C 00 2D 00 2E 00 2F 00 30 00 31 00 32 00 +.,..../.0.1.2.
     0090: 33 00 34 00 35 00 36 00 37 00 38 00 39 00 3A 00 3.4.5.6.7.8.9.:.
     00a0: 3B 00 3C 00 3D 00 3E 00 3F 00 40 00 41 00 42 00 ;.<.=.>.?.@.A.B.
     00b0: 43 00 44 00 45 00 46 00 60 00 61 00 62 00 63 00 C.D.E.F.`.a.b.c.
     00c0: 64 00 65 00 66 00 67 00 68 00 69 00 6A 00 6B 00 
    d.e.f.g.h.i.j.k. 00d0: 6C 00 6D 00 80 00 81 00 82 00 83 00 84 00 85 00 
    l.m............. 01a0: 20 C0 21 C0 22 C0 23 C0 24 C0 25 C0 26 C0 27 C0 
    .!.".#.$.%.&.'.
     01b0: 28 C0 29 C0 2A C0 2B C0 2C C0 2D C0 2E C0 2F C0 (.).*.+.,.-.../.
     01c0: 30 C0 31 C0 32 C0 33 C0 34 C0 35 C0 36 C0 37 C0 
    0.1.2.3.4.5.6.7. 01d0: 38 C0 39 C0 3A C0 3B C0 3C C0 3D C0 3E C0 3F C0 8.9.:.;.<.=.>.?. 01e0: 40 C0 41 C0 42 C0 43 C0 44 C0 45 C0 46 C0 47 C0 
    @.A.B.C.D.E.F.G. 01f0: 48 C0 49 C0 4A C0 4B C0 4C C0 4D C0 4E C0 4F C0 
    H.I.J.K.L.M.N.O. 0200: 50 C0 51 C0 52 C0 53 C0 54 C0 55 C0 56 C0 57 C0 
    P.Q.R.S.T.U.V.W. 0210: 58 C0 59 C0 5A C0 5B C0 5C C0 5D C0 5E C0 5F C0 X.Y.Z.[.\.].^._. 0220: 60 C0 61 C0 62 C0 63 C0 64 C0 65 C0 66 C0 67 C0 
    `.a.b.c.d.e.f.g. 0230: 68 C0 69 C0 6A C0 6B C0 6C C0 6D C0 6E C0 6F C0 
    h.i.j.k.l.m.n.o. 0240: 70 C0 71 C0 72 C0 73 C0 74 C0 75 C0 76 C0 77 C0 
    p.q.r.s.t.u.v.w. 0250: 78 C0 79 C0 7A C0 7B C0 7C C0 7D C0 7E C0 7F C0 x.y.z.{.|.}.~...


     02c0: 00 00 49 00 0B 00 04 03 00 01 02 00 0A 00 34 00 ..I...........4.
     02d0: 32 00 0E 00 0D 00 19 00 0B 00 0C 00 18 00 09 00 2...............
     0300: 10 00 11 00 23 00 00 00 0F 00 01 01 00 00 00 00 ....#...........
     0bd0: 00 00 00 00 00 00 00 00 00 12 7D 01 00 10 00 02 ..........}..... 
    [-] Closing connection 
    [-] Connecting to 127.0.0.1:443 using TLSv1.2 [-] Sending ClientHello [-] ServerHello received [-] Sending Heartbeat [Vulnerable] Heartbeat response was 16384 bytes instead of 3! 
    127.0.0.1:443 is vulnerable over TLSv1.2 [-] Displaying response (lines consisting entirely of null bytes are removed):
     0000: 02 FF FF 08 03 03 53 48 73 F0 7C CA C1 D9 02 04 ...... SHs.|.....
     0010: F2 1D 2D 49 F5 12 BF 40 1B 94 D9 93 E4 C4 F4 F0 ..I...@........
     0020: D0 42 CD 44 A2 59 00 02 96 00 00 00 01 00 02 00 .B.D.Y..........
     0060: 1B 00 1C 00 1D 00 1E 00 1F 00 20 00 21 00 22 00 .......... .!.".
     0070: 23 00 24 00 25 00 26 00 27 00 28 00 29 00 2A 00 #.$.%.&.'.(.).*.
     0080: 2B 00 2C 00 2D 00 2E 00 2F 00 30 00 31 00 32 00 +.,..../.0.1.2.
     0090: 33 00 34 00 35 00 36 00 37 00 38 00 39 00 3A 00 3.4.5.6.7.8.9.:.
     00a0: 3B 00 3C 00 3D 00 3E 00 3F 00 40 00 41 00 42 00 ;.<.=.>.?.@.A.B.
     00b0: 43 00 44 00 45 00 46 00 60 00 61 00 62 00 63 00 C.D.E.F.`.a.b.c.
     00c0: 64 00 65 00 66 00 67 00 68 00 69 00 6A 00 6B 00 
    d.e.f.g.h.i.j.k. 00d0: 6C 00 6D 00 80 00 81 00 82 00 83 00 84 00 85 00 
    l.m............. 01a0: 20 C0 21 C0 22 C0 23 C0 24 C0 25 C0 26 C0 27 C0 
    .!.".#.$.%.&.'.
     01b0: 28 C0 29 C0 2A C0 2B C0 2C C0 2D C0 2E C0 2F C0 (.).*.+.,.-.../.
     01c0: 30 C0 31 C0 32 C0 33 C0 34 C0 35 C0 36 C0 37 C0 
    0.1.2.3.4.5.6.7. 01d0: 38 C0 39 C0 3A C0 3B C0 3C C0 3D C0 3E C0 3F C0 8.9.:.;.<.=.>.?. 01e0: 40 C0 41 C0 42 C0 43 C0 44 C0 45 C0 46 C0 47 C0 
    @.A.B.C.D.E.F.G. 01f0: 48 C0 49 C0 4A C0 4B C0 4C C0 4D C0 4E C0 4F C0 
    H.I.J.K.L.M.N.O. 0200: 50 C0 51 C0 52 C0 53 C0 54 C0 55 C0 56 C0 57 C0 
    P.Q.R.S.T.U.V.W.
     0210: 58 C0 59 C0 5A C0 5B C0 5C C0 5D C0 5E C0 5F C0 X.Y.Z.[.\.].^._.
     0220: 60 C0 61 C0 62 C0 63 C0 64 C0 65 C0 66 C0 67 C0 `.a.b.c.d.e.f.g.
     0230: 68 C0 69 C0 6A C0 6B C0 6C C0 6D C0 6E C0 6F C0 
    h.i.j.k.l.m.n.o. 0240: 70 C0 71 C0 72 C0 73 C0 74 C0 75 C0 76 C0 77 C0 
    p.q.r.s.t.u.v.w. 0250: 78 C0 79 C0 7A C0 7B C0 7C C0 7D C0 7E C0 7F C0 x.y.z.{.|.}.~... 02c0: 00 00 49 00 0B 00 04 03 00 01 02 00 0A 00 34 00 ..I...........4. 02d0: 32 00 0E 00 0D 00 19 00 0B 00 0C 00 18 00 09 00 2............... 0300: 10 00 11 00 23 00 00 00 0F 00 01 01 00 00 00 00 ....#...........
     0bd0: 00 00 00 00 00 00 00 00 00 12 7D 01 00 10 00 02 ..........}..... 
    [-] Closing connection 

    [!] Vulnerable to Heartbleed bug (CVE-2014-0160) mentioned in 
    http://heartbleed.com/ 
    [!] Vulnerability Status: VULNERABLE 

    ===================================== 

    Loading module: CCS Injection script by TripWire VERT ... 

    Checking localhost:443 for OpenSSL ChangeCipherSpec (CCS) 
    Injection bug (CVE-2014-0224) ... 

    [!] The target may allow early CCS on TLSv1.2 
    [!] The target may allow early CCS on TLSv1.1 
    [!] The target may allow early CCS on TLSv1 
    [!] The target may allow early CCS on SSLv3 

    [-] This is an experimental detection script and does not definitively determine vulnerable server status. 

    [!] Potentially vulnerable to OpenSSL ChangeCipherSpec (CCS) 
    Injection vulnerability (CVE-2014-0224) mentioned in http:// 
    ccsinjection.lepidum.co.jp/ 
    [!] Vulnerability Status: Possible 

    ===================================== 

    Checking localhost:443 for HTTP Compression support against 
    BREACH vulnerability (CVE-2013-3587) ... 

    [*] HTTP Compression: DISABLED 
    [*] Immune from BREACH attack mentioned in https://media. 
    blackhat.com/us-13/US-13-Prado-SSL-Gone-in-30-secondsA-BREACH-beyond-CRIME-WP.pdf 
    [*] Vulnerability Status: No 



    --------------- RAW HTTP RESPONSE --------------
    HTTP/1.1 200 OK Date: Wed, 23 Jul 2014 13:48:07 GMT Server: Apache/2.4.3 (Win32) OpenSSL/1.0.1c PHP/5.4.7 X-Powered-By: PHP/5.4.7 Set-Cookie: SessionID=xxx; expires=Wed, 23-Jul-2014 12:48:07 GMT; path=/; secure Set-Cookie: SessionChallenge=yyy; expires=Wed, 23-Jul-2014 
    12:48:07 GMT; path=/ Content-Length: 193 Connection: close Content-Type: text/html 
    <html> 
    <head> 
    <title>Login page </title> 
    </head> 
    <body> 
    <script src="http://othersite/test.js"></script> 

    <link rel="stylesheet" type="text/css" href="http://somesite/ 
    test.css"> 

    ===================================== 

    Checking localhost:443 for correct use of Strict Transport Security (STS) response header (RFC6797) ... 

    [!] STS response header: NOT PRESENT 
    [!] Vulnerable to MITM threats mentioned in https://www.owasp. 
    org/index.php/HTTP_Strict_Transport_Security#Threats 
    [!] Vulnerability Status: VULNERABLE 

    --------------- RAW HTTP RESPONSE --------------
    HTTP/1.1 200 OK 
    Date: Wed, 23 Jul 2014 13:48:07 GMT 
    Server: Apache/2.4.3 (Win32) OpenSSL/1.0.1c PHP/5.4.7 
    X-Powered-By: PHP/5.4.7 
    Set-Cookie: SessionID=xxx; expires=Wed, 23-Jul-2014 12:48:07 
    GMT; path=/; secure 
    Set-Cookie: SessionChallenge=yyy; expires=Wed, 23-Jul-2014 

    12:48:07 GMT; path=/ Content-Length: 193 Connection: close Content-Type: text/html 
    <html> 
    <head> 
    <title>Login page </title> 
    </head> 
    <body> 
    <script src="http://othersite/test.js"></script> 

    <link rel="stylesheet" type="text/css" href="http://somesite/ 

    test.css"> 
    ===================================== 
    Checking localhost for HTTP support against HTTPS Stripping attack ... 
    [!] HTTP Support on port [80] : SUPPORTED [!] Vulnerable to HTTPS Stripping attack mentioned in https:// www.blackhat.com/presentations/bh-dc-09/Marlinspike/ BlackHat-DC-09-Marlinspike-Defeating-SSL.pdf [!] Vulnerability Status: VULNERABLE 
    ===================================== 
    Checking localhost:443 for HTTP elements embedded in SSL page ... 
    [!] HTTP elements embedded in SSL page: PRESENT [!] Vulnerable to MITM malicious content injection attack [!] Vulnerability Status: VULNERABLE 
    --------------- HTTP RESOURCES EMBEDDED --------------
    -
     http://othersite/test.js

     -
     http://somesite/test.css 


    ===================================== 
    Checking localhost:443 for ROBUST use of anti-caching mechanism ... 
    [!] Cache Control Directives: NOT PRESENT [!] Browsers, Proxies and other Intermediaries will cache SSL page and sensitive information will be leaked. [!] Vulnerability Status: VULNERABLE 
    Robust Solution: 
    -
     Cache-Control: no-cache, no-store, must-revalidate, pre-check=0, post-check=0, max-age=0, s-maxage=0 

    -
     Ref: https://www.owasp.org/index.php/Testing_for_ Browser_cache_weakness_(OTG-AUTHN-006)


           http://msdn.microsoft.com/en-us/library/ ms533020(v=vs.85).aspx 
    ===================================== 
    Checking localhost:443 for Surf Jacking vulnerability (due to Session Cookie missing secure flag) ... 
    [!] Secure Flag in Set-Cookie:  PRESENT BUT NOT IN ALL COOKIES [!] Vulnerable to Surf Jacking attack mentioned in https://resources.enablesecurity.com/resources/Surf%20Jacking.pdf [!] Vulnerability Status: VULNERABLE 


    --------------- RAW HTTP RESPONSE --------------
    HTTP/1.1 200 OK Date: Wed, 23 Jul 2014 13:48:07 GMT Server: Apache/2.4.3 (Win32) OpenSSL/1.0.1c PHP/5.4.7 X-Powered-By: PHP/5.4.7 Set-Cookie: SessionID=xxx; expires=Wed, 23-Jul-2014 12:48:07 GMT; path=/; secure Set-Cookie: SessionChallenge=yyy; expires=Wed, 23-Jul-2014 
    12:48:07 GMT; path=/ Content-Length: 193 Connection: close Content-Type: text/html 
    ===================================== 

    Checking localhost:443 for ECDHE/DHE ciphers against FORWARD SECRECY support ... 

    [*] Forward Secrecy: SUPPORTED 
    [*] Connected using cipher - TLS_ECDHE_RSA_WITH_ 
    AES_128_CBC_SHA on protocol - TLSv1 
    [*] Attackers will NOT be able to decrypt sniffed SSL packets 
    even if they have compromised private keys. 
    [*] Vulnerability Status: No 

    ===================================== 

    Checking localhost:443 for RC4 support (CVE-2013-2566) ... 

    [!] RC4: SUPPORTED 
    [!] Vulnerable to MITM attack described in http://www.isg.rhul. 
    ac.uk/tls/ 
    [!] Vulnerability Status: VULNERABLE 

    ===================================== 

    Checking localhost:443 for TLS 1.1 support ... 

    Checking localhost:443 for TLS 1.2 support ... 

    [*] TLS 1.1, TLS 1.2: SUPPORTED 
    [*] Immune from BEAST attack mentioned in http://www. 
    infoworld.com/t/security/red-alert-https-has-beenhacked-174025 
    [*] Vulnerability Status: No 

    ===================================== 

    Loading module: sslyze by iSecPartners ... 

    Checking localhost:443 for Session Renegotiation support (CVE
    2009-3555,CVE-2011-1473,CVE-2011-5094) ... 
    [*] Secure Client-Initiated Renegotiation : NOT SUPPORTED [*] Mitigated from DOS attack (CVE-20111473,CVE-2011-5094) mentioned in https://www.thc.org/thcssl-dos/ [*] Vulnerability Status: No 
    [*] INSECURE Client-Initiated Renegotiation : NOT SUPPORTED [*] Immune from TLS Plain-text Injection attack (CVE2009-3555) - http://cve.mitre.org/cgi-bin/cvename. cgi?name=CVE-2009-3555 [*] Vulnerability Status: No 
    ===================================== 
    Loading module: TestSSLServer by Thomas Pornin ... 
    Checking localhost:443 for SSL version 2 support ... 
    [*] SSL version 2 : NOT SUPPORTED [*] Immune from SSLv2-based MITM attack [*] Vulnerability Status: No 
    ===================================== 
    Checking localhost:443 for LANE (LOW,ANON,NULL,EXPORT) weak ciphers support ... 
    Supported LANE cipher suites:
      SSLv3
         RSA_EXPORT_WITH_RC4_40_MD5
         RSA_EXPORT_WITH_RC2_CBC_40_MD5
         RSA_EXPORT_WITH_DES40_CBC_SHA
         RSA_WITH_DES_CBC_SHA
         DHE_RSA_EXPORT_WITH_DES40_CBC_SHA
         DHE_RSA_WITH_DES_CBC_SHA
         TLS_ECDH_anon_WITH_RC4_128_SHA
         TLS_ECDH_anon_WITH_3DES_EDE_CBC_SHA
         TLS_ECDH_anon_WITH_AES_256_CBC_SHA

      (TLSv1.0: same as above)
      (TLSv1.1: same as above)
      (TLSv1.2: same as above) 

    [!] LANE ciphers : SUPPORTED [!] Attackers may be ABLE to recover encrypted packets. [!] Vulnerability Status: VULNERABLE 
    ===================================== 
    Checking localhost:443 for GCM/CCM ciphers support against Lucky13 attack (CVE-2013-0169) ... 
    Supported GCM cipher suites against Lucky13 attack: 


      TLSv1.2
         TLS_RSA_WITH_AES_128_GCM_SHA256
         TLS_RSA_WITH_AES_256_GCM_SHA384
         TLS_DHE_RSA_WITH_AES_128_GCM_SHA256
         TLS_DHE_RSA_WITH_AES_256_GCM_SHA384
         TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
         TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 

    [*] GCM/CCM ciphers : SUPPORTED [*] Immune from Lucky13 attack mentioned in http://www.isg. rhul.ac.uk/tls/Lucky13.html [*] Vulnerability Status: No 
    ===================================== 
    Checking localhost:443 for TLS Compression support against 
    CRIME (CVE-2012-4929) & TIME attack  ... 
    [*] TLS Compression : DISABLED [*] Immune from CRIME & TIME attack mentioned in https://media.blackhat.com/eu-13/briefings/Beery/bh-eu-13-a-perfectcrime-beery-wp.pdf [*] Vulnerability Status: No 
    ===================================== 
    [+] Breacher finished scanning in 12 seconds. [+] Get your latest copy at http://yehg.net/ 


|

SSL 인증서 유효성 검사 테스트 - 클라이언트와 서버
-----------------------------------------------------------------------------------------

Firstly upgrade the browser because CA certs expire and in every release of the browser these are renewed. Examine the validity of the certificates used by the application. Browsers will issue a warning when encountering expired certificates, certificates issued by untrusted CAs, and certificates which do not match name wise with the site to which they should refer. 
By clicking on the padlock that appears in the browser window when visiting an HTTPS site, testers can look at information related to the certificate . including the issuer, period of validity, encryption characteristics, etc. If the application requires a client certificate, that tester has probably installed one to access it. Certificate information is available in the browser by inspecting the relevant certificate(s) in the list of the installed certificates. 
These checks must be applied to all visible SSL-wrapped communication channels used by the application. Though this is the usual https service running on port 443, there may be additional services involved depending on the web application architecture and on deployment issues (an HTTPS administrative port left open, HTTPS services on non-standard ports, etc.). Therefore, apply these checks to all SSL-wrapped ports which have been discovered. For example, the nmap scanner features a scanning mode (enabled by the .sV command line switch) which identifies SSL-wrapped services. The Nessus vulnerability scanner has the capability of performing SSL checks on all SSL/TLS-wrapped services. 


예제 1. 인증서 유효성 테스트 (수동)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Rather than providing a fictitious example, this guide includes an anonymized real-life example to stress how frequently one stumbles on https sites whose certificates are inaccurate with respect to naming. The following screenshots refer to a regional site of a high-profile IT company. 
We are visiting a .it site and the certificate was issued to a .com site. Internet Explorer warns that the name on the certificate does not match the name of the site. 

[그림]
Warning issued by Microsoft Internet Explorer 

The message issued by Firefox is different. Firefox complains because it cannot ascertain the identity of the .com site the certificate refers to because it does not know the CA which signed the certificate. In fact, Internet Explorer and Firefox do not come pre-loaded with the same list of CAs. Therefore, the behavior experienced with various browsers may differ. 

[그림]
Warning issued by Mozilla Firefox

|

다른 취약점 테스트
-----------------------------------------------------------------------------------------

앞서 언급한 바와 같이, 사용된 SSL/TLS 프로토콜, 암호화 방식이나 인증서와 관련되지 않은 
다른 유형의 취약점이 있습니다.

서버가 HTTP 및 HTTPS 프로토콜 모두 웹 사이트에서 제공하고, 공격자가 안전한 하나 대신 비 보안 채널을 사용하여 피해자를 강제로 허용하는 경우 취약점이 존재합니다.

|

Surf Jacking 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Surf Jacking 공격은 Sandro Gauci에 의해 발표되었고, 공격자는 피해자 연결이 SSL 또는 TLS를 사용하여 
암호화되었을 경우 HTTP 세션을 하이재킹하여 수행합니다.

다음은 해당 공격에 대한 시나리오 입니다.

- 패해가자 https://somesecuresite/ 에 보안 웹 사이트로 로그인
- 보안 사이트에서 클라이언트 로그인에 대한 세션 쿠키를 발행합니다.
- 로그인 동안, 피해자는 새로운 브라우저 창을 열고 http://examplesite/로 접속
- 동일 네트워크에 접속 상에 접속 중인 공격자는 http://examplesite에 평문 트래픽을 볼 수 있습니다.
- 공격자는 http://examplesite에 평문 트래픽에 대한 응답으로 "301 Moved permanently"로 다시 전송
응답은 examplesite가 somesecuresite로 보여지도록 "Location: http://somesecuresite/" 헤더를 포함합니다.
URL 스키마는 HTTP이지 HTTPS가 아닌 것을 알 수 있습니다.
- 피해자의 브라우저는 http://somesucuresite/로 새로운 평문 접속을 시작하고,
평문으로 HTTP 헤더에 쿠키를 포함하여 전송합니다.
- 공격자는 이 트래픽을 보고 나중에 사용하기 위해 쿠키를 기록합니다.


다음 테스트를 수행하여 웹 사이트 취약점 여부 확인

1. HTTP와 HTTPS 프로토콜 둘다 지원하는 웹 사이트인지 체크protocols 
2. "Secure" 플래그를 가지고 있지 않은 쿠키인지 확인

|

SSL Strip 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

일부 어플리케이션은 HTTP와 HTTPS 둘다 지원합니다.

either for usability or so users can type both addresses and get to the site. 

종종 사용자는 링크 또는 리다이렉션에서 HTTPS 웹 사이트로 이동합니다.

Typically personal banking sites have a similar configuration with an iframed log in or a form with action attribute over HTTPS but the page under HTTP. 
An attacker in a privileged position can intercept traffic when the user is in the http site and manipulate it to get a Man-In-The-Middle attack under HTTPS. 
어플리케이션이 HTTP와 HTTPS 둘다 지원한다면 취약합니다.


|

HTTP 프록시를 통한 테스트 
-----------------------------------------------------------------------------------------

기업 환경 내부에서 테스터는 직접 액세스할 수 없는 서비스가 있기 때문에, 
CONNECT 메소드를 사용하여 HTTP 프록시를 통해서만 접근할 수 있습니다.

대부분의 툴은 SSL/TLS 핸드쉐이크를 시작하려면 원하는 TCP 포트에서 연결을 시도하기 때문에, 
이 시나리오에서 동작하지 않습니다.

socat과 같은 미들 소프트웨어의 도움으로 HTTP 프록시 뒤에 서비스로 사용 툴을 활성화 할 수 있습니다.

|

예제 8. HTTP proxy를 통한 테스트
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

프록시 10.13.37.100:3128을 통해 destined.application.lan:443에 접속하기 위해 socat을 다음과 같이 실행합니다.

.. code-block:: console

    $ socat TCP-LISTEN:9999,reuseaddr,fork PROXY:10.13.37.100:destined.application.lan:443,proxyport=3128 

테스터는 다음 명령을 통해 localhost:9999에 접속할 수 있습니다.

.. code-block:: console

    $ openssl s_client -connect localhost:9999 


socat을 이용한 프록시를 통해 localhost:9999로 destined.application.lan:443에 접속할 수 있습니다.

|

설정 검토
==========================================================================================

|

취약한 SSL/TLS 암호화 방식 테스트
-----------------------------------------------------------------------------------------

HTTPS 서비스를 제공하는 웹 서버의 설정 체크
만약 웹 어플리케이션이 SSL/TLS를 제공한다면, 다음을 확인합니다.

|

예제 9. 윈도우 서버
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

마이크로소프트 윈도우 서버에서 레지스트리 키를 사용하여 설정을 체크합니다.

.. code-block:: html

    HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\ 

Ciphers, Protocols, KeyExchangeAlgorithms을 포함한 서브키를 가지고 있습니다.


예제 10: 아파치
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

암호화 방식과 protocols를 확인하기 위해 아파치2 웹 서버에서 지원하는 ssl.conf를 열고,
SSLCipherSuite, SSLProtocol, SSLHonorCipherOrder,SSLInsecureRenegotiation, SSLCompression을 검색합니다.

|

SSL 인증서 유효성 테스트 - 클라이언트와 서버
-----------------------------------------------------------------------------------------

서버와 클라이언트 레벨에서 어플리케이션에 의해 사용되는 인증서의 유효성을 검사합니다.
웹 서버 레벨에서 주로 인증서가 이용되지만, SSL에 의해 보호할 통신 경로를 추가할 수 있습니다.
테스터는 모든 SSL 보안 채널을 확인하기 위해 어플리케이션의 아키텍쳐를 확인해야합니다.

Testers should check the application architecture to identify all SSL protected channels. 

|

Tools 
==========================================================================================

- Qualys SSL Labs - SSL Server Test: https://www.ssllabs.com/ssltest/index.html
- Tenable - Nessus Vulnerability Scanner: http://www.tenable.com/products/nessus
- TestSSLServer: http://www.bolet.org/TestSSLServer/
- sslyze: https://github.com/iSECPartners/sslyze
- SSLAudit: https://code.google.com/p/sslaudit/
- SSLScan: http://sourceforge.net/projects/sslscan/
- nmap
- curl: http://curl.haxx.se/
- Stunnel: http://www.stunnel.org
- socat: http://www.dest-unreach.org/socat/
- testssl.sh: https://testssl.sh/

|

References 
==========================================================================================

OWASP Resources 
-----------------------------------------------------------------------------------------
 
- 쿠키 속성 테스트 (OTG-SESS-002)
- 네트워크 및 인프라 설정 테스트 (OTG-CONFIG-001)
- HTTP Strict Transport 보안 테스트 (OTG-CONFIG-007) 
- 민감한 정보가 암호화되지 않은 채널에서 보내지는 경우 침투 테스트 (OTG-CRYPST-003)
- 암호화된 채널에서 자격 증명 전송 테스트 (OTG-AUTHN-001)
- OWASP Cheat sheet - Transport Layer Protection: https://www.owasp.org/index.php/Transport_Layer_Protection_Cheat_Sheet
- OWASP TOP 10 2013 - A6 Sensitive Data Exposure: https://www.owasp.org/index.php/Top_10_2013-A6-Sensitive_Data_Exposure
- OWASP TOP 10 2010 - A9 Insufficient Transport Layer Protection: https://www.owasp.org/index.php/Top_10_2010-A9-Insufficient_Transport_Layer_Protection
- OWASP ASVS 2009 - Verification 10: https://code.google.com/p/owasp-asvs/wiki/Verification_V10
- OWASP Application Security FAQ - Cryptography/SSL: https://www.owasp.org/index.php/OWASP_Application_ Security_FAQ#Cryptography.2FSSL

|

Whitepapers 
-----------------------------------------------------------------------------------------

- RFC5246 - The Transport Layer Security (TLS) Protocol Version 1.2 (Updated by RFC 5746, RFC 5878, RFC 6176): http:// www.ietf.org/rfc/rfc5246.txt
- RFC2817 - Upgrading to TLS Within HTTP/1.1
- RFC6066 - Transport Layer Security (TLS) Extensions: Extension Definitions: http://www.ietf.org/rfc/rfc6066.txt 
- SSLv2 Protocol Multiple Weaknesses: http://osvdb.org/56387
- Mitre - TLS Renegotiation MiTM: http://cve.mitre.org/ cgi-bin/cvename.cgi?name=CVE-2009-3555
- Qualys SSL Labs - TLS Renegotiation DoS: https://community.qualys.com/blogs/securitylabs/2011/10/31/tls-renegotiation-and-denial-of-service-attacks
- Qualys SSL Labs - SSL/TLS Deployment Best Practices: https://www.ssllabs.com/projects/best-practices/index. html
- Qualys SSL Labs - SSL Server Rating Guide: https://www.ssllabs.com/projects/rating-guide/index.html
- Qualys SSL Labs - SSL Threat Model: https://www.ssllabs.com/projects/ssl-threat-model/index.html
- Qualys SSL Labs - Forward Secrecy: https://community.qualys.com/blogs/securitylabs/2013/06/25/ssl-labs-deploying-forward-secrecy
- Qualys SSL Labs - RC4 Usage: https://community.qualys.com/blogs/securitylabs/2013/03/19/rc4-in-tls-is-brokennow-what
- Qualys SSL Labs - BEAST: https://community.qualys.com/blogs/securitylabs/2011/10/17/mitigating-the-beast-attack-on-tls
- Qualys SSL Labs - CRIME: https://community.qualys.com/blogs/securitylabs/2012/09/14/crime-information-leakage-attack-against-ssltls
- SurfJacking attack: https://resources.enablesecurity.com/resources/Surf%20Jacking.pdf
- SSLStrip attack: http://www.thoughtcrime.org/software/sslstrip/
- PCI-DSS v2.0: https://www.pcisecuritystandards.org/ security_standards/documents.php
- Xiaoyun Wang, Hongbo Yu: How to Break MD5 and Other Hash Functions: http://link.springer.com/chapter/10.1007/11426639_2

|