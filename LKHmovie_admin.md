# LKHmovie_admin 프로젝트

1. 프로젝트 생성
- help > Eclipse MarketPlace > sts 검색 > Spring Tools 3 머시기 release 설치
- new > spring legacy project > project name 입력 > templates : spring MVC project > next > top-level package : "com.SampleBoard.khlim" > finish > 프로젝트 생성
<br>
2. 프로젝트 설정 변경
- project 우클릭 > properties 
    - project facets > java : 1.8
    - runtimes : apache tomcat v8.0
    - java build path > libraries > 톰캣 및 자바 버전 확인
- pom.xml에서 `<org.springframework-version>`태그에 스프링 최신버전 입력 (이 당시껀 4.3.25)
- WEB-INF 폴더 하위의 spring 폴더 삭제
    - root-context.xml과 servlet-context.xml 들어있음
