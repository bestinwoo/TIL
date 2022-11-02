# Servlet과 ServletContainer



## Servlet

서블릿이란 클라이언트 요청을 처리하고, 그 결과를 반환하는 웹 프로그래밍 기술로 자바로 구현된 CGI (Common Gateway Interface)이다.

이전의 웹 프로그램들은 클라이언트의 요청에 대한 응답으로 이미 만들어진 정적인 페이지를 넘겨주었지만 웹 프로그래밍이 발전하면서 동적인 페이지를 제공할 수 있게 되었다. 

동적인 페이지를 제공하기 위해서는 웹 서버가 다른 곳에 도움을 요청하여 동적인 페이지를 작성해야 한다. 여기서 웹 서버가 동적인 페이지를 제공할 수 있도록 도와주는 어플리케이션이 서블릿이며, 동적인 페이지를 생성하는 애플리케이션이 CGI이다.

**Servlet의 특징**

- 클라이언트의 요청에 대해 동적으로 작동하는 웹 어플리케이션 컴포넌트
- html을 사용하여 요청에 응답한다.
- Java Thread를 이용하여 동작한다.
- MVC 패턴에서 Controller로 이용된다.
- HTTP 프로토콜 서비스를 지원하는 HttpServlet 클래스를 상속받는다.
- UDP보다 속도가 느리다.
- HTML 변경이 Servlet을 재컴파일해야 하는 단점이 있다.

**Servlet의 동작 방식**

![sevlet](https://user-images.githubusercontent.com/33615669/199551937-8cdf09dd-94a2-4e2f-b80b-20225d9f3c37.png)

1. 클라이언트가 URL을 입력하면 HTTP Request가 서블릿 컨테이너로 전송된다.
2. 요청을 전송 받은 서블릿 컨테이너는 HttpServletRequest, HttpServletResponse 객체를 생성한다.
3. web.xml을 기반으로 클라이언트가 요청한 URL이 어느 서블릿에 대한 요청인지 찾는다.
4. 해당 서블릿에서 service 메서드를 호출하고 GET, POST 여부에 따라 doGet, doPost를 호출한다.
5. doGet(), doPost() 메서드는 동적 페이지를 생성한 후 HttpServletResponse 객체에 응답을 보낸다.
6. 응답이 끝나면 HttpServletRequest, HttpServletResponse 객체를 소멸시킨다.

**Servlet의 생명주기**

![sevlet_life_cycle](https://user-images.githubusercontent.com/33615669/199551938-b276c14c-a35f-4878-957d-bb4d1901b3e6.png)

1. 클라이언트 요청이 들어오면 컨테이너는 서블릿이 메모리에 있는지 확인한다. 메모리에 없다면 init() 메서드를 호출하여 메모리에 적재한다. 실행중 서블릿이 변경될 경우, 기존 서블릿을 파괴하고 init()을 통해 새로운 내용을 다시 메모리에 적재한다.
2. init()이 호출된 후 클라이언트의 요청에 따라 service() 메서드를 통해 요청에 대한 응답이 doGet(), doPost()로 분기된다. 이 때 HttpServletRequest, HttpServletResponse에 의해 request와 response 객체가 제공된다.
3. 컨테이너가 서블릿에 종료 요청을 하면 destroy() 메서드가 호출되는데 마찬가지로 한번만 실행되며, 종료시 처리해야할 작업들은 destroy() 메서드를 오버라이딩하여 구현한다.

## Servlet Container (서블릿 컨테이너)

서블릿 컨테이너는 서블릿을 관리해주는 컨테이너로 서블릿 컨테이너는 클라이언트의 요청(Request)를 받아주고 응답(Response)할 수 있도록 웹 서버와 소켓으로 통신한다. 대표적인 예로는 톰캣이 있다.

**Servlet Container의 역할**

1. 웹 서버와의 통신 지원
   
    소켓을 만들고 웹서버와 통신하는 과정을 API로 제공하여 복잡한 과정을 생략해줌으로써 개발자가 서블릿에 구현해야 할 비지니스 로직에 초점을 두게끔 도와준다.
    
2. 서블릿 생명주기 관리
   
    서블릿 컨테이너는 서블릿 클래스를 로딩하여 인스턴스화하고, 초기화 메서드를 호출하고, 요청이 들어오면 적절한 서블릿 메서드를 호출한다. 또한 서블릿 역할이 끝난 후에는 GC(가비지 컬렉션)을 진행하여 소멸시킨다.
    
3. 멀티 스레드 지원 및 관리
   
    서블릿 컨테이너는 요청이 올 때 마다 새로운 스레드를 생성하는데, HTTP 서비스 메서드를 실행하고 나면, 스레드는 자동으로 죽게된다. 서버가 다중 스레드를 생성 및 운영해주니 스레드의 안정성에 대해서 걱정하지 않아도 된다.





## 참고

https://mangkyu.tistory.com/14

https://tecoble.techcourse.co.kr/post/2021-05-23-servlet-servletcontainer/