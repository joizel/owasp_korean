==========================================================================================
OTG-INFO-004 (웹 서버에 응용 프로그램 확인)
==========================================================================================

|

개요
==========================================================================================

웹 응용 프로그램 취약점 테스트에서 가장 중요한 단계는 웹 서버에 호스팅되어 있는 특정 응용 프로그램을 찾는 것입니다.
많은 응용 프로그램들이 원격 관리 권한을 획득하거나 데이터를 얻기 위해 알려진 취약점 및 공격 기법들이 있습니다.
추가적으로, 많은 응용 프로그램들이 내부적으로 사용한다는 인식 때문에 대부분 잘못된 설정을 하거나 최신으로 업데이트 하지 않아, 위협이 존재할 수 있습니다.

가상 웹 서버 확산으로 기존의 IP와 웹서버의 1:1 형식은 원래 의미를 잃었습니다.

동일한 IP 주소로 확인된 도메인 명이 여러 웹 사이트 또는 응용 프로그램을 가지는게 드문 일이 아닙니다.
이 시나리오는 호스팅 환경에 한정되지 않지만, 통상적으로 기업에 적용됩니다.

보안 전문가들에게는 때때로 침투 테스트 대상으로 IP 주소가 주어지기도 합니다.
IP로 주어지는 침투 테스트 시나리오는 대상을 통해 접근할 수 있는 모든 웹 응용 프로그램을 테스트하기 위한 경우 입니다.
문제는 주어진 IP 주소가 포트 80에서 HTTP 서비스를 호스팅하는 것이지만, 만약 테스터가 IP 주소를 지정하여 접근하면, "No web server configured at this address" 또는 이와 유사한 메시지가 보고됩니다.

그러나, 시스템은 관련없는 DNS에 연결된 웹 응용 프로그램의 수를 숨길 수 있습니다.

분명히 분석의 범위는 테스터가 모든 응용 프로그램을 테스트하는 데 큰 영향을 받습니다.

때때로, 대상 특성이 더 풍부해집니다.

테스터는 IP 주소와 그에 상응하는 DNS를 제공받을 수 있습니다.

그럼에도 불구하고, 이 리스트는 부분적인 정보를 전달할 것입니다. 즉, DNS를 생략할 수 있고, 클라이언트는 그것을 인식하지 못할 수도 있습니다. (이것은 큰 조직에서 발생할 가능성이 더 큽니다.)

평가 범위에 영향을 미치는 다른 문제는 명확하지 않은 URL에 게시된 응용 프로그램에 의해 표현됩니다.
(예제> http://www.example.com/some-strange-URL), 

이것은 설정 실수와 같은 에러나 알려지지 않은 관리자 인터페이스와 같이 의도적으로 발생될 수 있습니다. 

이러한 문제를 해결하기 위해서는, 웹 응용 프로그램의 검색을 수행 할 필요가 있습니다.

|

테스트 목적
==========================================================================================

웹 서버에 존재하는 응용 프로그램 확인
   
|


테스트 방법
==========================================================================================

|

Black Box Testing
-------------------------------------------------------------------------------------------

웹 응용 프로그램 발견은 주어진 인프라에서 웹 응용 프로그램을 식별하기 위한 과정입니다.
후자는 일반적으로 IP 주소의 집합으로 지정되지만, DNS 명 또는 이들 조합으로 구성될 수 있습니다.

This information is handed out prior to the execution of an assessment, be it a classic-style penetration test or an application-focused assessment. 
In both cases, unless the rules of engagement specify otherwise (e.g., "test only the application located at the URL http://www.example.com/"), the assessment should strive to be the most comprehensive in scope, i.e. it should identify all the applications accessible through the given target. 
The following examples examine a few techniques that can be employed to achieve this goal.

Note: Some of the following techniques apply to Internet-facing web servers, namely DNS and reverse-IP web-based search services and the use of search engines. Examples make use of private IP addresses (such as 192.168.1.100), which, unless indicated otherwise, represent generic IP addresses and are used only for anonymity purposes.

There are three factors influencing how many applications are related to a given DNS name (or an IP address):

|

1. 다른 기본 URL
-------------------------------------------------------------------------------------------

The obvious entry point for a web application is www.example.com, i.e., with this shorthand notation we think of the web application originating at http://www.example.com/ (the same applies for https). 
However, even though this is the most common situation, there is nothing forcing the application to start at "/".

예를 들어, 다음과 같은 세 가지 웹 응용 프로그램에 동일한 기호 이름을 연결할 수 있습니다.

- http://www.example.com/url1 
- http://www.example.com/url2 
- http://www.example.com/url3

이 경우 URL http://www.example.com/은 의미있는 페이지로 연결되지 않으며, 세 가지 응용 프로그램은 테스터가 정확하게 연결하는 방법을 알지 못하는 한 "숨김"상태가 됩니다.
즉, 테스터는 url1, url2 또는 url3로 알고 있습니다.

There is usually no need to publish web applications in this way, unless the owner doesn’t want them to be accessible in a standard way, and is prepared to inform the users about their exact location. 
This doesn’t mean that these applications are secret, just that their existence and location is not explicitly advertised.

|

2. 비표준 포트
-------------------------------------------------------------------------------------------

While web applications usually live on port 80 (http) and 443 (https), there is nothing magic about these port numbers. 
In fact, web applications may be associated with arbitrary TCP ports, and can be referenced by specifying the port number as follows: http[s]://www.example.com:port/. 

For example, http://www.example.com:20000/.

nmap –PN –sT –sV –p0-65535 192.168.1.100

Web Application Penetration Testing

|

3. Virtual hosts
-------------------------------------------------------------------------------------------

DNS allows a single IP address to be associated with one or more
symbolic names. For example, the IP address 192.168.1.100 might
be associated to DNS names www.example.com, helpdesk.example.
com, webmail.example.com. It is not necessary that all the names
belong to the same DNS domain. This 1-to-N relationship may be reflected
to serve different content by using so called virtual hosts. The
information specifying the virtual host we are referring to is embedded
in the HTTP 1.1 Host: header [1].

One would not suspect the existence of other web applications in addition
to the obvious www.example.com, unless they know of helpdesk.example.com
and webmail.example.com.

|

Approaches to address issue 1 - non-standard URLs
-------------------------------------------------------------------------------------------

There is no way to fully ascertain the existence of non-standardnamed web applications. 

Being non-standard, there is no fixed criteria governing the naming convention, however there are a number of techniques that the tester can use to gain some additional insight.
First, if the web server is mis-configured and allows directory browsing, it may be possible to spot these applications. 
Vulnerability scanners may help in this respect.

Second, these applications may be referenced by other web pages and there is a chance that they have been spidered and indexed by web search engines. 
If testers suspect the existence of such "hidden" applications on www.example.com they could search using the site operator and examining the result of a query for "site: www.example.com".
Among the returned URLs there could be one pointing to such a non-obvious application.

Another option is to probe for URLs which might be likely candidates for non-published applications. 
For example, a web mail front end might be accessible from URLs such as https://www.example.com/webmail, https://webmail.example.com/, or https://mail.example.com/. 
The same holds for administrative interfaces, which may be published at hidden URLs (for example, a Tomcat administrative interface), and yet not referenced anywhere. 
So doing a bit of dictionary-style searching(or "intelligent guessing") could yield some results. Vulnerability scanners may help in this respect.

|

Approaches to address issue 2 - non-standard ports
-------------------------------------------------------------------------------------------

It is easy to check for the existence of web applications on non-standard
ports. A port scanner such as nmap [2] is capable of performing
service recognition by means of the -sV option, and will identify http[s]
services on arbitrary ports. What is required is a full scan of the whole
64k TCP port address space.

For example, the following command will look up, with a TCP connect
scan, all open ports on IP 192.168.1.100 and will try to determine what
services are bound to them (only essential switches are shown – nmap
features a broad set of options, whose discussion is out of scope):
It is sufficient to examine the output and look for http or the indication
of SSL-wrapped services (which should be probed to confirm
that they are https). For example, the output of the previous command
coullook like:

.. code-block:: console

    nmap –PN –sT –sV –p0-65535 192.168.1.100

It is sufficient to examine the output and look for http or the indication
of SSL-wrapped services (which should be probed to confirm
that they are https). 
For example, the output of the previous command could look like:

.. code-block:: console

    901/tcp	 open http Samba SWAT administration server
    1241/tcp open ssl Nessus security scanner
    3690/tcp open unknown
    8000/tcp open http-alt?
    8080/tcp open http Apache Tomcat/Coyote JSP engine 1.1

From this example, one see that:

- There is an Apache http server running on port 80.
- It looks like there is an https server on port 443 (but this needs to be confirmed, for example, by visiting https://192.168.1.100 with a browser).
- On port 901 there is a Samba SWAT web interface.
- The service on port 1241 is not https, but is the SSL-wrapped Nessus daemon.
- Port 3690 features an unspecified service (nmap gives back its fingerprint - here omitted for clarity - together with instructions to submit it for incorporation in the nmap fingerprint database, provided you know which service it represents).
- Another unspecified service on port 8000; this might possibly be http, since it is not uncommon to find http servers on this port. Let’s examine this issue:

.. code-block:: console

    Interesting ports on 192.168.1.100:
    (The 65527 ports scanned but not shown below are in state:
    closed)
    PORT STATE SERVICE VERSION
    22/tcp open ssh OpenSSH 3.5p1 (protocol 1.99)
    80/tcp open http Apache httpd 2.0.40 ((Red Hat Linux))
    443/tcp open ssl OpenSSL

This confirms that in fact it is an HTTP server. Alternatively, testers
could have visited the URL with a web browser; or used the GET or
HEAD Perl commands, which mimic HTTP interactions such as the
one given above (however HEAD requests may not be honored by all
servers).

- Apache Tomcat running on port 8080.

The same task may be performed by vulnerability scanners, but first
check that the scanner of choice is able to identify http[s] services
running on non-standard ports. For example, Nessus [3] is capable of
identifying them on arbitrary ports (provided it is instructed to scan all
the ports), and will provide, with respect to nmap, a number of tests
on known web server vulnerabilities, as well as on the SSL configuration
of https services. As hinted before, Nessus is also able to spot
popular applications or web interfaces which could otherwise go unnoticed
(for example, a Tomcat administrative interface).

|

Approaches to address issue 3 - virtual hosts
-------------------------------------------------------------------------------------------

주어진 IP 주소 x.y.z.t에 할당된 DNS명을 식별하기 위해 사용할 수 있는 수많은 기술이 있습니다..

DNS zone transfers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This technique has limited use nowadays, given the fact that zone 
transfers are largely not honored by DNS servers. 
However, it may
be worth a try. First of all, testers must determine the name servers
serving x.y.z.t. 
If a symbolic name is known for x.y.z.t (let it be www.
example.com), its name servers can be determined by means of tools
such as nslookup, host, or dig, by requesting DNS NS records.

If no symbolic names are known for x.y.z.t, but the target definition
contains at least a symbolic name, testers may try to apply the same
process and query the name server of that name (hoping that x.y.z.t
will be served as well by that name server). 
For example, if the target
consists of the IP address x.y.z.t and the name mail.example.com, determine
the name servers for domain example.com.
The following example shows how to identify the name servers for
www.owasp.org by using the host command:

.. code-block:: console

    $ host -t ns www.owasp.org
    www.owasp.org is an alias for owasp.org.
    owasp.org name server ns1.secure.net.
    owasp.org name server ns2.secure.net.

A zone transfer may now be requested to the name servers for domain
example.com. If the tester is lucky, they will get back a list of the
DNS entries for this domain. This will include the obvious www.example.com
and the not-so-obvious helpdesk.example.com and webmail.
example.com (and possibly others). Check all names returned by the
zone transfer and consider all of those which are related to the target
being evaluated.
Trying to request a zone transfer for owasp.org from one of its name
servers:

.. code-block:: console

    $ host -l www.owasp.org ns1.secure.net
    Using domain server:
    Name: ns1.secure.net
    Address: 192.220.124.10#53
    Aliases:

    Host www.owasp.org not found: 5(REFUSED)
    ; Transfer failed.


DNS inverse queries
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This process is similar to the previous one, but relies on inverse (PTR) DNS records.
Rather than requesting a zone transfer, try setting the record type to PTR and issue a query on the given IP address. 
If the testers are lucky, they may get back a DNS name entry. 
This technique relies on the existence of IP-to-symbolic name maps, which is not guaranteed.


Web-based DNS searches
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This kind of search is akin to DNS zone transfer, but relies on webbased services that enable name-based searches on DNS. 
One such service is the Netcraft Search DNS service, available at http://searchdns.netcraft.com/?host. 
The tester may query for a list of names belonging to your domain of choice, such as example.com.
Then they will check whether the names they obtained are pertinent to the target they are examining.

Reverse-IP services
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Reverse-IP services are similar to DNS inverse queries, with the difference that the testers query a web-based application instead of a name server. 

There are a number of such services available. 

Since they tend to return partial (and often different) results, it is better to use multiple services to obtain a more comprehensive analysis.

- Domain tools reverse IP: http://www.domaintools.com/reverse-ip/ (requires free membership)
- MSN search: http://search.msn.com syntax: "ip:x.x.x.x" (without the quotes)
- Webhosting info: http://whois.webhosting.info/ syntax: http://whois.webhosting.info/x.x.x.x
- DNSstuff: http://www.dnsstuff.com/ (multiple services available) http://www.net-square.com/mspawn.html (multiple queries on domains and IP addresses, requires installation)
- tomDNS: http://www.tomdns.net/index.php (some services are still private at the time of writing)
- SEOlogs.com: http://www.seologs.com/ip-domains.html (reverse-IP/domain lookup)

The following example shows the result of a query to one of the above reverse-IP services to 216.48.3.18, the IP address of www.owasp.org.

Three additional non-obvious symbolic names mapping to the same address have been revealed. 

Googling
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

이 전 기술로 부터 정보 수집 후, 테스터는 가능한 세밀하게 구분하고 자신의 분석을 증가하기 위해 써치 엔진에 의존 할 수 있습니다.
대상에 속하는 추가 도메인 명의 증거를 얻을 수 있거나, 비 명백한 URL을 통해 액세스 할 수 있습니다.

For instance, considering the previous example regarding www.owasp.org, the tester could query Google and other search engines looking for information (hence, DNS names) related to the newly discovered domains of webgoat.org, webscarab.com, and webscarab.net.

구글링 기술은 Spiders, Robots, Crawlers 테스트를 위해 설명되었습니다.

|

Gray Box Testing
-------------------------------------------------------------------------------------------

Not applicable.

|

도구
==========================================================================================

- DNS lookup tools: nslookup, dig.
- Search engines: Google, Bing
- Specialized DNS-related web-based search service: see text.
- Nmap: http://www.insecure.org
- Nessus Vulnerability Scanner: http://www.nessus.org
- Nikto: http://www.cirt.net/nikto2

|

참고 문헌
==========================================================================================

- RFC 2616: Hypertext Transfer Protocol – HTTP 1.1

|