# REST API

- REST(REpresentational State Transfer)는 2000년도에 Roy Fielding의 박사학위 논문에서 최초로 소개됨, 당시 웹(HTTP) 설계의 우수성에 비해 제대로 사용되지 못하는 점에 웹의 장점을 살릴 수 있는 아키텍처로 REST를 발표

- REST는 HTTP 기반으로 필요한 자원에 접근하는 방식을 정해놓은 아키텍처

- 여기서 말하는 자원은 저장된 데이터, 동영상, 문서, 이미지와 같은 파일, 서비스를 모두 포함한다.

- 그러므로 REST API는 REST를 통해 API를 구현한 것을 말한다.



# REST에는 4개의 속성이 존재

1. 서버에 있는 모든 resource는 각 resource 당 클라이언트가 바로 접근할 수 있는 고유 URI가 존재
2. 모든 요청은 클라이언트가 요청할 때마다 필요한 정보를 주기 때문에 서버에서는 세션 정보를 보관할 필요가 없다. 그렇기 때문에 서비스에 자유도가 높아지고 유연한 아키텍처 적응이 가능
3. HTTP 메소드를 사용, 모든 resource는 일반적으로 HTTP 인터페이스인 GET, POST, PUT, DELETE 4개의 메소드로 접근되어야 함
4. 서비스 내에 하나의 resource가 주변에 연관된 리소스들과 연결되어 표현이 되어야 한다.



# REST의 구성요소

## resource

REST에서는 자원에 접근할 때 URI(Uniform Resource Identifier)로 하게 된다.

URI는 자원의 위치를 나타내는 일종의 식별자



### URI 설계 규칙

1. `/` 쓰임새

   - 슬래시 구분자(`/`) 는 계층 관계를 나타내는 데 사용

   ```
   https://github.com/JeonChulMin/IT-Terms
   ```

   - 위 사이트에서 IT-Terms 정보를 찾을 때 github부터 하위 소속들을 거쳐 IT-Terms까지 도달하는 구조
   - 그렇기 때문에 URI 마지막 문자로 `/` 를 포함하지 않는다.

2. URI를 이루는 resource들은 동사보다는 명사로 이루어져야 한다.

   - resource 간의 관계를 표현하는 방법
     - /리소스명/리소스ID/관계가 있는 다른 리소스명
     - /users/{userid}/devices

3. URI에서는 `_`(언더바) 보다는 `-`(하이픈)을 권장

   - URI를 쉽게 읽고 해석하기 위해, 긴 URI 경로를 사용하게 된다면 하이픈을 사용해 가독성을 높일 수 있다.
   - 글꼴에 따라 다르지만 밑줄은 보기 어렵거나 밑줄 때문에 문자가 가려지기도 함
   - 이런 문제를 피하기 위해 언더바 대신 하이픈을 사용하는 것이 좋음

4. URI 경로에는 소문자가 적합

   - URI 경로에 대문자 사용은 되도록 피해야 한다.
   - 대소문자에 따라 다른 리소스로 인식하기 때문

5. 파일 확장자는 URI에 포함시키지 않는다.

   ```
   https://github.com/JeonChulMin/IT-Terms/images/DB.jpg
   ```

   - REST API에서는 메시지 바디 내용의 포맷을 나타내기 위한 파일 확장자를 URI 안에 포함시키지 않는다.
   - Accept header를 사용하도록 해야한다.

   ```
   GET / JeonChulMin/IT-Terms/images/DB HTTP/1.1 HOST: github.com
   Accept: image/jpg
   ```

   

## HTTP 메소드

- 자원 접근 시 어떤 성격의 요청인지 HTTP 메소드가 알려준다.
- ex) GET, POST, PUT, DELETE

| 메소드 | 설명                              |
| ------ | --------------------------------- |
| GET    | 해당 resource를 조회              |
| POST   | 해당 URI를 요청하면 resource 생성 |
| PUT    | 해당 resource 수정                |
| DELETE | 해당 resource 삭제                |



## Message

- 메시지는 HTTP header, body, 응답상태코드로 구성되어 있으며 header와 body에 포함된 메시지는 메시지를 처리하기 위한 충분한 정보를 포함

### Body

- 자원에 대한 정보를 전달
- ex) 데이터 포맷(JSON, XML, 사용자 정의 포맷)

### Header

- HTTP body에 어떤 포맷으로 데이터가 담겼는지 정의
- 요청 HTTP header는 `Accept` 항목으로 응답 HTTP 헤더는 `Content-type`으로 컨텐츠 타입을 설명

### 응답상태코드

- 응답상태코드를 통해 리소스 요청에 대한 응답을 할 수 있다.

| 응답상태코드          | 설명                                                  |
| --------------------- | ----------------------------------------------------- |
| 1xx (정보)            | 요청을 받았으며 프로세스를 계속 진행                  |
| 2xx (성공)            | 요청을 성공적으로 받았으며 인식했고 수용한 경우       |
| 3xx (리다이렉션)      | 요청 완료를 위해 추가 작업 조치가 필요한 경우         |
| 4xx (클라이언트 오류) | 요청의 문법이 잘못되었거나 요청을 처리할 수 없을 경우 |
| 5xx (서버 오류)       | 서버가 명백히 유효한 요청에 대해 충족을 실패할 경우   |



# REST API 장/단점

### 장점

- 언어와 플랫폼에 독립적
- SOAP보다 개발이 쉽고 단순
- REST가 지원하는 프레임워크나 언어 등 도구들이 없어도 구현 가능
- 기존 웹 인프라를 사용 가능, HTTP를 그대로 사용하기 때문



### 단점

- HTTP 프로토콜만 사용이 가능
- p2p 통신 모델을 가정했기 때문에 둘 이상을 대상으로 하는 분산화경에는 유용하지 않음
- 보안, 정책 등에 대한 표준이 없기 때문에 관리가 어려움



# References

- https://medium.com/@dydrlaks/rest-api-3e424716bab

- https://brunch.co.kr/@leedongins/65

  