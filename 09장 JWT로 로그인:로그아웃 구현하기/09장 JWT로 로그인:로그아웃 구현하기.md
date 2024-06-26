# 09장 JWT로 로그인/로그아웃 구현하기

## 9.1 사전 지식: 토큰 기반 인증

- 9.1.1 토큰 기반 인증이란?
    - 대표적인 사용자 인증 확인 방법
    - 토큰(서버에서 클라이언트를 구분하기 위한 유일한 값)을 사용
    - 서버가 토큰을 생성하여 클라이언트에게 제공 → 클라이언트는 토큰을 가지고 있다가 요청을 토큰과 함께 신청 → 서버는 토큰만 보고 유효한 사용자인지 검증
    - 토큰 전달하고 인증 받는 과정
        1. 클라이언트 → 서버 : 로그인 요청
            1. 클라이언트가 아이디와 비밀번호를 서버에게 전달하며 인증 요청
        2. 서버 → 클라이언트 : 토큰 생성 후 응답
            1. 서버에서 아이디, 비밀번호를 확인하고 유효한 사용자인지 검증
            2. 유효한 사용자면 토큰을 생성하여 응답
        3. 클라이언트 : 토큰 저장
        4. 클라이언트 → 서버 : 토큰 정보와 함께 요청
            1. 이후 인증이 필요한 API를 사용할 때 토큰을 함께 보냄
        5. 서버 : 토큰 검증
        6. 서버 → 클라이언트 : 응답
        - 토큰 기반 인증의 특징
            - 무상태성
                - 토큰 기반 인증에서 클라이언트에서 인증 정보가 담긴 토큰을 생성하고 인증함
                - 사용자의 인증 정보가 담겨 있는 토큰이 클라이언트에 있으므로 서버에 저장할 필요가 없음
                    - 서버가 데이터를 유지하지 않아 그만큼 자원을 소비하지 않아도 됨
                - 클라이언트에서 사용자의 인증 상태를 유지하며 요청 처리  = 상태 관리
                - 서버 입장에서 클라이언트의 인증 정보를 저장하거나 유지하지 않아도 됨 → 완전한 무상태로 효율적인 검증 가능
            - 확장성
                - 서버를 확장할 때 상태 관리를 신경 쓸 필요가 없어 서버 확장에 용이함
                - 예) 물건 파는 서비스에서 결제를 위한 서버와 주문을 위한 서버가 분리되어 있음 → 하나의 토큰으로 결제, 주문 서버에게 요청 보낼 수 있음
                - 페이스북 로그인, 구글 로그인 같이 토큰 기반 인증을 사용하는 다른 시스템에 접근해 로그인 방식 확장 가능
            - 무결성
                - 토큰 발급 이후 토큰 변경 행위는 불가능
    - 세션 기반 인증
        - 스프링 시큐리티에서 기본적으로 제공하는 인증 방법
        - 사용자마다 사용자의 정보를 담은 세션을 생성하고 저장하여 인증
- 9.1.2 JWT
    - 인증하려면 HTTP 요청 헤더 중에 Authorization 키값에 **Bearer + JWT 토큰값**을 넣어 보내야 함
    - 구조
        - .을 기준으로 헤더, 내용, 서명으로 이루어져 있음
        - 헤더 : 토큰 타입과 해싱 알고리즘을 지정하는 정보 담음
            - typ : 토큰 타입 지정, JWT라는 문자열이 들어가게 됨
            - alg : 해싱 알고리즘 지정
        - 내용 : 토큰과 관련된 정보를 담음
            - 클레임 : 내용의 한 덩어리, 키값의 한 쌍으로 이루어짐
                - 등록된 클레임 : 토큰에 대한 정보를 담는 데 사용
                    - iss : 토큰 발급자
                    - sub : 토큰 제목
                    - aud : 토큰 대상자
                    - exp : 토큰의 만료시간
                    - nbf : 토큰의 활성날짜와 비슷한 개념, Not Before로 이 날짜가 지나기 전까지는 토큰이 처리되지 않음
                    - iat : 토큰이 발급된 시간
                    - jti : JWT의 고유 식별자로서 주로 일회성 토큰에 사용
                - 공개 클레임
                    - 공개되어도 상관없는 클레임
                    - 충돌을 방지할 수 있는 이름을 가져야 함
                    - 보통 클레임 이름을 URI로 지음
                - 비공개 클레임
                    - 공개되면 안 되는 클레임
                    - 클라이언트와 서버 간 통신에 사용
                    - 등록된 클레임도, 공개 클레임도 아닌 것은 비공개 클레임
        - 서명
            - 해당 토큰이 조작되었거나 변경되지 않았음을 확인하는 용도로 사용
            - 헤더의 인코딩값과 내용의 인코딩값을 합친 후에 주어진 비밀키를 사용해 해시값 생성
    - 토큰 유효기간
        - 리프레시 토큰
            - 액세스 토큰과 별개의 토큰
            - 액세스 토큰이 만료되었을 때 새로운 액세스 토큰을 발급하기 위해 사용함
            - 액세스 토큰의 유효 기간은 짧게, 리프레시 토큰의 유효 기간은 길게 설정하면?
                - 공격자가 액세스 토큰을 탈취해도 몇 분 뒤에는 사용할 수 없게 되어 더 안전함
        - 클라이언트 → 서버 : 인증 요청
        - 서버 → 클라이언트 : 액세스 토큰 & 리프레시 토큰 응답
        - 클라이언트, 데이터베이스 : 리프레시 토큰 저장
        - 클라이언트 → 서버 : 요청
        - 서버 → 클라이언트 : 토큰 유효성 검사 & 응답
        - 클라이언트 → 서버 : (만료된 액세스 토큰과 함께) 요청
        - 서버 → 클라이언트 : 토큰 만료 응답
        - 클라이언트 → 서버 : (리프레시 토큰과 함께) 액세스 토큰 발급 요청
        - 서버 → DB : 리프레시 토큰 조회 & 유효성 검사
        - 서버 → 클라이언트 : 새로운 액세스 토큰 응

## 9.2 JWT 서비스 구현하기

- 9.2.1 의존성 추가하기
- 9.2.2 토큰 제공자 추가하기
    - jwt를 사용하여 JWT 생성, 유효한 토큰인지 검증하는 역할을 하는 클래스 추가
    - JWT 토큰을 만들려면 이슈 발급자, 비밀키를 필수로 설정해야 함
    - 해당 값들을 변수로 접근하는 데 사용할 JwtProperties 클래스를 만듦
    - 토큰을 생성하고 올바른 토큰인지 유효성 검사를 하고, 토큰에서 필요한 정보를 가져오는 TokenProvider 클래스 작성
    - 테스트 코드 작성
        - generateToken() 메서드 : 토큰 생성하는 메서드를 테스트하는 메서드
            
            
            | given | 토큰에 유저 정보를 추가하기 위한 테스트 유저를 만듦 |
            | --- | --- |
            | when | 토큰 제공자의 generateToken() 메서드를 호출해 토큰을 만듦 |
            | then | jjwt 라이브러리를 사용해 토큰을 복호화함. 토큰을 만들 때 클레임으로 넣어둔 id값이 given절에서 만든 유저 ID와 동일한지 확인함 |
        - validToken_invalidToken() 메서드 : 토큰이 유효한 토큰인지 하는 검증하는 메서드인 validToken() 메서드를 테스트하는 메서드
            - validToken_invalidToken() 메서드 : 검증 실패 확인
                
                
                | given | jjwt 라이브러리를 사용해 토큰 생성함. 이때 만료 시간은 1970년 1월 1일부터 현재 시간을 밀리초 단위로 치환한 값에 1000을 빼 이미 만료된 토큰으로 생성함 |
                | --- | --- |
                | when | 토큰 제공자의 validToken() 메서드를 호출해 유효한 토큰인지 검증한 뒤 결과값 반환받음 |
                | then | 반환값이 false인 것을 확인함 |
            - validToken_validToken() 메서드 : 검증 성공 확인
                
                
                | given | jjwt 라이브러리를 사용해 토큰 생성함. 만료 시간은 현재 시간으로부터 14일 뒤로 만료되지 않은 토큰으로 생성함 |
                | --- | --- |
                | when | 토큰 제공자의 validToken() 메서드를 호출해 유효한 토큰인지 검증한 뒤 결과값 반환받음 |
                | then | 반환값이 true인 것을 확인함 |
        - getAuthentication() 메서드 : 토큰을 전달받아 인증 정보를 담은 객체 Authentication를 반환하는 메서드인 getAuthentication()를 테스트
            
            
            | given | jjwt 라이브러리를 사용해 토큰 생성함. 이때 토큰 제목인 subject는 “user@email.com” 값 사용 |
            | --- | --- |
            | when | 토큰 제공자의 getAuthentication() 메서드를 호출해 인증 객체를 반환받음 |
            | then | 반환받은 인증 객체의 유저 이름을 가져와 given절에서 설정한 subject값인 “user@email.com”과 같은지 확인함 |
        - getUserId() 메서드 : 토큰 기반으로 유저 ID를 가져오는 메서드를 테스트하는 메서드
            
            
            | given | jjwt 라이브러리를 사용해 토큰 생성함. 이때 클레임을 추가함. 키는 “id”, 값은 1인 유저 ID |
            | --- | --- |
            | when | 토큰 제공자의 getUserId() 메서드를 호출해 유저 ID를 반환받음 |
            | then | 반환받은 유저 ID가 given절에서 설정한 유저 ID값인 1과 같은지 확인함 |
- 9.2.3 리프레시 토큰 도메인 구현하기
    - 리프레시 토큰 : DB에 저장하는 정보이므로 엔티티와 리포지터리를 추가해야 함
    - 만들 엔티티와 매핑되는 테이블 구조
        
        
        | 칼럼명 | 자료형 | null 허용 | 키 | 설명 |
        | --- | --- | --- | --- | --- |
        | id | BIGINT | N | 기본키 | 일련번호 기본키 |
        | user_id | BIGINT | N |  | 유저 ID |
        | refresh_token | VARCHAR(255) | N |  | 토큰값 |
        - domain 디렉터리에 [RefreshToken.java](http://RefreshToken.java) 파일 추가
        - repository 디렉터리에 [RefreshTokenRepository.java](http://RefreshTokenRepository.java) 파일 추가
- 9.2.4 토큰 필터 구현하기
    - 필터 : 각종 요청이 요청을 처리하기 위한 로직으로 전달되기 전후에 URL 패턴에 맞는 모든 요청을 처리하는 기능 제공
        - 요청이 오면 헤더값 비교해 토큰 있는지 확인 → 유효 토큰이라면 시큐리티 콘텍스트 홀더에 인증 정보 저장
            - 시큐리티 컨텍스트 : 인증 객체가 저장되는 보관소, 인증 정보가 필요할 때 언제든지 인증 객체를 꺼내 사용할 수 있음
                - 이 클래스는 스레드 로컬에 저장되므로 코드 아무 곳에서나 참조할 수 있음, 독립적 사용 가능
            - 시큐리티 컨텍스트 홀더 : 시큐리티 컨텍스트 객체를 저장하는 객체
    - TokenAuthenticationFilter.java
        - 필터는 액세스 토큰값이 담긴 Autorization 헤더값을 가져온 뒤 액세스 토큰이 유효하다면 인증 정보를 설정함

## 9.3 토큰 API 구현하기

- 리프레시 토큰을 전달받아 검증하고, 유효한 리프레시 토큰이면 새로운 액세스 토큰을 생성하는 토큰 API 구현
- 9.3.1 토큰 서비스 추가하기
    - 리프레시 토큰을 전달받아 토큰 제공자를 사용해 새로운 액세스 토큰을 만드는 토큰 서비스 클래스 생성
    - 전달받은 유저 ID로 유저를 검색해서 전달하는 findById() 메서드 추가 구현
    - service 디렉터리에 [RefreshTokenService.java](http://RefreshTokenService.java) 파일 생성
        - 전달받은 리프레시 토큰으로 리프레시 토큰 객체를 검색해서 전달하는 findByRefreshToken() 메서드 구현
    - service 디렉터리에 [TokenService.java](http://TokenService.java) 파일 생성
        - createNewAccessToken() : 전달받은 리프레시 토큰으로 토큰 유효성 검사 진행, 유효한 토큰일 때 리프레시 토큰으로 사용자 ID 찾음 → generateToken() 메서드 호출 후 새로운 액세스 토큰 생성
- 9.3.2 컨트롤러 추가하기
    - 실제로 토큰을 발급받는 API 생성
    - 토큰 생성 요청 및 응답을 담당할 DTO인 CreateAccessTokenRequest와 CreateAccessTokenResponse 클래스 생성
    - 실제로 요청을 받고 처리할 컨트롤러 생성
        - /api/token POST 요청이 오면 토큰 서비스에서 리프레시 토큰을 기반으로 새로운 액세스 토큰 생성
    - 테스트 코드 작성