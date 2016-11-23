============================================================================================
OTG-BUSLOGIC-001 (비즈니스 로직 데이터 인증 테스트)
============================================================================================

|

개요
============================================================================================

어플리케이션은 논리적으로 프론트 엔드에서 뿐만 아니라 시스템 어플리케이션의 서버 측에서도 유효한 데이터 입력이 되도록 해야합니다. 

서버에서 데이터를 로컬에서만 확인하면 프록시를 통한 인젝션이나 다른 시스템 이관시 응용 프로그램이 취약해질 수 있습니다.

이것은 범위 값 분석(BVA)을 수행하는 것과는 조금 다르며, 대부분의 경우 엔트리 포인트에서 간단히 확인할 수는 없지만 대개 다른 시스템을 확인해야합니다.

비즈니스 데이터 유효성 검사 관련 취약점은 응용 프로그램에 따라 다르며, 단순한 비즈니스 논리 워크 플로우를 위반하는 것이 아니라, 논리 데이터에 더 관심을 두기 때문에 요청 변조 관련 취약점과는 다릅니다.

응용 프로그램의 프론트 엔드와 백 엔드는 사용하고 전달되는 데이터가 논리적으로 유효하다는 것을 검증하고 확인해야 합니다.

사용자가 응용 프로그램에 유효한 데이터를 제공하더라도, 비즈니스 논리로 인해 응용 프로그램이 상황에 따라 다르게 동작할 수 있습니다.

|

예제
============================================================================================

|

예제 1
-----------------------------------------------------------------------------------------

사용자가 카펫을 주문 할 수 있는 멀티미디어 전자상 거래 사이트를 운영한다고 가정합니다.
사용자는 카펫, 사이즈, 지불 방법을 선택하고, 프론트 엔드 어플리케이션은 모든 입력 정보가 정확하고 유효한지 확인합니다.
But, the business logic in the background has two paths, if the carpet is in stock it is directly shipped from your warehouse, but if it is out of stock in your warehouse a call is made to a partner's system and if they have it in-stock they will ship the order from their warehouse and reimbursed by them.
What happens if an attacker is able to continue a valid in-stock transaction and send it as out-of-stock to your partner? 
What happens if an attacker is able to get in the middle and send messages to the partner warehouse ordering carpet without payment? 

|

예제 2
-----------------------------------------------------------------------------------------

Many credit card systems are now downloading account balances nightly so the customers can check out more quickly for amounts under a certain value. The inverse is also true. 
If I pay my credit card off in the morning I may not be able to use the available credit in the evening. 
Another example may be if I use my credit card at multiple locations very quickly it may be possible to exceed my limit if the systems are basing decisions on last night's data. 

|

테스트 방법
============================================================================================

일반 테스트 방법
-----------------------------------------------------------------------------------------

- 프로젝트 문서를 검토하고 데이터 입력 포인트 또는 시스템 또는 소프트웨어 사이에 입력할 수 있는 있는 포인트
를 찾아 테스트 공격 진행
- 어플리케이션 및 시스템에서 논리적으로 잘못된 데이터 입력 검색

|

구체적인 테스트 방법
-----------------------------------------------------------------------------------------

- "valid" 값 허용되는 지 확인하기 위해 어플리케이션의 프론트 엔트 GUI 기능 유효성 테스트 수행
- Using an intercepting proxy observe the HTTP POST/GET look ing for places that variables such as cost and quality are passed. Specifically, look for "hand-offs" between application/systems that may be possible injection of tamper points. 
- Once variables are found start interrogating the field with log ically "invalid" data, such as social security numbers or unique identifiers that do not exist or that do not fit the business logic. This testing verifies that the server functions properly and does not accept logically invalid data them. 

|

관련 테스트 케이스
============================================================================================

- 모든 입력 유효성 검사 테스트 케이스
- 계정 리스트 및 추측 가능한 사용자 테스트 (OTG-IDENT-004) 
- 세션 관리 스키마 우회 테스트 (OTG-SESS-001) 
- 노출 세션 변수 테스트 (OTG-SESS-004) 

|

도구 
============================================================================================

- OWASP Zed Attack Proxy (ZAP)

|

참고 문헌 
============================================================================================

- Beginning Microsoft Visual Studio LightSwitch Development
- http://books.google.com/books?id=x76L_kaTgdEC&pg=PA280&lpg=PA280&dq=business+logic+example+valid+data+example&source=bl&ots=GOfQ-7f4Hu&sig=4jOejZVligZOrvjBFRAT4-jy8DI&hl=en&sa=X&ei=mydYUt6qE-OX54APu7IDgCQ&ved=0CFIQ6AEwBDgK#v=onepage&q=business%20logic%20example%20valid%20data%20example&f=false 

|

개선 방안 
============================================================================================

응용 프로그램/시스템은 응용 프로그램이나 시스템의 모든 입력 및 전달 지점에서 "논리적으로 유효한" 데이터 만 받아 들여지고, 시스템에 일단 입력되면 데이터가 단순하게 신뢰되지 않도록 해야 합니다.

|