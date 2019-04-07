---
title: "[HTTP] CORS(Cross Origin Resource Sharing)란?"
date: 2019-04-01 20:00:00
description: "CORS란 Cross Origin Resource Sharing의 약자로, 서로 다른 origin 간 HTTP 요청이 가능하도록 해주는 표준이다. 보안상의 이유로 브라우저에서는 스크립트 본문에서 작성된 cross-origin HTTP를 제한하고 있다."
categories: http
tags: http
---

## CORS(Cross Origin Resource Sharing) 정의

CORS란 Cross Origin Resource Sharing의 약자로, 서로 다른 origin 간 HTTP reqeust가 가능하도록 해주는 표준이다.

HTTP request는 기본적으로 Cross-Site HTTP Requests가 가능하다. 이는  `<img src="other-domain.com">`, `<script src="other-domain.com/script.js">`와 같이 다른 도메인에 존재하는 리소스(이미지, 스크립트 등)를 참조 할 수 있음을 뜻한다.

그러나, 보안상의 이유로 브라우저에서는 `<script></script>`로 둘러쌓인 내부에서 작성된 cross-origin HTTP를 제한하고 있다. 예를 들어 [XMLHttpRequest](https://developer.mozilla.org/ko/docs/XMLHttpRequest){:target="_blank"}나 [Fatch API](https://developer.mozilla.org/ko/docs/Web/API/Fetch_API){:target="_blank"}는 [same-origin Policy](https://developer.mozilla.org/ko/docs/Web/Security/Same-origin_policy){:target="_blank"}를 따르기 때문에 올바른 CORS header를 포함하지 않는 한 다른 origin에 request를 할 수 없다. 때문에 서버 개발자는 CORS을 이해하고 스펙을 따라 HTTP request에 응답을 해야한다.

![CORS principle](/asserts/images/img-cors-principle.png)
*출처: MDN - CORS*

이 글에서는 CORS 요정의 종류와 관련된 header 속성을 살펴 보도록 하자.

## CORS 요청의 종류

CORS는 다음과 같은 4개의 타입으로 구분된다. 브라우저가 요청 내용을 분석한 뒤 한가지 방식을 선택해 서버에 요청하기 때문에 개발자는 목적에 맞는 방식을 선택하고 조건에 맞춰 코딩해야 한다.

- [Simple requests](#simple-requests)
- [Preflighted requests](#preflighted-requests)
- [Requests with credentials](#requests-with-credentials)
- [Request without Credential](#request-without-credentials)

### Simple requests {#simple-requests}

Simple reqeusts는 문자 그대로 간단한 요청에 해당하며, 아래 조건을 모두 만족할 경우를 뜻한다.

- `GET`, `POST`, `HEAD` method만 허용된다.
- `User-Agent`가 자동으로 지정한 header(`Connection`, `User-Agent`등) 외에는 아래의 header 값만 수동으로 지정할 수 있다.
  - `Accept`
  - `Accept-Language`
  - `Content-Language`
  - `Content-Type`(다만, 아래 지정된 3가지에 한해 허용된다.)
  - `DPR`
  - `Downlink`
  - `Save-Data`
  - `Viewport-Width`
  - `Width`
- `Content-Type` header는 아래의 3가지만 허용된다.
  - `application/x-www-form-urlencoded`
  - `multipart/form-data`
  - `text/plain`
- 요청에 사용된 `XMLHttpRequestUpdate` 객체에 이벤트 리스너가 등록되어 있지 않아야 한다.
- `ReadableStream`가 request에 포함되지 않아야 한다.

![Simple requests diagram](/asserts/images/img-simple-request.png)
*Simple requests diagram*

#### Simple Reqeusts 예제

다음은 Simple Reqeusts의 예제이다.

##### Request

```http
GET /resources/access-control-with-credentials/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.5; en-US; rv:1.9.1b3pre) Gecko/20081130 Minefield/3.1b3pre
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Connection: keep-alive
Referer: http://foo.example/examples/credential.html
Origin: http://foo.example
Cookie: pageAccess=2
```

##### Response

```http
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 00:23:53 GMT
Server: Apache/2.0.61
Access-Control-Allow-Origin: *
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: application/xml
```

위의 예제는 `foo.example`에서 `bar.other`로 리소스를 요청하고 있다.
Simple request는 1번의 요청과 1번의 응답이 발생한다.

### Preflighted requests {#preflighted-requests}

Simple requests의 조건에 해당되지 않은 요청은 Preflighted requests로 분류된다. Preflighted requests은 이름에 걸맞게 예비 요청과 본 요청으로 나누어진다.

- 먼저 `OPTIONS`을 이용해 서버에 예비 요청을 보내 본 요청이 안전한지 검증한다(예비 요청은 브라우저가 알아서 진행하므로 개발자가 직접 구현하지 않아도 된다.).
- 그 이후 본 요청을 보내고, 서버는 이에 응답한다.

![Preflighted requests diagram](/asserts/images/img-preflight-request.png)
*Preflighted requests diagram*

#### Preflighted requests 예제

먼저 예비 요청에 대한 해더 정보이다.

##### Preflighted Request

```http
OPTIONS /resources/post-here/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.5; en-US; rv:1.9.1b3pre) Gecko/20081130 Minefield/3.1b3pre
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Connection: keep-alive
Origin: http://foo.example
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-PINGOTHER, Content-Type
```

##### Preflighted Response

```http
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://foo.example
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
Access-Control-Max-Age: 86400
Vary: Accept-Encoding, Origin
Content-Encoding: gzip
Content-Length: 0
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain
```

예비 요청에 대한 검증이 완료되면 본 요청을 수행한다.

##### Real Request

```http
POST /resources/post-here/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.5; en-US; rv:1.9.1b3pre) Gecko/20081130 Minefield/3.1b3pre
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Connection: keep-alive
X-PINGOTHER: pingpong
Content-Type: text/xml; charset=UTF-8
Referer: http://foo.example/examples/preflightInvocation.html
Content-Length: 55
Origin: http://foo.example
Pragma: no-cache
Cache-Control: no-cache
```

##### Real Response

```http
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:40 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://foo.example
Vary: Accept-Encoding, Origin
Content-Encoding: gzip
Content-Length: 235
Keep-Alive: timeout=2, max=99
Connection: Keep-Alive
Content-Type: text/plain
```

### Requests with credentials {#requests-with-credentials}

HTTP Cookie와 HTTP Authentication 정보를 인식할 수 있게 해주는 요청이다.
클라이언트는 요청 생성 시 반드시 `withCredentials = true;`로 지정해주어야 한다.

아래는 요청을 위한 스크립트 예시이다.

```js
var invocation = new XMLHttpRequest();
var url = 'http://bar.other/resources/credentialed-content/';
    
function callOtherDomain(){
  if(invocation) {
    invocation.open('GET', url, true);
    invocation.withCredentials = true; // << 주의!!
    invocation.onreadystatechange = handler;
    invocation.send();
  }
}
```

또한 서버에서는 `Access-Control-Allow-Credentials: true`로 지정해 주어야 한다. 또한, `Access-Control-Allow-Origin` 속성을 요청한 도메인으로 지정해주어야 한다. `*`와 같은 와일드 카드는 사용 할 수 없다.

#### Requests with credentials 예제

##### Request

```http
GET /resources/access-control-with-credentials/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.5; en-US; rv:1.9.1b3pre) Gecko/20081130 Minefield/3.1b3pre
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Connection: keep-alive
Referer: http://foo.example/examples/credential.html
Origin: http://foo.example
Cookie: pageAccess=2
```

##### Response

```http
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:34:52 GMT
Server: Apache/2.0.61 (Unix) PHP/4.4.7 mod_ssl/2.0.61 OpenSSL/0.9.7e mod_fastcgi/2.4.2 DAV/2 SVN/1.4.2
X-Powered-By: PHP/5.2.6
Access-Control-Allow-Origin: http://foo.example
Access-Control-Allow-Credentials: true
Cache-Control: no-cache
Pragma: no-cache
Set-Cookie: pageAccess=3; expires=Wed, 31-Dec-2008 01:34:53 GMT
Vary: Accept-Encoding, Origin
Content-Encoding: gzip
Content-Length: 106
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain
```

### Request without Credential {#request-without-credentials}

Requests with credentials이 아닌 모든 요청은 Request without Credential에 해당한다.

## CORS 관련 HTTP request Header 속성

### Origin

```http
Origin: <origin>
```

cross-site 접근 요청 혹은 사전 전달 요청의 출처

### Access-Control-Request-Method

```http
Access-Control-Request-Method: <method>
```

[Preflighted request](#preflighted-requests)의 사전 요청에서 본 요청에서 어떤 HTTP method가 사용될 것인지 서버에 알리기 위해 사용된다.

### Access-Control-Request-Headers

```http
Access-Control-Request-Headers: <field-name>[, <field-name>]*
```

[Preflighted request](#preflighted-requests)의 사전 요청에서 본 요청에서 어떤 HTTP header가 사용될 것인지 서버에 알리기 위해 사용된다.

## CORS 관련 HTTP response Header 속성

### Access-Control-Allow-Origin

```http
Access-Control-Allow-Origin: <origin> | *
```

`origin`는 리소스에 접근하는 URI을 뜻한다. 자격 증명이 없는 요청은 와일드 카드인 `*`을 사용 할 수 있다. `*` 와일드 카드를 사용 할 경우 리소스에 접근하는 모든 요청이 허용된다.

### Access-Control-Expose-Headers

```http
Access-Control-Expose-Headers: <field-name>[, <field-name>]*
```

요청이 허용되는 화이트리스트 Header를 지정한다. 예를 들어 `Access-Control-Expose-Headers: X-My-Custom-Header, X-Another-Custom-Header`일 경우 `X-My-Custom-Header`, `X-Another-Custom-Header` Header만 허용된다.

### Access-Control-Max-Age

```http
Access-Control-Max-Age: <delta-seconds>
```

캐시되는 기간을 뜻한다. `delta-seconds`값은 초 단위이며, `60`으로 저징될 경우 1분간 지속된다는 의미이다.

### Access-Control-Allow-Credentials

```http
Access-Control-Allow-Credentials: true | false
```

[Requests with credentials](#requests-with-credentials)에서 자격 요건을 충족했는지 여부로 사용되거나, [Preflighted request](#preflighted-requests)의 사전 요청에 대한 응답으로 본 요청을 수행할지 여부를 판단한다.

### Access-Control-Allow-Methods

```http
Access-Control-Allow-Methods: <method>[, <method>]*
```

허용된 method 종류를 지정한다.

### Access-Control-Allow-Headers

```http
Access-Control-Allow-Headers: <field-name>[, <field-name>]*
```

본 요청에서 어떤 Header들이 허용되는지 응답하는 용도로 사용된다.

## 참고 링크
- [MDN - Cross-Origin Resource Sharing (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS){:target="_blank"}
- [Cross Origin Resource Sharing - CORS](https://homoefficio.github.io/2015/07/21/Cross-Origin-Resource-Sharing/){:target="_blank"}
- [Examples of CORS in Action](http://arunranga.com/examples/access-control/){:target="_blank"}