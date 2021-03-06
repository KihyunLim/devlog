# project invitation - admin

---
## 개발 환경


---
## 개발 범위 (관리자)
1. 회원 관리
   - 회원 CRUD
2. 청첩장 관리
   - 청첩장 기간, 내용 관리, 청첩장 사이트 이동
   - 방명록, 약도 등 내용을 선택적으로 사용 할 수 있게 제작
   - hookup/[id] 형식으로 url 만들어서 디자인 구별할 수 있게
3. 추가개발
   - 결제 메뉴
   - 다양한 청첩장 양식 선택 가능
     - 사용될 메뉴 항목들을 DB화 잘 시켜야 할 듯

## 개발 범위 (사용자)
1. 청첩장 사이트
   - 일단 hookup 쓰고 (관리자용 그대로 쓸테니 추가개발까진 안될 듯)
2. 제작 주문 참고 사이트
   - https://inviteyou.co.kr/index.html
   - 회원 가입부터 해서 할 게 많겠다.
   - 결제기능도 추가하고 싶음
3. 추가개발
   - 제작 주문 사이트에서 다양한 양식의 청첩장 고를 수 있게 추가개발

---
## 개발 상세 (관리자)
1. 회원 관리
   - 검색 : 아이디, 이름, 핸드폰
   - 정렬 : 아이디, 이름, 핸드폰
   - 회원 추가/삭제 버튼
   - 아이디, 이름, 핸드폰, 청첩장 o/x
   - 더블 클릭 시 회원 상세
     - 아이디, 비밀번호, 이름, 핸드폰, 가입일자, 청첩장 게시 기간(가장 최근으로)
   - 회원 상세에서 회원 정보 수정 가능
     - 비밀번호, 이름, 핸드폰

2. 청첩장 관리
   - 소메뉴
     - 청첩장 목록 조회, 청첩장 상세 조회(/수정), 청첩장 추가
   - 청첩장 목록 조회
     - 검색 : 아이디, 이름, 게시 기간
     - 아이디, 이름, 청첩장 게시 상태, 청첩장 게시 시작/종료 기간, 청첩장 사이트 이동 버튼, 삭제 버튼
     - 더블 클릭 시 상세화면으로 이동
   - 청첩장 상세 조회(/수정), 추가
     - 기본 정보 : 순번, 아이디, 이름, 게시 상태, 게시 기간
       - 게시기간은 달력 팝업 라이브러리 사용(datepicker 있네)
     - (필)Home, Groom & Bride : 결혼 일자/일시/장소, 메인사진, 신랑/신부 각자 사진, 신랑/신부/부/모 이름 (이름 밑에 영역에 간단 소개를 할까? 누구의 차남/녀를 쓸까?)
     - (선)Love Story : 목록_(jquery 이벤트 드래그앤드롭으로 li 통째로 옮길 수 있게 ㄱ) 이미지, 일자, 제목, 내용
     - (X)People : 내가 안쓸거임
     - (필)When Where :  목록_ 행사(폐백,결혼식 체크 할 수 있게), 일자, 일시, 장소
       - 지도 보기 버튼, 미니맵도 넣을 수 있으면 좋을 듯 => 이건 청첩장 시이트에서
       - 추가할 아이콘이 없으니 폐백 결혼식만 체크박스로 선택하게 
       - 두개면 col-md-6 하나면 col-md-12
     - (선)Gallery : 이미지 업로드, 추가 삭제 버튼
       - 사용자 페이지는 adminlte3 examples > pages > e-commerce 쓰면 좋을듯
       - 관리자 페이지는 gallery 샘플 사용하고 div태그에 버튼같은거 추가
       - drag&drop 이벤트로 순서 설정 가능하면 좋고
     - (선)Sweet Message : 목록_ 순번, 이름, 내용, 일시, 비밀번호, 다운로드, 삭제(관리자용)
     - (선)RSVP : 참여 한다 안한다 기능인거 같은데 난 방명록 기능으로 수정해서 사용 ㄱ  (위에 작성된 방명록이랑 영역 같이사용)
       - 관리자에는 필요 없고 청첩장 사이트에 있으면 될
