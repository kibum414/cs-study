## 출처 (Origin) 란 ?

- **URL** : https://domain.com:3000/users?sort=asc&page=1#foo
    - **Protocol (Scheme)** : http, https
    - **Host** : 사이트 도메인
    - **Port** : 포트 번호
    - **Path** : 사이트 내부 경로
    - **Query String** : 요청의 Key 와 Value
    - **Fragment** : 해시 태그
- **출처 (Origin)** : Protocol, Host, Port 를 모두 합친 URL
    
    ```jsx
    console.log(location.origin);
    
    "https://www.naver.com" // 포트 번호 80번은 생략됨
    ```
    
- 출처에 포트 번호가 명시적으로 포함되어 있다면 포트 번호까지 모두 일치해야 같은 출처로 인정한다.

<br /><br /><br />

```
😎 웹 생태계에는 다른 출처로의 리소스 요청을 제한하는 것과 관련된 두 가지 정책(CORS, SOP)이 존재한다.
```

<br />

## SOP (Same-Origin Policy, 동일 출처 정책)

- “같은 출처에서만 리소스를 공유할 수 있다” 라는 규칙을 가진 정책
- 다른 Origin 으로 요청을 보낼 수 없도록 금지하는 브라우저의 기본적인 보안 정책

<br />

### 동일 출처 정책이 필요한 이유

> 출처가 다른 두 어플리케이션이 자유롭게 소통할 수 있다면, 해커가 CSRF(Cross-Site Request Forgery)나 XSS(Cross-Site Scripting)와 같은 방법을 이용해서 해커가 심어놓은 코드를 실행하여 개인 정보를 가로채기가 쉬워진다.
> 

<br />

### 같은 출처와 다른 출처의 구분

두 URL 의 구성 요소 중 `Scheme` , `Host` , `Port` 세 가지만 동일하면 출처가 같다고 판단한다.

- `https://domain.com` 과 같은 출처로 인정되는 경우
    
    
    | URL |  같은 출처 여부 | 이유 |
    | --- | --- | --- |
    | https://domain.com/about | O | 스킴, 호스트, 포트 같음 |
    | https://domain.com/about?q=하이 | O | 스킴, 호스트, 포트 같음 |
    | https://user:password@domain.com | O | 스킴, 호스트, 포트 같음 |
    | http://domain.com | X | 스킴이 다름 |
    | https://api.domain.com | X | 호스트가 다름 |
    | https://domain.com:8000 | ? | 브라우저의 구현에 따라 다름 |

출처를 비교하는 로직은 서버가 아닌 브라우저에 구현되어 있다.

만약 CORS 정책을 위반하는 리소스 요청을 하더라도, 해당 서버가 같은 출처에서 보낸 요청만 받겠다는 로직을 갖고 있는 경우가 아니라면 서버는 정상적으로 응답을 하고, 이후 브라우저가 이 응답을 분석해서 CORS 정책 위반이라고 판단되면 그 응답을 사용하지 않고 버리는 순서로 동작한다.

결론적으로, CORS 는 브라우저의 구현 스펙에 포함되는 정책이기 때문에 브라우저를 통하지 않고 서버 간 통신을 할 때는 이 정책이 적용되지 않는다.

<br />

### SOP 의 예외 상황 (다른 출처의 리소스 요청이라도 허용하는 경우)

- `<script>` 태그로 JavaScript 를 실행하는 경우
- 이미지를 렌더링 하는 경우
- `<link>` 태그로 스타일 시트 파일을 불러오는 경우
- HTML 문서를 화면에 보여주는 경우
- **CORS 정책을 지키는 요청의 경우**

> CORS 요청을 제외한 나머지 경우는 요청 시 `Sec-Fetch-Mode` 헤더의 값을 `no-cors` 로 설정한다.
이는 서버가 보내준 응답에 대해 CORS 정책을 검사하지 않게 하는 대신 해당 응답을 JavaScript 단에서 읽을 수 없도록 한다.
> 

<br /><br /><br />

## CORS (Cross-Origin Resource Sharing, 교차 출처 자원 공유)

- 브라우저에서 다른 출처 간의 리소스 공유에 대한 허용/비허용 정책
- SOP 정책을 위반해도 CORS 정책에 따르면 다른 출처의 리소스라도 허용한다.
- 브라우저가 자신이 보낸 요청 및 서버로부터 받은 응답의 데이터가 CORS 정책을 지키는지 검사하여 안전한 요청을 보낸 건지 검사한다.
    - 서버가 정상적으로 응답을 해줬더라도 안전한 요청이 아니라고 판단되면 해당 응답을 버림
    - 그렇기 때문에 서버 간 통신에서는 이러한 정책이 전혀 적용되지 않음

<br />

### CORS 동작 원리

1. 클라이언트가 HTTP 프로토콜을 사용하여 다른 출처의 리소스 요청
    - 브라우저는 요청 헤더에 `Origin` 이라는 필드에 요청을 보내는 출처를 함께 담아 보낸다.
        
        ```jsx
        Origin: http://localhost:3000
        ```
        
2. 서버는 응답 헤더에 `Access-Control-Allow-Origin` 을 담아 클라이언트로 전달
    - 서버가 이 요청에 대한 응답을 할 때 응답 헤더에  `Access-Control-Allow-Origin` 라는 필드를 추가하고 값으로 “이 리소스를 접근하는 것이 허용된 출처” 를 내려준다.
        
        ```jsx
        Access-Control-Allow-Origin: http://localhost:3000
        ```
        
3. 클라이언트에서 자신이 보냈던 요청의 `Origin` 과 서버가 보내준 `Access-Control-Allow-Origin` 을 비교
    - 유효하지 않다면 그 응답을 사용하지 않고 버린다.
    - 위의 경우에는 둘다 `http://localhost:3000` 으로 일치하기 때문에 유효한 응답으로 다른 출처의 리소스를 문제 없이 가져오게 된다.

즉, CORS 요청을 위해서는 서버에서 응답의 `Access-Control-Allow-Origin` 헤더에 허용되는 `Origin` 의 목록 혹은 와일드카드(`*` : 모든 Origin 허용)를 설정해주면 된다.

<br /><br /><br />

## CORS 작동 방식 3가지 시나리오

### 예비 요청 (Preflight Request)

> 먼저 **예비 요청**을 보내 서버와 잘 통신되는지 확인한 후 **본 요청**을 보내는 방식
> 
- 예비 요청의 역할은 본 요청을 보내기 전에 브라우저 스스로 안전한 요청인지 미리 확인하는 것
- 브라우저가 예비 요청을 보내는 것을 **Preflight** 라고 부르며, 이 예비 요청의 HTTP 메소드로 GET 이나 POST 가 아닌 OPTIONS 라는 요청이 사용된다는 것이 특징

![예비 요청](https://user-images.githubusercontent.com/76759852/213872114-a8adf3c6-d10b-4869-bcc7-e3eedb5f5862.png)

1. 자바스크립트의 `fetch()` 메소드를 통해 리소스를 받아오려고 한다.
2. 브라우저는 서버에 HTTP OPTIONS 메소드로 예비 요청(Preflight)을 먼저 보낸다.
    - `Origin` 헤더에 자신의 출처를 넣는다.
    - `Access-Control-Request-Method` 헤더에 실제 요청에 사용할 메소드를 설정한다.
    - `Access-Control-Request-Headers` 헤더에 실제 요청에 사용할 헤더들을 설정한다.
3. 서버는 이 예비 요청에 대한 응답으로 어떤 것을 허용하고 어떤 것을 금지하고 있는지에 대한 헤더 정보를 담아서 브라우저로 보내준다.
    - `Access-Control-Allow-Origin` 헤더에 허용되는 Origin 들의 목록 혹은 와일드카드(`*`)를 설정한다.
    - `Access-Control-Allow-Methods` 헤더에 허용되는 메소드들의 목록 혹은 와일드카드(`*`)를 설정한다.
    - `Access-Control-Allow-Headers` 헤더에 허용되는 헤더들의 목록 혹은 와일드카드(`*`)를 설정한다.
    - `Access-Control-Max-Age` 헤더에 해당 예비 요청이 브라우저에 캐시될 수 있는 시간을 초 단위로 설정한다.
4. 브라우저는 응답의 정보를 자신이 전송한 요청의 정보와 비교하여 해당 요청이 안전한지 확인하고 본 요청을 보내게 되는데, 이 때는 `Access-Control-Request-XXX` 형태의 헤더는 보내지 않는다.
5. 서버가 본 요청에 대한 응답을 하면 최종적으로 이 응답 데이터를 자바스크립트로 넘겨준다.

![예비 요청 2](https://user-images.githubusercontent.com/76759852/213872117-c4db02d6-2cfe-4216-a1ef-5cbdc8813c21.png)

<br />

### 단순 요청 (Simple Request)

> **예비 요청(Preflight)을 생략하고 바로 서버에 직행으로 본 요청**을 보낸 후, 서버가 이에 대한 응답의 헤더에 `Access-Control-Allow-Origin` 헤더를 보내주면 브라우저가 CORS 정책 위반 여부를 검사하는 방식
> 
- 대표적으로 아래 3가지 경우를 만족할 때만 예비 요청을 생략할 수 있다.
    1. 요청의 메소드가 GET, HEAD, POST 중 하나여야 한다.
    2. User Agent 가 자동으로 설정한 헤더를 제외하면, `Accept` , `Accept-Language` , `Content-Language` , `Content-Type` , `DPR` , `Downlink` , `Save-Data` , `Viewport-Width` , `Width` 헤더일 경우에만 적용된다.
    3. Content-Type 헤더가 `application/x-www-form-urlencoded` , `multipart/form-data` , `text/plain` 중 하나여야 한다.
- 대부분 HTTP API 요청은 `text/xml` 이나 `application/json` 으로 통신하기 때문에 3번째 조건에 위반된다. 따라서 대부분의 API 요청은 예비 요청으로 이루어진다라고 생각하자.

![단순 요청](https://user-images.githubusercontent.com/76759852/213872125-6ade74f9-8117-41c3-8ac1-928e9feba3b7.png)

<br />

### 인증된 요청 (Credentialed Request)

> 클라이언트에서 서버에게 **자격 인증 정보(Credential)**를 실어 요청할 때 사용되는 방식
> 
- **자격 인증 정보 (Credential)** : 쿠키(Cookie) 혹은 Authorization 헤더에 설정하는 토큰 값 등
- **인증 정보를 함께 보내야 하는 요청 (Credential Request)** 이라면 별도로 따라줘야 하는 CORS 정책이 존재
    1. 클라이언트에서 인증 정보를 보내도록 설정하기
        - 기본적으로 브라우저가 제공하는 요청 API 들은 별도의 옵션 없이 브라우저의 쿠키와 같은 인증과 관련된 데이터를 함부로 요청 데이터에 담지 않도록 되어 있음
        - 요청에 인증과 관련된 정보를 담을 수 있게 해주는 옵션이 `credentials` 옵션
        - `credentials` 옵션에는 3가지 값을 사용할 수 있음
            
            
            | 옵션 값 | 설명 |
            | --- | --- |
            | same-origin (기본 값) | 같은 출처 간 요청에만 인증 정보를 담을 수 있다. |
            | include | 모든 요청에 인증 정보를 담을 수 있다. |
            | omit | 모든 요청에 인증 정보를 담지 않는다. |
        - `XMLHttpRequest` , jQuery 의 `ajax` , 또는 `axios` 사용 : `withCredentials` 옵션을 `true` 로 설정
        - `fetch` API 사용 : `credentials` 옵션을 `include` 로 설정
        - 이러한 별도의 설정을 해주지 않으면 쿠키 등의 인증 정보는 절대로 자동으로 서버에게 전송되지 않음
    2. 서버에서 인증된 요청에 대한 헤더 설정하기
        - 위와 같은 설정을 통해 인증 정보를 포함한 요청을 보내면 서버는 이러한 요청에 대해 일반적인 CORS 요청과는 다르게 대응해줘야 함
            - `Access-Control-Allow-Origin`에는 와일드카드(`*`)가 아닌 분명한 Origin(명시적인 URL)으로 설정되어야 한다. (인증 정보는 민감한 정보이기 때문에 출처를 정확하게 설정해주어야 한다.)
            - 응답 헤더의 `Access-Control-Allow-Credentials` 항목을 `true` 로 설정해야 한다.
        - 그렇지 않으면 CORS 정책에 의해 응답이 거부됨

![인증된 요청](https://user-images.githubusercontent.com/76759852/213872134-9938b903-082c-4228-b656-f5214dec8f65.png)

<br /><br /><br />

## CORS 를 해결하는 방법

### 서버에서 `Access-Control-Allow-Origin`  세팅하기

- 직접 서버에서 HTTP 헤더 설정을 통해 출처를 허용하게 설정하는 가장 정석적인 해결책이다.
- 각 서버의 문법에 맞게 `Access-Control-Allow-Origin` 헤더를 추가해주면 된다.

### 프록시 사이트 이용하기

- **프록시 (Proxy)** : 클라이언트와 서버 사이의 중계 대리점
- 프론트에서 직접 서버에 리소스를 요청했는데 서버에서 따로 설정을 해주지 않아 CORS 에러가 뜬다면, **모든 출처를 허용한 서버 대리점**을 통해 요청하면 된다.

### Chrome 확장 프로그램 이용하기

- Allow CORS: Access-Control-Allow-Origin 크롬 확장 프로그램을 설치한다.
- 로컬(localhost) 환경에서 API 를 테스트할 경우 CORS 문제를 해결할 수 있다.
