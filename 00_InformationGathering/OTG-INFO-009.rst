==========================================================================================
OTG-INFO-009
==========================================================================================

|

웹 어플리케이션 핑거프린트

|

개요
==========================================================================================

There is nothing new under the sun, and nearly every web application
that one may think of developing has already been developed.
With the vast number of free and open source software projects
that are actively developed and deployed around the world, it is
very likely that an application security test will face a target site
that is entirely or partly dependent on these well known applications
(e.g. Wordpress, phpBB, Mediawiki, etc). Knowing the web
application components that are being tested significantly helps
in the testing process and will also drastically reduce the effort
required during the test. These well known web applications have
known HTML headers, cookies, and directory structures that can
be enumerated to identify the application.

|

테스트 목적
==========================================================================================

Identify the web application and version to determine known vulnerabilities
and the appropriate exploits to use during testing.

|


테스트 방법
==========================================================================================

**Cookies**

A relatively reliable way to identify a web application is by the application-specific
cookies.

Consider the following HTTP-request:

.. code-block:: console

    GET / HTTP/1.1
    User-Agent: Mozilla/5.0 (Windows NT 6.2; WOW64; rv:31.0)
    Gecko/20100101 Firefox/31.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    ‘’’Cookie: wp-settings-time-1=1406093286; wp-settingstime-2=1405988284’’’
    DNT: 1
    Connection: keep-alive
    Host: blog.owasp.org

The cookie CAKEPHP has automatically been set, which gives information
about the framework being used. List of common cookies
names is presented in Cpmmon Application Identifiers section.
However, it is possible to change the name of the cookie.

**HTML source code**

This technique is based on finding certain patterns in the HTML
page source code. Often one can find a lot of information which
helps a tester to recognize a specific web application. One of the
common markers are HTML comments that directly lead to application
disclosure. More often certain application-specific paths
can be found, i.e. links to application-specific css and/or js folders.
Finally, specific script variables might also point to a certain application.
From the meta tag below, one can easily learn the application
used by a website and its version. The comment, specific paths
and script variables can all help an attacker to quickly determine
an instance of an application

.. code-block:: html

    <meta name=”generator” content=”WordPress 3.9.2” />

More frequently such information is placed between <head></
head> tags, in <meta> tags or at the end of the page. Nevertheless,
 it is recommended to check the whole document since it can be
useful for other purposes such as inspection of other useful comments
and hidden fields.
Specific files and folders
Apart from information gathered from HTML sources, there is another
approach which greatly helps an attacker to determine the
application with high accuracy. Every application has its own specific
file and folder structure on the server. It has been pointed out
that one can see the specific path from the HTML page source but
sometimes they are not explicitly presented there and still reside
on the server.
In order to uncover them a technique known as dirbusting is used.
Dirbusting is brute forcing a target with predictable folder and file
names and monitoring HTTP-responses to emumerate server
contents. This information can be used both for finding default
files and attacking them, and for fingerprinting the web application.
Dirbusting can be done in several ways, the example below
shows a successful dirbusting attack against a WordPress-powered 
target with the help of defined list and intruder functionality
of Burp Suite.
We can see that for some WordPress-specific folders (for instance,
/wp-includes/, /wp-admin/ and /wp-content/) HTTP-reponses
are 403 (Forbidden), 302 (Found, redirection to wp-login.
php) and 200 (OK) respectively. This is a good indicator that the
target is WordPress-powered. The same way it is possible to dirbust
different application plugin folders and their versions. On
the screenshot below one can see a typical CHANGELOG file of a
Drupal plugin, which provides information on the application being
used and discloses a vulnerable plugin version.

Tip: before starting dirbusting, it is recommended to check the robots.txt
file first. Sometimes application specific folders and other
sensitive information can be found there as well. An example of
such a robots.txt file is presented on a screenshot below.

Specific files and folders are different for each specific application.
It is recommended to install the corresponding application during
penetration tests in order to have better understanding of what infrastructure
is presented and what files might be left on the server.
However, several good file lists already exist and one good example
is FuzzDB wordlists of predictable files/folders (http://code.google.
com/p/fuzzdb/).



|

Tools
==========================================================================================



|

References
==========================================================================================


|

Remediation
==========================================================================================


|