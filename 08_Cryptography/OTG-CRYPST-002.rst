============================================================================================
OTG-CRYPST-002 (패딩 오라클 침투 테스트)
============================================================================================

|

개요
==========================================================================================

패딩 오라클은 클라이언트에 의해 제공되는 암호 데이터를 복호화하는 어플리케이션 함수 입니다.
(예를 들어, 클라이언트에 저장되는 내부 세션 상태) 복호화 후 패딩의 유효성 상태를 확인합니다.
패팅 오라클 존재는 공격자가 

The existence of a padding oracle allows an attacker to decrypt encrypted data and encrypt arbitrary data without knowledge of the key used for these cryptographic operations. 

This can lead to leakage of sensible data or to privilege escalation vulnerabilities, if integrity of the encrypted data is assumed by the application. 

Block ciphers encrypt data only in blocks of certain sizes. Block sizes used by common ciphers are 8 and 16 bytes. Data where the size doesn't match a multiple of the block size of the used cipher has to be padded in a specific manner so the decryptor is able to strip the padding. A commonly used padding scheme is PKCS#7. It fills the remaining bytes with the value of the padding length. 

**예제**

If the padding has the length of 5 bytes, the byte value 0x05 is repeated five times after the plain text. 
An error condition is present if the padding doesn't match the syntax of the used padding scheme. A padding oracle is present if an application leaks this specific padding error condition for encrypted data provided by the client. This can happen by exposing exceptions (e.g. BadPaddingException in Java) directly, by subtle differences in the responses sent to the client or by another side-channel like timing behavior. 
Certain modes of operation of cryptography allow bit-flipping attacks, where flipping of a bit in the cipher text causes that the bit is also flipped in the plain text. Flipping a bit in the n-th block of CBC encrypted data causes that the same bit in the (n+1)-th block is flipped in the decrypted data. The n-th block of the decrypted cipher text is garbaged by this manipulation. 
The padding oracle attack enables an attacker to decrypt encrypted data without knowledge of the encryption key and used cipher by sending skillful manipulated cipher texts to the padding oracle and observing of the results returned by it. This causes loss of confidentiality of the encrypted data. E.g. in the case of session data stored on the client side the attacker can gain information about the internal state and structure of the application. 
A padding oracle attack also enables an attacker to encrypt arbitrary plain texts without knowledge of the used key and cipher. If the application assumes that integrity and authenticity of the decrypted data is given, an attacker could be able to manipulate internal session state and possibly gain higher privileges. 

|

테스트 방법
==========================================================================================

Black Box Testing
------------------------------------------------------------------------------------------

패딩 오라클 취약점 테스트
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

우선 패딩 오라클을 위한 입력 가능한 포인트를 확인해야합니다.

일반적으로 다음 조건이 충족되어야 합니다.

1. 데이터가 암호화되어 있어야 합니다. 좋은 후보로는 랜덤하게 나타나는 값입니다.
2. 블록 암호가 사용되어야 합니다. 복호화된 암호문의 길이가 8 또는 16 바이트와 같이 암호문 블록 사이즈 배수여야 합니다. 다른 암호문은 길이의 공약수를 공유할 수 있어야 합니다.

**예제**

.. code-block:: html
    
    Dg6W8OiWMIdVokIDH15T/A==
    |0e 0e 96 f0 e8 96 30 87 55 a2 42 03 1f 5e 53 fc|

만약 입력할 수 있는 부분이 확인되면, 암호화된 값을 비트단위로 조작한 다음 어플리케이션 행위를 확인해야합니다.
일반적으로 Base64로 인코딩된 값은 암호문 앞에 초기 벡터를 포함할 것입니다.
평문 텍스트 p와 암호문의 블록사이즈 n이 주어지고, 블록수는 b는 ceil(length(b)/n)이 될 것입니다.
암호화된 문자열 길이는 초기 벡터때문에 y=(b+1)*n이 될 것입니다.

To verify the presence of the oracle, decode the string, flip the last bit of the second-to-last block b-1 (the least significant bit of the byte at y-n-1), re-encode and send. 
Next, decode the original string, flip the last bit of the block b-2 (the least significant bit of the byte at y-2*n-1), re-encode and send. 
If it is known that the encrypted string is a single block (the IV is stored on the server or the application is using a bad practice hard-coded IV), several bit flips must be performed in turn. 
An alternative approach could be to prepend a random block, and flip bits in order to make the last byte of the added block take all possible values (0 to 255). 

The tests and the base value should at least cause three different states while and after decryption: 
 
- Cipher text gets decrypted, resulting data is correct. 
- Cipher text gets decrypted, resulting data is garbled and causes some exception or error handling in the application logic. 
- Cipher text decryption fails due to padding errors. 

Compare the responses carefully. Search especially for exceptions and messages which state that something is wrong with the padding. If such messages appear, the application contains a padding oracle. If the three different states described above are observable implicitly (different error messages, timing side-channels), there is a high probability that there is a padding oracle present at this point. Try to perform the padding oracle attack to ensure this. 

**예제**
 
- ASP.NET throws "System.Security.Cryptography.Cryptographic Exception: Padding is invalid and cannot be removed." if padding of a decrypted cipher text is broken. 
- In Java a javax.crypto.BadPaddingException is thrown in this case. 
- Decryption errors or similar can be possible padding oracles. 


**예상 결과**

A secure implementation will check for integrity and cause only two responses: ok and failed. There are no side channels which can be used to determine internal error states. 


Grey Box Testing 
------------------------------------------------------------------------------------------

패딩 오라클 취약점 테스트
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Verify that all places where encrypted data from the client, that should only be known by the server, is decrypted. The following conditions should be met by such code: 

1. The integrity of the cipher text should be verified by a secure mechanism, like HMAC or authenticated cipher operation modes like GCM or CCM. 
2. All error states while decryption and further processing are handled uniformly. 

|

Tools 
==========================================================================================

- PadBuster: https://github.com/GDSSecurity/PadBuster 
- python-paddingoracle: https://github.com/mwielgoszewski/py thon-paddingoracle 
- Poracle: https://github.com/iagox86/Poracle 
- Padding Oracle Exploitation Tool (POET): http://netifera.com/research/

**예제**

- Visualization of the decryption process - http://erlend.oftedal.no/blog/poet/ 

|

References 
==========================================================================================

Whitepapers 
-----------------------------------------------------------------------------------------

- Wikipedia - Padding oracle attack: http://en.wikipedia.org/wiki/Padding_oracle_attack 
- Juliano Rizzo, Thai Duong, "Practical Padding Oracle Attacks": http://www.usenix.org/event/woot10/tech/full_papers/Rizzo.pdf 

|