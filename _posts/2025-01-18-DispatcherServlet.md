---
title: DispatcherServlet의 등장 과정
date: 2025-01-18 18:41:00 +09:00
categories: [spring]
tags:
  [
    spring,
    dispatcherservlet
  ]
---

## 들어가며

@Controller와 @RequestMapping 애노테이션으로 간단하게 URL에 대한 컨트롤러를 매핑시키는 스프링과 달리, 예전에는 수동으로 직접 매핑하고 관리하여 불편했었다.
<br>
변천사를 살펴보며 오늘날 `DispatcherServlet`까지 등장하게 된 배경을 알아보자.

<br>

## 초기 JAVA 웹애플리케이션

초기에는 서블릿/JSP 기술로 웹서버를 개발했었다.
<br>
당시 `web.xml`파일에 URL과 서블릿을 **1:1로 매핑**하여 수동으로 정의하고 관리해야 했다.

```xml
<web-app>
    <servlet>
        <servlet-name>HelloServlet</servlet-name>
        <servlet-class>com.example.HelloServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>HelloServlet</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>

    <servlet>
        <servlet-name>GoodbyeServlet</servlet-name>
        <servlet-class>com.example.GoodbyeServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>GoodbyeServlet</servlet-name>
        <url-pattern>/goodbye</url-pattern>
    </servlet-mapping>
</web-app>
```

위 방식은 서블릿 명이나 클래스명이 바뀐 경우에 일일이 찾아서 직접 바꿔주어야 했다.
<br>
때문에 URL이 많아질수록 가독성이 떨어지고, 인증/로깅 등의 부가작업을 처리할 때도 매핑작업이 추가되어 유지보수가 어려웠다.

<br>

## MVC 패턴 도입

위의 문제를 개선하기 위해 MVC 패턴 도입 후에는 **URL 매핑 정보를 애노테이션과 코드로 관리했다.**

<img src="/assets/img/25/01/18/MVC.png" style="border-radius:5px" alt="MVC" width="600">

```java
// 서블릿과 URL 매핑
@WebServlet(name = "mvcMemberListServlet", urlPatterns = "/servlet-mvc/members")
public class MvcMemberListServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        List<Member> members = memberRepository.findAll();
        request.setAttribute("members", members);

        // JSP 뷰로 데이터 전달
        String viewPath = "/WEB-INF/views/members.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

컨트롤러가 요청을 받아 데이터를 처리하고 뷰로 전달한다.
<br>

## 문제점

MVC 패턴의 도입으로 URL 매핑, 요청 처리가 개선되었지만 아직 문제가 남아있었다.

1. view로 이동하는 코드가 항상 중복 호출된다.
2. viewPath의 형식도 중복된다. 또한 하드코딩으로 경로를 관리하는 것은 유지보수를 어렵게 한다.
3. 인증, 로깅 등의 공통 작업을 정의하기 어렵다.

결국 공통 처리가 어렵다는 것이 가장 큰 문제다. 이를 해결하기 위해 `Front Controller 패턴`이 등장한다.

<br>

## Front Controller

<img src="/assets/img/25/01/18/front_controller.png" style="border-radius:5px" alt="front_controller" width="600">

상속을 사용하여 URL 요청에 대한 비즈니스 로직만 정의하고, 나머지 로직은 모두 프론트 컨트롤러에서 처리된다.
<br>
모든 컨트롤러들을 Map으로 관리하며, 공통 작업도 쉽게 정의할 수 있다.

<details>
  <summary>Front Controller</summary>
  <div markdown="1">

  ```java
  @WebServlet(name = "frontController", urlPatterns = "/front-controller/*")
  public class FrontControllerServlet extends HttpServlet {

      private final Map<String, Controller> handlerMappingMap = new HashMap<>();

      public FrontControllerServlet() {
          handlerMappingMap.put("/front-controller/members", new MemberListController());
          handlerMappingMap.put("/front-controller/members/new", new MemberFormController());
      }

      @Override
      protected void service(HttpServletRequest request, HttpServletResponse response)
              throws ServletException, IOException {

          String requestURI = request.getRequestURI();
          Controller controller = handlerMappingMap.get(requestURI);

          if (controller == null) {
              response.setStatus(HttpServletResponse.SC_NOT_FOUND);
              response.getWriter().write("404 Not Found");
              return;
          }

          if (controller instanceof LoggableController) {
              logRequest(request);
          }

          String viewName = controller.process(request, response);
          MyView view = viewResolver(viewName);
          if (view != null) {
              view.render(request, response);
          }
      }

      private void logRequest(HttpServletRequest request) {
          System.out.println("Logging Request - URL: " + request.getRequestURI());
          System.out.println("Logging Request - HTTP Method: " + request.getMethod());
      }

      private MyView viewResolver(String viewName) {
          return new MyView("/WEB-INF/views/" + viewName + ".jsp");
      }
  }
  ```
  </div>
</details>

<details>
  <summary>View</summary>
  <div markdown="1">

  ```java
  public class MyView {

      private final String viewPath;

      public MyView(String viewPath) {
          this.viewPath = viewPath;
      }

      public void render(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
          RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
          dispatcher.forward(request, response);
      }
  }
  ```
  </div>
</details>

<details>
  <summary>Controller(인터페이스)</summary>
  <div markdown="1">
  
  ```java
  public interface Controller {
      String process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
  }
  // LoggableController 생략
  ```
  </div>
</details>

<details>
  <summary>Controller(구현체)</summary>
  <div markdown="1">

  ```java
  public class MemberListController implements Controller {

      private final MemberRepository memberRepository = MemberRepository.getInstance();

      @Override
      public String process(HttpServletRequest request, HttpServletResponse response) throws IOException {
          List<Member> members = memberRepository.findAll();
          request.setAttribute("members", members);

          return "members";
      }
  }
  ```
  </div>
</details>

<br>

구현체 컨트롤러를 보면 알겠지만, 레포지토리로부터 데이터를 가져와 응답데이터에 저장한 후 view 경로를 반환하는 3줄의 로직으로 간단하게 변경되었다.
<br>
또한 View 경로도 논리적 뷰 이름을 물리적 뷰 이름으로 변환하는 방식으로 중복을 제거하였다.
<br>
이로써 위에서 언급한 모든 문제를 해결하였다.
<br>

<img src="/assets/img/25/01/18/introduce.png" style="border-radius:5px" alt="introduce" width="600">

<br>

## DispatcherServlet

위의 FrontController에서 다음 기능들을 추가하면 오늘날 스프링 프레임워크의 `DispatcherServlet`이 된다.
- 서블릿 종속성 제거
- DTO와 유사한 기능을 수행하는 `ModelAndView`
- 어댑터를 통해 개선한 컨트롤러 구조의 유연성 등

<img src="/assets/img/25/01/18/DispatcherServlet.png" style="border-radius:5px" alt="DispatcherServlet" width="600">

> DispatcherServlet은 모든 요청을 중앙에서 처리하고 적절한 핸들러로 요청을 분배하는 역할을 한다.
{: .prompt-info }

> 마치 로드밸런서와 유사하다고 볼 수 있다.

이렇게 추상화된 프론트 컨트롤러 덕분에 아래와 같은 코드만으로 URL에 대한 컨트롤러를 매핑할 수 있게 된다.

```java
@Controller
@RequestMapping("/members")
public class MemberController {

    private final MemberRepository memberRepository = MemberRepository.getInstance();

    @GetMapping
    public String list(Model model) {
        List<Member> members = memberRepository.findAll();
        model.addAttribute("members", members);
        return "members";
    }
}
```