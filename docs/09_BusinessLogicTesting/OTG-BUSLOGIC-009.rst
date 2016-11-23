============================================================================================
OTG-BUSLOGIC-009 (악성 파일 형식 업로드 테스트)
============================================================================================

|

개요
============================================================================================

수많은 어플리케이션 비즈니스 프로세스는 데이터 및 정보 업로드를 허가합니다.

We regularly check the validity and security of text but accepting files can introduce even more risk. To reduce the risk we may only accept certain file extensions, but attackers are able to encapsulate malicious code into inert file types. Testing for malicious files verifies that the application/system is able to correctly protect against attackers uploading malicious files. 

Vulnerabilities related to the uploading of malicious files is unique in that these "malicious" files can easily be rejected through including business logic that will scan files during the upload process and reject those perceived as malicious. Additionally, this is different from uploading unexpected files in that while the file type may be accepted the file may still be malicious to the system. 

Finally, "malicious" means different things to different systems, for example Malicious files that may exploit SQL server vulnerabilities may not be considered a "malicious" to a main frame flat file environment. 

The application may allow the upload of malicious files that include exploits or shellcode without submitting them to malicious file scanning. Malicious files could be detected and stopped at various points of the application architecture such as: IPS/IDS, application server anti-virus software or anti-virus scanning by application as files are uploaded (perhaps offloading the scanning using SCAP). 

|

예제
============================================================================================

|

Suppose a picture sharing application allows users to upload their .gif or .jpg graphic files to the web site. What if an attacker is able to upload a PHP shell, or exe file, or virus? The attacker may then upload the file that may be saved on the system and the virus may spread itself or through remote processes exes or shell code can be executed. 

|

테스트 방법
============================================================================================

일반 테스트 방법
-----------------------------------------------------------------------------------------

- Review the project documentation and use exploratory testing looking at the application/system to identify what constitutes and "malicious" file in your environment. 
- 알려진 "악성" 파일 개발 또는 획득
- 어플리케이션 및 시스템에 악성 파일을 업로드하고, 올바르게 차단되는지 확인하십시오.
- 만약 여러 파일을 한 번에 업로드 할 수 있는 경우, 각 파일이 제대로 차단되었는지 테스트를 해야합니다.

|

구체적인 테스트 방법 1 
-----------------------------------------------------------------------------------------

- 메타스플로잇 페이로드 생성 기능을 사용하면, 메타스플로잇 "msfpayload" 명령을 사용하여 윈도우 실행 파일로 쉘 코드를 생성합니다.
- 어플리케이션 업로드 기능을 통해 실행 파일 제출하고, 적절하게 승인 또는 차단되는 지 확인

|

구체적인 테스트 방법 2
-----------------------------------------------------------------------------------------

- 어플리케이션의 악성 코드 탐지 프로세를 실패하는 파일을 만들거나 개발.
ducklin.htm 또는 ducklin-html.htm과 같이 인터넷에서 많이 이용할 수 있습니다.
- 어플리케이션 업로드 기능을 통해 실행 파일 제출하고, 적절하게 승인 또는 차단되는 지 확인

구체적인 테스트 방법 3
-----------------------------------------------------------------------------------------

- 프록시를 통해 승인된 파일에 대한 "valid" 요청을 캡쳐 설정
- 유효/승인할 수 있는 확장자를 통해 "invalie" 요청을 보내고, 적절하게 승인 또는 차단되는 지 확인

|

관련 테스트 케이스
============================================================================================

- 민감한 정보를 확인하기 위해 파일 확장자 처리 테스트 (OTG-CONFIG-003) 
- 예기치 않은 파일 형식 업로드 테스트 (OTG-BUSLOGIC-008) 

|

도구 
============================================================================================
 
- 메타스플로잇의 페이로드 생성 기능
- Intercepting proxy 

|

참고 문헌 
============================================================================================

- OWASP - Unrestricted File Upload: https://www.owasp.org/index.php/Unrestricted_File_Upload 
- Why File Upload Forms are a Major Security Threat: http:/www.acunetix.com/websitesecurity/upload-forms-threat/ 
- Stop people uploading malicious PHP files via forms - http://stackoverflow.com/questions/602539/stop-peopleuploading-malicious-php-files-via-forms 
- CWE-434: Unrestricted Upload of File with Dangerous Type - http://cwe.mitre.org/data/definitions/434.html 
- Matasploit Generating Payloads - http://www.offensive-security.com/metasploit-unleashed/Generating_Payloads 
- Anti-Malware Test file - http://www.eicar.org/86-0-Intended use.html 

|

개선 방안 
============================================================================================

While safeguards such as black or white listing of file extensions, using "Content-Type" from the header, or using a file type recognizer may not always be protections against this type of vulnerability. Every application that accepts files from users must have a mechanism to verify that the uploaded file does not contain malicious code. Uploaded files should never be stored where the users or attackers can directly access them. 

|