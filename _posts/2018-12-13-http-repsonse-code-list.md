---
title: "[HTTP]  응답코드 목록"
date: 2018-12-13 23:11 -0400
categories: http
tags: http rest
---

항상 헷갈리는 HTTP 응답 코드를 정리하고자 한다. 전체 코드 보다는 주로 사용 할 것같은 코드 위주로 진행하겠다 :-)  
*물론 주관적인 사용 빈도이다.

## 200번대 (성공)

200번대 코드는 클라이언트가 요청한 동작을 수신하여 이해했고 성공적으로 처리했음을 의미한다.

 코드 | 구분   | 설명                                                                               
------|--------|------------------------------------------------------------------------------------
 200  | 성공   | 클라이언트의 요청을 정상적으로 수행함                                              
 201  | 작성됨 | 요청이 정상적으로 수행되었으며, 새로운 리소스를 작성함(POST를 통한 게시물 생성 등) 

## 300번대(리다이렉션 완료)

300번대 코드는 클라이언트의 요청을 마치기 위해 추가 동작이 필요함을 의미한다.

 코드 | 구분      | 설명                                                                                                         
------|-----------|------------------------------------------------------------------------------------------------------
 301  | 영구 이동 | 요청된 페이지가 새로운 위치로 이동됨(Location header에 변경된 URI를 작성해주면, 요청이 해당 위치로 전달된다) 
 302  | 임시 이동 | 요청된 페이지가 임시로 새로운 위치로 이동됨                                                                  

## 400번대(요청 오류)

400번대 코드는 클라이언트에 오류가 있음을 나타낸다.

 코드 | 구분               | 설명                                                                                                         
------|--------------------|--------------------------------------------------------------------------------------------------------------
 400  | 잘못된 요청        | 요청이 부적절하여 구문을 인식하지 못함                                                                       
 401  | 권한 없음          | 인증이 필요한 리소스에 인증 없이 접근하고자 함(로그인 하지 않은 사용자가 관리자 페이지에 접근 시도했을 경우) 
 403  | 금지됨             | 인증 여부에 상관없이 응답하고 싶지 않은 리소스에 접근하고자 함(403보다는 400이나 404를 사용할 것을 권고)     
 404  | 찾을 수 없음       | 요청한 페이지를 찾을 수 없음                                                                                 
 405  | 허용되지 않는 방법 | 서버에서 허용되지 않는 method를 통해 요청함(POST만 허용되는 페이지에 GET으로 요청했을 경우)                  
 410  | 사라짐             | 요청한 페이지가 영구적으로 삭제됨                                                                            

## 500번대(서버 오류)

500번대 코드는 서버가 요청에 정확히 대응하지 못했음을 의미한다.

 코드 | 구분           | 설명                                                                          
------|----------------|-------------------------------------------------------------------------------
 500  | 내부 서버 오류 | 서버에 오류가 발생해 요청을 수행 할 수 없음                                   
 501  | 구현되지 않음  | 서버에 요청을 수행 할 수 있는 기능이 없음(서버가 요청 method를 인식하지 못함) 

## 참고링크
- [위키 피디아 HTTP 상태 코드][link-wiki]
- [REST API 제대로 알고 사용하기][link-rest-api]

[link-wiki]: https://ko.wikipedia.org/wiki/HTTP_%EC%83%81%ED%83%9C_%EC%BD%94%EB%93%9C
[link-rest-api]: https://meetup.toast.com/posts/92