============================================================================================
OTG-IDENT-003 (계정 공급 과정 테스트)
============================================================================================

|

개요
============================================================================================

The provisioning of accounts presents an opportunity for an attacker to create a valid account without application of the proper identification and authorization process. 

|

테스트 목적
============================================================================================

Verify which accounts may provision other accounts and of what type. 

|

테스트 방법 
============================================================================================

Determine which roles are able to provision users and what sort of accounts they can provision. 
- Is there any verification, vetting and authorization of provisioning requests? 
- Is there any verification, vetting and authorization of de-provisioning requests? 
- Can an administrator provision other administrators or just users? 
- Can an administrator or other user provision accounts with privileges greater than their own? 
- Can an administrator or user de-provision themselves? 
- How are the files or resources owned by the de-provisioned user managed? Are they deleted? Is access transferred? 

**예제**

In WordPress, only a user's name and email address are required to provision the user, as shown below: 

|

Tools 
============================================================================================

While the most thorough and accurate approach to completing this test is to conduct it manually, HTTP proxy tools could be also useful. 

|