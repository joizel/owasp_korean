============================================================================================
OTG-BUSLOGIC-008
============================================================================================

|

예기치 않은 파일 형식 업로드 테스트

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

- Review the project documentation and perform some exploratory testing looking for file types that should be "unsupported" by the application/system. 
- Try to upload these "unsupported" files an verify that it are properly rejected. 
- If multiple files can be uploaded at once, there must be tests in place to verify that each file is properly evaluated. 


구체적인 테스트 방법 1 
-----------------------------------------------------------------------------------------

- Study the applications logical requirements. 
- Prepare a library of files that are "not approved" for upload that may contain files such as: jsp, exe, or html files containing script. 
- In the application navigate to the file submission or upload mechanism. 
- Submit the "not approved" file for upload and verify that they are properly prevented from uploading 

|

Related Test Cases 
============================================================================================

- 민감한 정보를 확인하기 위해 파일 확장자 처리 테스트 (OTG-CONFIG-003) 
- 악성 파일 형식 업로드 테스트 (OTG-BUSLOGIC-009) 

|

References 
============================================================================================

- OWASP - Unrestricted File Upload: https://www.owasp.org index.php/Unrestricted_File_Upload 
- File upload security best practices: Block a malicious file upload: http://www.computerweekly.com/answer/Fileupload-security-best-practices-Block-a-malicious-file-upload 
- Stop people uploading malicious PHP files via forms: http:/ stackoverflow.com/questions/602539/stop-peopleuploading-malicious-php-files-via-forms 
- CWE-434: Unrestricted Upload of File with Dangerous Type: http://cwe.mitre.org/data/definitions/434.html 
- Secure Programming Tips - Handling File Uploads: https:/ www.datasprings.com/resources/dnn-tutorials/artmid/535/ articleid/65/secure-programming-tips-handling-file-uploads? AspxAutoDetectCookieSupport=1 

|

Remediation 
============================================================================================

Applications should be developed with mechanisms to only accept and manipulate "acceptable" files that the rest of the application functionality is ready to handle and expecting. Some specific examples include: Black or White listing of file extensions, using "Content-Type" from the header, or using a file type recognizer, all to only allow specified file types into the system. 

|