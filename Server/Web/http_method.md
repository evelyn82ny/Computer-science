## HTTP method

- GET: 리소스 조회하며 서버에 전달하고 싶은 데이터는 **Query(쿼리 파라미터, 쿼리 스트링)** 를 통해 전달(메시지 바디는 지원하지 않는 곳이 많아 권장하지 않음)
- POST: 새로운 리소스를 등록하는 메서드로 메시지 바디를 통해 서버로 요청 데이터 전달
- PUT: 기존 리소스를 대체하거나 없다면 새로 생성하는 메소드로 클라이언트가 리소스의 위치를 알고 있다는 점이 POST와의 차이
- PATCH: 기존 리소스를 부분 변경
- DELETE: 리소스 삭제
<br>

- HEAD: GET과 동일하지만 메시지 부분을 제외하고 상태 줄과 헤더만 반환
- OPTIONS: 대상 리소스에 대한 ㄴ통신 가능 옵션(메서드)을 설명 -> 주로 CORS에서 사용
- CONNECT: 대상 자원으로 식별되는 서버에 대한 터널을 설정
- TRACE: 대상 리소스에 대한 경로를 따라 메시지 루프백 테스트를 수행

<br>

![png](/Server/_img/http.png)

> 출처: https://ko.wikipedia.org/wiki/HTTP

<br>

## 멱등성 Idempotent

여러번 호출해도 결과가 달라지지 않는 성질을 멱등성이라 한다.
GET, PUT, DETELE 메서드는 여러번 호출해도 결과가 같기 때문에 멱등 메소드라고 한다.
<br>

하지만 POST를 여러번 호출하면 같은 데이터가 여러개 생긴다. 이는 모두 다른 객체이므로 멱등 메소드라고 할 수 없다.