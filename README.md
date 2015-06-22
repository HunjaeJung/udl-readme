# UDL(Uniform Deeplink) Engine for deeplink.org

## 요약

- [deeplink.org](http://deeplink.org)에서 사용하고 있는 Open Deeplink Engine 입니다. 
- 이 엔진은 다음과 같은 역할을 담당합니다.
	1. [모바일 어플리케이션 딥링크 및 마켓 주소 등록/관리](#register-deeplink)
	2. [각각의 딥링크에 해당하는 자원(컨텐츠) 맵핑/색인화](#app-indexing)
	3. [접속 환경과 무관하게 사용자 요청을 핸들링하는 UMDE(Unificated Mobile Deeplink End-Point)](#umde)
	4. [검색 요청에 따라 UMDE와 컨텐츠 정보를 제공](#search)
	5. [딥링크를 통해 접속하는 트래픽에 대한 분석](#analyze)

## 세부 구조 및 사용법


### <a name="register-deeplink"></a>모바일 어플리케이션 딥링크 및 마켓 주소 등록/관리

### <a name="app-indexing"></a>각각의 딥링크에 해당하는 자원(컨텐츠) 맵핑/색인화

### <a name="umde"></a>접속 환경과 무관하게 사용자 요청을 핸들링하는 UMDE(Unificated Mobile Deeplink End-Point)

### <a name="search"></a>검색 요청에 따라 UMDE와 컨텐츠 정보를 제공

- 사용자의 검색 요청(Query)에 맞는 UMDE와 컨텐츠 정보를 제공합니다.
- 특정 모바일 앱, 혹은 전체 모바일 앱에 대해서 검색이 가능합니다.
- 사용자는 REST API를 통해 검색 요청을 보낼 수 있으며, GET 및 POST 방식을 지원합니다.
- 한글 및 영문의 경우 자동으로 형태소 분석 후 단어 단위 검색이 시행됩니다.
- GET 방식
	- GET 방식은 단순히 '검색어'로만 검색 요청을 보낼 때 Query String을 사용하여 편리하게 이용할 수 있습니다.
    ```
    [GET]
    [URL]
    http://engine.deeplink.org/[app_name]/_search?q=[query]
    ```
    - ```[app_name]``` 부분은 검색하고자 하는 모바일 어플리케이션의 등록된 이름을 삽입하면 됩니다.
    - 다음과 같이 ```[app_name]``` 부분이 생략되는 경우는 전체 모바일 앱에 대해 검색을 시행합니다. 권한에 따라 검색 요청이 거절될 수도 있습니다.
    ```
    [GET]
    [URL]
    http://engine.deeplink.org/_search?q=[query]
	```
    - ```[query]``` 에는 검색하고자 하는 텍스트를 입력하면 됩니다. 단, 영문이 아닌 경우에는 반드시 URL Encoding을 거쳐 보내야 안전하게 처리됩니다.
    - GET 방식은 Request Body를 허용하지 않습니다.
- POST 방식
	- POST 방식은 다양한 검색 조건을 넣어 검색을 요청할 때 사용합니다.
	- GET 방식과 동일하게 ```[app_name]```이 없으면 전체 모바일 앱에 대한 검색을 시행합니다.
	- 다음과 같이 JSON 형식의 Request Body를 이용하여 검색을 요청합니다.
	```
    [POST]
    [HEADER]
    Content-type: application/json
    [URL]    
	http://engine.deeplink.org/[app_name]/_search
    [BODY]
    {
    	"q": {
	        "query": String,
        	"fields": [String]
        },
        "sort": [{
        	FIELD_NAME: TYPE
        }],
        "limit": Number
    }
	```
    - 검색 옵션 세부 설명
    	- q
	    	- query (필수)
	    	검색어를 입력합니다. 한글 및 영문의 경우 자동으로 형태소 분석 후 단어 단위로 검색을 시행합니다. 검색 요청에 반드시 포함되어야 합니다.
    		- fields (옵션)
    		```query```에 넣은 내용으로 실제 검색을 시행할 컨텐츠의 필드명을 String Array로 입력합니다. 일반적인 SQL문의 'WHERE'절이라고 생각하면 좋습니다. 특정 앱에 어떠한 컨텐츠 필드가 있는지 알 수 없을 때는 [_getContentFields](#_getContentFields) 섹션을 참고하십시오. 이 부분이 입력되지 않으면 모든 필드에서 검색을 수행합니다.
            ```
            [EXAMPLE/'아이유'라는 단어를 '가수' 필드에서 검색하는 예]
            "q": {
            	"query": "아이유",
                "fields": "artist"
            }
			```
        - sort (옵션)
       	검색 결과를 정렬하고자 할 때에는 아래의 예와 같이 데이터를 넣어주십시오. 일반적인 SQL문의 'ORDER BY'절을 생각하시면 좋습니다. 배열에 입력된 순서대로 정렬을 진행합니다. 입력되지 않는 경우 검색어에 가장 정확한 결과 순서(```_score``` 내림 차순)로 정렬됩니다.
        ```
        [EXAMPLE/인기도 내림차순 정렬 후, 아티스트명 오름차순 정렬을 진행하는 예]
        "sort": [
        	{"popular": "desc"},
            {"artist": "asc"}
        ]
		```
        ```STRING_TYPE```부분은 ```"ASC"```(오름차순 정렬)와 ```"DESC"```(내림차순 정렬)를 지원합니다.
        - limit (옵션)
        검색 결과의 갯수를 제한하고자 할 때 사용합니다. 일반적인 SQL문의 'LIMIT' 혹은 'TOP'을 생각하시면 좋습니다. 입력하지 않는 경우 기본값은 10개 입니다.
        ```
        [EXAMPLE/최대 검색 결과를 100개로 제한하는 예]
        "limit": 100
		```
    - 예제
    위의 옵션을 조합하면 다음과 같은 검색 Request Body가 생성됩니다.
    ```
    [EXAMPLE/'beat' 앱에서 아이유'라는 단어를 '가수' 필드에서 검색. 인기도 내림차순 정렬 후, 아티스트명 오름차순 정렬. 최대 검색 결과를 100개로 제한.]
    [POST]
    [HEADER]
    Content-type: application/json
    [URL]    
	http://engine.deeplink.org/beat/_search
    [BODY]
    {
    	"q": {
	        "query": "아이유",
        	"fields": ["artist"]
        },
        "sort": [
        	{"popular": "desc"},
            {"artist": "asc"}
        ],
        "limit": 100
    }
	```            
- 검색 결과
	- Response Code
	검색 결과에 따라 HTTP Response Code를 다음과 같이 제공합니다. IANA에서 관리하는 [HTTP 상태 코드 레지스트리](http://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml)와 거의 유사하거나 같은 의미를 내포하고 있습니다.
		- 200: 성공적으로 검색을 완료하였습니다. (Your request successfully finished)
		- 204: 검색 결과가 존재하지 않습니다. (No matched data)
		- 400: 검색 쿼리가 적절하지 않습니다. (Bad querying request)
		- 401: 권한 없음: Auth. Header를 전송하지 않았거나 그러한 Auth. Token이 없습니다. 혹은 요청한 정보에 대한 접근 권한이 없습니다. (Incorrect request token or No Permission)
		- 403: 접근이 금지되었습니다. 문의바랍니다. (Forbidden)
		- 500: 내부 에러입니다. 문의바랍니다. (Internal server error)
	- Response Body
	검색에 성공한 경우, 다음의 예제와 같은 검색 결과를 돌려줍니다.
    ```
    
	```
- <a name="_getContentFields"></a>해당 앱의 Field 정보 가져오기
	- Not implemented yet!

### <a name="analyze"></a>딥링크를 통해 접속하는 트래픽에 대한 분석


Copyrights (C) 2015 [Team Teheran Slippers](mailto:jeegle.team@gmail.com)
