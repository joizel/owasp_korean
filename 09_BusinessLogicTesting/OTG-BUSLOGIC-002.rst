============================================================================================
OTG-BUSLOGIC-002
============================================================================================

|

요청 위조 기능 테스트

|

개요
============================================================================================

Forging requests is a method that attackers use to circumvent the front end GUI application to directly submit information for back end processing. The goal of the attacker is to send HTTP POST/GET requests through an intercepting proxy with data values that is not supported, guarded against or expected by the applications business logic. Some examples of forged requests include exploiting guessable or predictable parameters or expose "hidden" features and functionality such as enabling debugging or presenting special screens or windows that are very useful during development but may leak information or bypass the business logic. 
Vulnerabilities related to the ability to forge requests is unique to each application and different from business logic data validation in that it s focus is on breaking the business logic workflow. 
Applications should have logic checks in place to prevent the system from accepting forged requests that may allow attackers the opportunity to exploit the business logic, process, or flow of the application. Request forgery is nothing new; the attacker uses an intercepting proxy to send HTTP POST/GET requests to the application. Through request forgeries attackers may be able to circumvent the business logic or process by finding, predicting and manipulating parameters to make the application think a process or task has or has not taken place. 
Also, forged requests may allow subvention of programmatic or business logic flow by invoking "hidden" features or functionality such as debugging initially used by developers and testers sometimes referred to as an "Easter egg". "An Easter egg is an intentional inside joke, hidden message, or feature in a work such as a computer program, movie, book, or crossword. According to game designer Warren Robinett, the term was coined at Atari by personnel who were alerted to the presence of a secret message which had been hidden by Robinett in his already widely distributed game, Adventure. The name has been said to evoke the idea of a traditional Easter egg hunt." http://en.wikipedia.org/wiki/ Easter_egg_(media) 

|

예제
============================================================================================

|

예제 1
-----------------------------------------------------------------------------------------

Suppose an e-commerce theater site allows users to select their ticket, apply a onetime 10% Senior discount on the entire sale, view the subtotal and tender the sale. If an attacker is able to see through a proxy that the application has a hidden field (of 1 or 0) used by the business logic to determine if a discount has been taken or not. The attacker is then able to submit the 1 or "no discount has been taken" value multiple times to take advantage of the same discount multiple times. 

|

예제 2
-----------------------------------------------------------------------------------------

Suppose an online video game pays out tokens for points scored for finding pirates treasure and pirates and for each level completed. These tokens can later be that can later be exchanged for prizes. Additionally each level's points have a multiplier value equal to the level. If an attacker was able to see through a proxy that the application has a hidden field used during development and testing to quickly get to the highest levels of the game they could quickly get to the highest levels and accumulate unearned points quickly. 
Also, if an attacker was able to see through a proxy that the application has a hidden field used during development and testing to enabled a log that indicated where other online players, or hidden treasure were in relation to the attacker, they would then be able to quickly go to these locations and score points. 

|


테스트 방법
============================================================================================

Generic Testing Method 

- Review the project documentation and use exploratory testing looking for guessable, predictable or hidden functionality of fields. 
- Once found try to insert logically valid data into the application/system allowing the user go through the application/system against the normal busineess logic workflow. 

Specific Testing Method 1 

- Using an intercepting proxy observe the HTTP POST/GET looking for some indication that values are incrementing at a regular interval or are easily guessable. 
- If it is found that some value is guessable this value may be changed and one may gain unexpected visibility. 

Specific Testing Method 2 

- Using an intercepting proxy observe the HTTP POST/GET looking for some indication of hidden features such as debug that can be switched on or activated. 
- If any are found try to guess and change these values to get a different application response or behavior. 

|

Related Test Cases 
============================================================================================

Testing for Exposed Session Variables (OTG-SESS-004) 
Testing for Cross Site Request Forgery (CSRF) (OTG-SESS-005) 
Testing for Account Enumeration and Guessable User Account (OTG-IDENT-004) 

|

Tools 
============================================================================================

OWASP Zed Attack Proxy (ZAP) - https://www.owasp.org index.php/OWASP_Zed_Attack_Proxy_Project 

ZAP는 웹 어플리케이션에서 취약점을 찾아주는 통합 침투 테스트 툴로 사용하기 쉽습니다.
폭 넓은 보안 경력을 가진 사람들이 사용하도록 설계되었고, 개발자나 기능 테스터들에게 이상적입니다.

ZAP는 자동 스캐너뿐만 아니라 수동으로 보안 취약점을 찾을 수 있는 툴 셋도 제공합니다.

|

References 
============================================================================================

Cross Site Request Forgery - Legitimizing Forged Requests 
http://fragilesecurity.blogspot.com/2012/11/cross-siterequest-forgery-legitimazing.html 
Debugging features which remain present in the final game 
http://glitchcity.info/wiki/index.php/List_of_video_games_ with_debugging_features#Debugging_features_which_ remain_present_in_the_final_game 
Easter egg - http://en.wikipedia.org/wiki/Easter_egg_(media) 
Top 10 Software Easter Eggs - http://lifehacker.com/371083/ top-10-software-easter-eggs 

|

Remediation 
============================================================================================

The application must be smart enough and designed with business logic that will prevent attackers from predicting and manipulating parameters to subvert programmatic or business logic flow, or exploiting hidden/undocumented functionality such as debugging. 

|