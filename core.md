### HTTP 기본
- HTTP : 웹상 모든 데이터 전달은 HTTP.
- **Stateful vs Stateless**
  - Stateful : 상태유지. 클라이언트쪽에서 보내온 정보를 계속 가지고 있어야 돼. 때문에 클라이언트와 연결된 서버를 바꾸기가 어려워
  - Stateless : 무상태. 클라이언트쪽에서 보내온 정보를 유지하고 있지 않음. 따라서 클라이언트쪽에서 요청할떄마다 데이터를 길게 보내줘야돼. 그러나 수평확장. 스케일아웃하기에 참 좋아. 대응이 유연.
  - 일반적으로 무상태로 설계를 대부분 하되, 로그인 정보를 유지해야 되는 상황 같은 경우에는 브라우저쿠키와 서버세션을 이용해서 상태유지로 설계함. 상태유지는 최소한으로.
- HTTP는 기본이 비연결성. 
  - 서버랑 계속 연결된 상태라면 서버의 자원 소비가 크겠지. 그렇고 뭐 할때마다 매번 3way 해야되면 성능이 떨어질테고. 

### HTTP 메서드
- URI 설계
  - URI는 리소스만 식별. 리소스와 행위를 분리해야돼
- **메소드**
  - GET: 조회
  - POST: 새로운 리소스 생성, 변경 뿐만 아니라, 리소스를 전달 받아서 프로세스 처리 해야되는 경우, get 이외의 경우에 대부분 post로 넘김.
  - PUT : 덮어씌움. 없다면 생성. 때문에 POST와 다르게 URI를 파일 위치까지 명시해줌. members/100
  - PATCH : 부분 변경. PUT은 전달하지 않은 필드들을 null로 덮어버리지만, PATCH는 다른 필드값을 갖는 애만 골라서 변경해줌. 다만 서버에 따라 지원 안하는 경우도 있는데, 그럴때는 그냥 POST 이용
  - DELETE : 삭제

### HTTP 메서드 활용
- 신규 회원 등록 같은건 POST, 파일 관리 시스템은 PUT
  - 파일 관리 시스템은 새로운 파일을 등록한다 치면, 클라이언트쪽에 리소스의 uri를 정확히 지정해줘야겠지. ex) "members/star.jpg" 때문에 post가 아니라 put.
- 순수 http form에는 get과 post만 사용할 수 있어. 때문에 edit, delete도 다 post로 진행시켜야 하고, 때문에 리소스가 아닌 행위로 명명한 **컨트롤 uri**라는 개념이 들어옴
  - 컨트롤 uri의 예 : /members/{id}/delete

### HTTP 상태코드
- 2xx : 성공. 200 요청성공, 201 created 성공
- 3xx : 리다이렉트: 클라이언트가 요청한 uri가 아니라 바뀐 uri를 응답으로 보내서 클라이언트가 자동으로 바뀐 uri 쪽으로 요청을 보낼 수 있게 해줌
  - 302 : 보통 유지 안하고 get으로 변경시켜버림. 얘를 제일 많이 사용함.
  - 304 : 캐시를 리다이렉트.
  - PRG : Post Redirect Get. 클라이언트가 post 요청했는데, get으로 리다이렉트 시키는거. 주문 요청할때 post 이후 주문 완료 페이지를 get 하게 리다이렉트 시켜야 새로고침시 중복 주문 안됨.
- 4xx : 클라이언트 오류
  - 401 : 로그인 안됨,
  - 403 : 로그인 됐지만, 그 권한으로는 요청 리소스에 접근할 수 없어
  - 404 : 요청 uri에 리소스가 없어.
- 5xx : 서버 오류

### HTTP 헤더1 - 일반 헤더
- 헤더 : 메시지 바디에 대한 메타데이터를 가지고 있고, 클라이언트쪽에 대한 데이터도 담고 있음
- Content-Type, Content-encoding, Content-Language, Content-Length
- 콘텐츠협상 : 선호하는 데이터 형식의 우선순위 값을 지정해 요청 할 수 있음. ex) Accept-xxx
- 특별한 정보 : Host(접속할 도메인 명시. 요청시 필수 정보), Location(리다이렉트할 주소)
- 인증 : Authorization(클라이언트 인증 정보를 서버에 전달), WWW-Authenticate(리소스 접근시 필요한 인증 방법 정의)
- 쿠키 : 로그인 했다라는 정보를 매번 헤더에 담을 수 없으니 사용함
  - Set-Cookie : 서버에서 클라이언트로 쿠키 전달(응답)
  - Cookie: 클라이언트가 서버에서 받은 쿠키를 저장하고, http 요청시 서버로 전달
  - 만료 시간, 도메인, 경로등을 설정하여 제한하기도함.
  - 그밖에 set-cookie에 유저 id password를 그대로 주면 위험하기도 하고 용량도 크니, session key를 서버에 저장하고 session id를 담거나, Oauth 같은 인증토큰을 담아서 전달. 
  - 요청할때마다 쿠키를 전달하는것도 좀 번거로우니까 웹브라우저 자체내에 저장하는 방식도 있음.

### HTTP 헤더2 - 캐시와 조건부 요청
- **캐시 기본 동작**
  - A: star.jpg 주세요. 
  - B: 오케이. 헤더랑 바디 다 줌
  - A: star.jpg 주세요. 캐시 만료됨요. (아직 cache-control: max-age에 담긴 유효시간값이 다 지나지 않았다면, 그냥 캐시에서 꺼내씀.)
  - B: 검증 헤더 확인하고, 동일하면 너 캐시에 있는거 그대로 써도 됨. 헤더만 보냄. 동일하지 않으면, 여기 새로운거 받아. 헤더랑 바디 다 줌
- 검증 헤더
  - Last-Modified : 최종 수정 시간
  - Etag : 시간으로 검증하는게 아니라 보낼 데이터의 해쉬값이나 임의로 지정한 태그id를 비교.
- 조건부 요청
  - if-modified-since : 만약 최종 수정 시간이 동일하다면 안보내줘도 돼요 라는 요청
  - if-none-match : 만약 Etag 값이 동일하면 안보내줘도 돼요 라는 요청.
  - 304 not modified : 캐시 내용이 수정되지 않았습니다. 그러니까 가지고 있는 캐시 쓰세요.
  - 200 : 캐시 내용이 수정되었습니다. 하면서 새로운 데이터 보내줌.
- 프록시 캐시: 원 서버는 미국에 있는데, 매번 미국 서버한테 유튜브 영상 요청하면 응답 받는데 너무 느리겠지. 그래서 한국 어딘가에 프록시 캐시 서버를 만들어서, 한번 요청했던 컨텐츠들은 캐시로 해당 서버에다가 저장해놓는거.
- 캐시 무효화: 완전 캐시 무효화 하려면 no cache, no store, must-revaildate 다 써줘야됨.
