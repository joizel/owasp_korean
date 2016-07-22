==========================================================================================
OTG-INFO-007 (애플리케이션을 통해 경로 실행 맵)
==========================================================================================

|

개요
==========================================================================================

보안 검사를 시작하기 전에, 어플리케이션의 구조를 이해하는 것은 중요합니다.
어플리케이션의 레이아웃의 철저한 이해 없이, 완벽하게 테스트 될 수 없을 것입니다.

|

테스트 목적
==========================================================================================

대상 어플리케이션을 지도화하고 주요 워크플로우를 이해하기
Map the target application and understand the principal workflows.

|

테스트 방법
==========================================================================================

In black box testing it is extremely difficult to test the entire code base. 
Not just because the tester has no view of the code paths through the application, but even if they did, to test all code paths would be very time consuming. 
One way to reconcile this is to document what code paths were discovered and tested.

There are several ways to approach the testing and measurement of code coverage:

- Path - test each of the paths through an application that includes combinatorial and boundary value analysis testing for each decision path. While this approach offers thoroughness, the number of testable paths grows exponentially with each decision branch.
- Data flow (or taint analysis) - tests the assignment of variables via external interaction (normally users). Focuses on mapping the flow, transformation and use of data throughout an application.
- Race - tests multiple concurrent instances of the application manipulating the same data.

The trade off as to what method is used and to what degree each method is used should be negotiated with the application owner. Simpler approaches could also be adopted, including asking the application owner what functions or code sections they are particularly concerned about and how those code segments can be reached.

|

Black Box Testing
-----------------------------------------------------------------------------------------

어플리케이션 관리자에게 코드 커버리지를 설명하기 위해, 테스터는 스프레드시트로 시작하고 어플리케이션을 스파이더링에 의해 발견된 모든 링크를 문서화할 수 있습니다.

그런 다음 테스터는 응용 프로그램에서 결정 포인트 더 자세히보고 중요한 코드 경로가 발견 얼마나 많은 조사 할 수 있습니다.
이 후 발견 된 경로의 URL을, 산문 및 스크린 샷 설명과 함께 스프레드 시트에 문서화되어야한다.

Gray/White Box testing
-----------------------------------------------------------------------------------------

Ensuring sufficient code coverage for the application owner is far
easier with the gray and white box approach to testing. Information
solicited by and provided to the tester will ensure the minimum requirements
for code coverage are met.

Example
-----------------------------------------------------------------------------------------

**Automatic Spidering**

자동 스파이더는 지정한 웹사이트에 새로운 리소스를 자동으로 발견하는 데 사용하는 툴입니다.
스파이더 사직되는 방법에 따라 씨드라고 불리는 방문할 URL 리스트로 시작됩니다.
수많은 스파이더링 툴이 있으며, 이 문서에서는 Zed Attack Proxy를 사용합니다.

|

ZAP offers the following automatic spidering features, which can be selected based on the tester’s needs:

- Spider Site - The seed list contains all the existing URIs already found for the selected site.
- Spider Subtree - The seed list contains all the existing URIs already found and present in the subtree of the selected node.
- Spider URL - The seed list contains only the URI corresponding to the selected node (in the Site Tree).
- Spider all in Scope - The seed list contains all the URIs the user has selected as being ‘In Scope’.


Tools
==========================================================================================

- Zed Attack Proxy (ZAP)
- List of spreadsheet software
- Diagramming software

|

References
==========================================================================================

Whitepapers
-----------------------------------------------------------------------------------------

- http://en.wikipedia.org/wiki/Code_coverage

|