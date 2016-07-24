============================================================================================
OTG-BUSLOGIC-008 (예기치 않은 파일 형식 업로드 테스트)
============================================================================================

|

개요
============================================================================================

Many application's business processes allow for the upload and manipulation of data that is submitted via files. But the business process must check the files and only allow certain "approved" file types. Deciding what files are "approved" is determined by the business logic and is application/system specific. The risk in that by allowing users to upload files, attackers may submit an unexpected file type that that could be executed and adversely impact the application or system through attacks that may deface the web site, perform remote commands, browse the system files, browse the local resources, attack other servers, or exploit the local vulnerabilities, just to name a few. 

Vulnerabilities related to the upload of unexpected file types is unique in that the upload should quickly reject a file if it does not have a specific extension. Additionally, this is different from uploading malicious files in that in most cases an incorrect file format may not by it self be inherently "malicious" but may be detrimental to the saved data. For example if an application accepts Windows Excel files, if an similar database file is uploaded it may be read but data extracted my be moved to incorrect locations. 

The application may be expecting only certain file types to be uploaded for processing, such as .CSV, .txt files. The application may not validate the uploaded file by extension (for low assurance file validation) or content (high assurance file validation). This may result in unexpected system or database results within the application/system or give attackers additional methods to exploit the application/system. 

|

예제
============================================================================================

Suppose a picture sharing application allows users to upload a .gif or .jpg graphic file to the web site. What if an attacker is able to upload an html file with a <script> tag in it or php file? The system may move the file from a temporary location to the final location where the php code can now be executed against the application or system. 

|

테스트 방법
============================================================================================

일반 테스트 방법
-----------------------------------------------------------------------------------------

- 프로젝트 문서 검토 및 어플리케이션/시스템에서 "지원하지 않는" 파일 형식을 참는 탐지 테스트 수행
- "지원하지 않는" 파일을 업로드하고 적절하게 차단하는 지 확인
- 만약 여러 파일을 한 번에 업로드 할 수 있는 경우, 각 파일이 제대로 차단되었는지 테스트를 해야합니다.

|

구체적인 테스트 방법 1 
-----------------------------------------------------------------------------------------

- 어플리케이션 로직 요구사항 연구
- JSP, EXE, 또는 스크립트가 포함 된 HTML 파일 같은 파일을 포함하여 업로드를 하기 위해
"승인하지 않음"이 있는 파일 라이브러리 준비
- 어플리케이션에서 파일 제출 또는 업로드 메카니즘으로 이동
- 업로드를 하기 위해 "승인하지 않음" 파일 제출하고 업로드로부터 제대로 차단하는지 확인

|

관련 테스트 케이스
============================================================================================

- 민감한 정보를 확인하기 위해 파일 확장자 처리 테스트 (OTG-CONFIG-003) 
- 악성 파일 형식 업로드 테스트 (OTG-BUSLOGIC-009) 

|

References 
============================================================================================

- OWASP - Unrestricted File Upload: https://www.owasp.org/index.php/Unrestricted_File_Upload 
- Stop people uploading malicious PHP files via forms: http://stackoverflow.com/questions/602539/stop-peopleuploading-malicious-php-files-via-forms 
- CWE-434: Unrestricted Upload of File with Dangerous Type: http://cwe.mitre.org/data/definitions/434.html 
- Secure Programming Tips - Handling File Uploads: https://www.datasprings.com/resources/dnn-tutorials/artmid/535/articleid/65/secure-programming-tips-handling-file-uploads?AspxAutoDetectCookieSupport=1 

|

Remediation 
============================================================================================

Applications should be developed with mechanisms to only accept and manipulate "acceptable" files that the rest of the application functionality is ready to handle and expecting. Some specific examples include: Black or White listing of file extensions, using "Content-Type" from the header, or using a file type recognizer, all to only allow specified file types into the system. 

|