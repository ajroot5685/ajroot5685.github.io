---
title: JSP와 서블릿이 무엇일까?
date: 2024-10-06 22:13:00 +09:00
categories: [backend]
tags:
  [
    backend,
    jsp,
    servlet
  ]
---

## 웹 서버 프로그램
> `서블릿(Servlet)`과 `JSP`를 다루기 전에 먼저 웹 서버 프로그램이 생겨난 배경부터 짚어보겠다.

- 인터넷을 통해 정보를 공유하고자 하는 요구가 증가하면서 웹페이지를 각 클라이언트가 들고 있고 수시로 업데이트하는 것은 비효율적이었다.
- 때문에 중앙에서 웹페이지와 콘텐츠를 관리하고, 클라이언트가 필요할 때 요청하여 최신 정보를 받아가도록 웹 서버가 탄생하게 되었다.
- 이에 더 나아가 웹페이지에 회원 정보와 같은 값을 넣기 위해 DB와의 통신 작업이 필요해졌고, 웹서버로도 가능은 하지만 많은 부하가 발생하기 때문에 웹애플리케이션 서버(`WAS`)를 따로 두어 처리하게 되었다.

<br>

## 개요
- 이제 동적인 웹페이지 처리를 위한 기술들이 어떻게 발전해나가는지 보겠다.
- DB와의 통신 작업은 제외하고 간단하게 날짜라는 데이터를 넣어서 동적인 웹페이지를 만드는 예시를 보겠다.

<br>

## 순수 자바
- 당연히 초기에는 순수 자바로 WAS를 구축하고 응답을 반환했다.
- 순수 자바에서는 직접 HTTP 요청을 파싱하고 응답을 작성해야 했는데, HTTP 프로토콜에는 헤더, 상태 코드, 요청 메서드 등이 다양했기 때문에 수동으로 처리하는 것은 **비효율적이고 휴먼 에러가 발생**할 확률이 높았다.

<br>

#### [순수 자바 예시 코드]
- 요청을 받을 소켓을 열고, Reader를 생성시키고, 요청을 읽고, 소켓을 닫는 작업까지 다 수동으로 작성해야 했기에 번거롭다.

```java
public class WebServer {
    public static void main(String[] args) throws IOException {
        // 8080 포트에서 서버 소켓 열기
        ServerSocket serverSocket = new ServerSocket(8080);
        System.out.println("Server started on port 8080...");

        while (true) {
            // 클라이언트의 연결 요청 대기
            Socket clientSocket = serverSocket.accept();
            handleRequest(clientSocket);
        }
    }

    private static void handleRequest(Socket clientSocket) throws IOException {
        // 클라이언트로부터의 입력을 받기 위한 BufferedReader 생성
        BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
        
        // 클라이언트로의 출력을 위한 PrintWriter 생성
        PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true);
        
        // 요청 읽기
        String inputLine;
        while ((inputLine = in.readLine()) != null) {
            if (inputLine.isEmpty()) {
                break;
            }
        }

        // 동적인 현재 시간을 포함한 HTML 생성
        String currentTime = new Date().toString();

        // HTTP 응답 헤더 및 동적인 HTML 페이지 전송
        out.println("HTTP/1.1 200 OK");
        out.println("Content-Type: text/html");
        out.println();
        out.println("<html>");
        out.println("<head><title>동적인 웹사이트</title></head>");
        out.println("<body>");
        out.println("<h1>현재 시간이 반환되는 동적 웹사이트입니다.</h1>");
        out.println("<p>현재 시간 : " + currentTime + "</p>");
        out.println("</body>");
        out.println("</html>");
        
        // 스트림 및 소켓 닫기
        out.close();
        in.close();
        clientSocket.close();
    }
}
```

<br>

## 서블릿(Servlet)
- 서블릿은 자바의 서버 측 기술로, 클라이언트의 HTTP 요청을 처리하고, 그에 대한 응답을 생성하는 기능을 제공하는 표준 API이다.
- 즉, 자바로 WAS를 구축하기 위한 HTTP 관련 로직들이 추상화되어 개발자가 일일이 처리해주는 수고가 덜어졌다는 것이다.
- 하지만 서블릿에도 단점이 있었는데 HTML과 자바 코드가 모두 자바 클래스 안에서 작성하므로 **가독성이 떨어졌다.**

<br>

#### [서블릿 예시 코드]
- HTTP 통신 관련 로직들이 거의 생략되어 좀 더 효율적인 코드 작성이 가능해졌다.
- 예시에는 HTML이 간단하기에 괜찮아 보이지만, 복잡한 HTML을 응답할수록 자바 클래스는 지저분해진다.

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
        // 응답의 콘텐츠 타입을 설정 (HTML)
        response.setContentType("text/html");

        // 클라이언트에 응답을 보내기 위해 PrintWriter 객체 사용
        PrintWriter out = response.getWriter();

        String currentTime = new Date().toString();

        out.println("<html>");
        out.println("<head><title>동적인 웹사이트</title></head>");
        out.println("<body>");
        out.println("<h1>현재 시간이 반환되는 동적 웹사이트입니다.</h1>");
        out.println("<p>현재 시간 : " + currentTime + "</p>");
        out.println("</body>");
        out.println("</html>");
    }
}
```

<br>

## JSP(JavaServer Pages)
- JSP는 가독성이 떨어지는 문제를 HTML을 분리시키고, 동적 데이터가 필요한 경우 HTML 안에 자바 코드를 삽입하여 해결하였다.
- JSP는 내부적으로 서블릿으로 변환되어 실행되기 때문에 서블릿과 동일한 기능을 수행한다고 볼 수 있다.

<br>

#### [JSP 예시 코드]
- 이제 더이상 자바 클래스 안에 HTML을 안봐도 되므로 가독성이 좋아졌다.

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {

        // 현재 시간을 자바 객체로 저장
        String currentTime = new Date().toString();
        
        // JSP에서 사용할 수 있도록 request 객체에 데이터 저장
        request.setAttribute("currentTime", currentTime);

        // JSP 파일로 요청을 전달
        RequestDispatcher dispatcher = request.getRequestDispatcher("/hello.jsp");
        dispatcher.forward(request, response);  // JSP로 포워딩
    }
}
```

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html>
<html>
<head>
    <title>동적인 웹사이트</title>
</head>
<body>
    <h1>현재 시간이 반환되는 동적 웹사이트입니다.</h1>
    <p>현재 시간 : <%= request.getAttribute("currentTime") %></p>
    <!-- 또는 EL 표현식을 사용 가능 -->
    <!-- <p>현재 시간 : ${currentTime}</p> -->
</body>
</html>
```

<br>

#### [Servlet + JSP]

<img src="/assets/img/24/10/06/servlet+jsp.png" alt="servlet+jsp" width="600">

- JSP만 사용하기에는 자바 코드 작성이 불편하고, Servlet만 사용하기에는 HTML로 인해 가독성이 떨어지기 때문에 두가지를 결합하는 형태로 많이 쓰인다고 한다.
- 여기에 MVC 패턴을 도입하여 **JSP가 View를 담당**하고 **Servlet이 Controller를 담당**하여 효율적인 WAS 개발이 가능해졌다.

<br>

## 스프링 프레임워크
- 현대에 자주 쓰이는 스프링 프레임워크는 MVC 아키텍처 패턴를 채용하면서 서블릿을 기반으로 설계되었다.
- 서블릿이 훨씬 추상화되어 개발자는 서블릿과 관련된 코드도 보지 않아도 된다.
- Spring MVC에서는 `DispatcherServlet`이 서블릿 역할을 수행하여 클라이언트의 요청을 적절한 컨트롤러로 분배하고, 그 결과를 뷰로 전달하여 응답을 생성한다.

<br>

#### [스프링 예시 코드]
- 서블릿 관련 코드가 드러나지 않는다.
- `Model` 에 값을 담으면 `DispatcherServlet`을 통해 호출된 뷰로 자동으로 전달된다.

```java
@Controller
public class HelloController {

    @GetMapping("/hello")
    public String hello(Model model) {
        // 현재 시간을 모델에 저장
        model.addAttribute("currentTime", new Date().toString());
        
        // hello.jsp로 이동
        return "hello";
    }
}
```

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html>
<html>
<head>
    <title>동적인 웹사이트</title>
</head>
<body>
    <h1>현재 시간이 반환되는 동적 웹사이트입니다.</h1>
    <p>현재 시간 : ${currentTime}</p>
</body>
</html>
```

<br>

## 마무리
- 서블릿이나 JSP 같은 레거시 기술들을 들으면서 그저 *레거시*라는 이유로, 아무것도 몰랐기에 어려워했었다.
- 하지만 결국 자바라는 언어로 웹 요청을 더 효율적으로 처리하기 위해 만들어진 기술들이라는 것을 깨닫고 이제는 더이상 두렵지 않게 되었다.
    > 물론, 지금까지 발전해온 스프링 프레임워크가 있는 이상 이 레거시 기술들을 사용할 일은 없을 것이다.