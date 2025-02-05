[JSP 웹 프로그래밍](https://www.slog.gg/p/13824)

실행 alt+shift+x

### 수업 개요

JSP = Java Servlet(Servers) Page

- 백엔드
- 프론트엔드: HTML(문서내용), CSS(스타일), JS(인터랙션)

일반적으로 서버 API -> 화면 

- API = 특정 요청에 대한 결과물
  - /usr/article/list 하면 리스트의 결과물이 들어오는데 그 결과 내용이 API라고 이해하면 됨

JSP의 경우
- HTML 안에다 자바 코드를 작성한다

JSP 문법과 JSP에서 백엔드와 프론트엔드 통신을 알아야 한다.

### 톰캣 세팅

[톰캣 세팅 강의](https://www.youtube.com/watch?v=QjcCDm2cQt0)\
[기타 세팅 강의](https://www.youtube.com/watch?v=mD43RFfOuLg&feature=youtu.be)\
[톰캣 등록 강의](https://www.youtube.com/watch?v=sbJe0Ve0W5Y)

톰캣은 src/webapp 을 경로 삼아 파일을 읽기 때문에 webapp 디렉토리 생성해야 함

Settings - Compiler - build proejct automatically : 체크
Settings - advanced Settings - Allow auto-make to start even if developed application is currently running : 체크
C:\work\jsp_projects\JSP_board\conf\context.xml 파일수정

reloadable="true" 추가

### 톰캣

톰캣 - 완성된 프로그래밍

웹프로그래밍을 하기 위해 통신, 쓰레드 등등 여러 기능이 필요한데
이런 기능을 이미 톰캣이 구현을 해둠

즉, 추가적인 로직만 만들면 된다!

서블릿 - 하나의 페이지(역할)

톰캣
- 모듈() --> jsp_board_250205
  - 서블릿 --> /hello
  - 서블릿
  - 서블릿
  - 서블릿
- 모듈
- 모듈
- 모듈

http://localhost:8080/jsp_board/hello 라는 요청을 날리면 @WebServlet("/hello")을 실행해준다
@WebServlet("/hello") 이 실행되는 것은 추가적인 로직이 톰캣 안에 스며드는 것!

롬복과 자카르타 서블릿 추가(pom.xml)
```
<dependencies>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.32</version>
      <scope>provided</scope>
    </dependency>

    <dependency>
      <groupId>jakarta.servlet</groupId>
      <artifactId>jakarta.servlet-api</artifactId>
      <version>6.1.0</version>
      <scope>provided</scope>
    </dependency>
  </dependencies>
```

### HomeServlet 클래스
alt+insert -> 오버라이드 메서드 추가

```java
@WebServlet("/home")
public class HomeServlet extends HttpServlet {
  @Override
  protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    // 한글 인코딩
    req.setCharacterEncoding("UTF-8"); // 들어오는 데이터를 UTF-8로 해석
    resp.setCharacterEncoding("UTF-8"); // 완성된 HTML 결과물의 인코딩을 UTF-8로 하겠다
    resp.setContentType("text/html; charset=utf-8");

    resp.getWriter().append("Hello!");
  }
}

```
http://localhost:8080/home 입력 시 Hello!가 나오는 것을 확인할 수 있다


`resp.getWriter().append("내용");` 의 의미
브라우저에서 요청(req) -> HTML 안에는 head와 body로 구성되어 있음 -> `resp.getWriter().append("내용");`결과물을 html body에 적어줌

프로젝트 로직에서 `resp.getWriter().append("내용");` 하게 되면 -> 톰캣 내부에 `resp.getWriter().append("내용");`결과물이 계속 쌓임
-> 쌓인 결과물이  html body에 들어감 -> 웹 브라우저에 반영


### Rq 도입

`퍼사드 패턴`
요청과 응답의 통합적인 리모콘으로 요청과 응답을 동시에 조정하고 사용 = Rq




