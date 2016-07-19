============================================================================================
OTG-AUTHZ-004
============================================================================================

|

안전하지 않은 직접 객체 참조 테스트

|

개요
==========================================================================================

Insecure Direct Object References occur when an application provides direct access to objects based on user-supplied input. As a result of this vulnerability attackers can bypass authorization and access resources in the system directly, for example database records or files. 

Insecure Direct Object References allow attackers to bypass authorization and access resources directly by modifying the value of a parameter used to directly point to an object. Such resources can be database entries belonging to other users, files in the system, and more. This is caused by the fact that the application takes user supplied input and uses it to retrieve an object without performing sufficient authorization checks. How to Test 

To test for this vulnerability the tester first needs to map out all locations in the application where user input is used to reference objects directly. For example, locations where user input is used to access a database row, a file, application pages and more. Next the tester should modify the value of the parameter used to reference objects and assess whether it is possible to retrieve objects belonging to other users or otherwise bypass authorization. 

The best way to test for direct object references would be by having at least two (often more) users to cover different owned objects and functions. For example two users each having access to different objects (such as purchase information, private messages, etc.), and (if relevant) users with different privileges (for example administrator users) to see whether there are direct references to application functionality. By having multiple users the tester saves valuable testing time in guessing different object names as he can attempt to access objects that belong to the other user. 

Below are several typical scenarios for this vulnerability and the methods to test for each: 

The value of a parameter is used directly to retrieve a database record 
Sample request: 
http://foo.bar/somepage?invoice=12345 
In this case, the value of the invoice parameter is used as an index in an invoices table in the database. The application takes the value of this parameter and uses it in a query to the database. The application then returns the invoice information to the user. 
Since the value of invoice goes directly into the query, by modifying the value of the parameter it is possible to retrieve any invoice object, regardless of the user to whom the invoice belongs. To test for this case the tester should obtain the identifier of an invoice belonging to a different test user (ensuring he is not supposed to view this information per application business logic), and then check whether it is possible to access objects without authorization. 
The value of a parameter is used directly to perform an operation in the system 
Sample request: 

.. code-block:: html

    http://foo.bar/changepassword?user=someuser 


In this case, the value of the user parameter is used to tell the application for which user it should change the password. In many cases this step will be a part of a wizard, or a multi-step operation. In the first step the application will get a request stating for which user's password is to be changed, and in the next step the user will provide a new password (without asking for the current one). 
The user parameter is used to directly reference the object of the user for whom the password change operation will be performed. To test for this case the tester should attempt to provide a different test username than the one currently logged in, and check whether it is possible to modify the password of another user. 
The value of a parameter is used directly to retrieve a file system resource 
Sample request: 

.. code-block:: html

    http://foo.bar/showImage?img=img00011 

In this case, the value of the file parameter is used to tell the application what file the user intends to retrieve. By providing the name or identifier of a different file (for example file=image00012. jpg) the attacker will be able to retrieve objects belonging to other users. 
To test for this case, the tester should obtain a reference the user is not supposed to be able to access and attempt to access it by using it as the value of file parameter. Note: This vulnerability is often exploited in conjunction with a directory/path traversal vulnerability (see Testing for Path Traversal) 
The value of a parameter is used directly to access application functionality 
Sample request: 

.. code-block:: html

    http://foo.bar/accessPage?menuitem=12 

In this case, the value of the menuitem parameter is used to tell the application which menu item (and therefore which application functionality) the user is attempting to access. Assume the user is supposed to be restricted and therefore has links available only to access to menu items 1, 2 and 3. By modifying the value of menu-item parameter it is possible to bypass authorization and access additional application functionality. To test for this case the tester identifies a location where application functionality is determined by reference to a menu item, maps the values of menu items the given test user can access, and then attempts other menu items. 
In the above examples the modification of a single parameter is sufficient. However, sometimes the object reference may be split between more than one parameter, and testing should be adjusted accordingly. 

|

References 
==========================================================================================

Top 10 2013-A4-Insecure Direct Object References 

|

Session Management Testing
==========================================================================================

One of the core components of any web-based application is the mechanism by which it controls and maintains the state for a user interacting with it. This is referred to this as Session Management and is defined as the set of all controls governing state-full interaction between a user and the web-based application. This broadly covers anything from how user authentication is performed, to what happens upon them logging out. 
HTTP is a stateless protocol, meaning that web servers respond to client requests without linking them to each other. Even simple application logic requires a user's multiple requests to be associated with each other across a "session". This necessitates third party solutions 
. through either Off-The-Shelf (OTS) middleware and web server solutions, or bespoke developer implementations. Most popular web application environments, such as ASP and PHP, provide developers with built-in session handling routines. Some kind of identification token will typically be issued, which will be referred to as a "Session ID" or Cookie. 
There are a number of ways in which a web application may interact with a user. Each is dependent upon the nature of the site, the security, and availability requirements of the application. Whilst there are accepted best practices for application development, such as those outlined in the OWASP Guide to Building Secure Web Applications, it is important that application security is considered within the context of the provider's requirements and expectations. 

