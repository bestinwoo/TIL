# Dispatcher Servlet (디스패처 서블릿)

상태: 진행 중

## Dispatcher Servlet이란?

Spring의 웹 MVC 프레임워크는 다른 웹 MVC 프레임워크와 마찬가지로 Request 기반이며 컨트롤러에 요청을 전달하고 웹 애플리케이션 개발을 용이하게 하는 기타 기능을 제공하는 중앙 서블릿을 중심으로 설계되었다.

DispatcherServlet은 HTTP 프로토콜로 들어오는 모든 요청을 가장 먼저 받아 적합한 컨트롤러에 위임해주는 프론트 컨트롤러(Front Controller)로 정의할 수 있다.

클라이언트로부터 요청이 오면 톰캣과 같은 서블릿 컨테이너가 요청을 받게 된다. 그리고 이 모든 요청을 프론트 컨트롤러인 디스패처 서블릿이 가장 먼저 받게 된다. 디스패처 서블릿은 공통적인 작업을 먼저 처리한 후에 해당 요청을 처리해야 하는 컨트롤러에 작업을 위임한다.

프론트 컨트롤러는 주로 서블릿 컨테이너의 제일 앞에서 서버로 들어오는 클라이언트의 모든 요청을 받아서 처리해주는 컨트롤러로 MVC 구조에서 함께 사용되는 디자인 패턴이다.

## DispatcherServlet Workflow

클라이언트에서 HTTP 프로토콜로 요청을 했을 때, DispatcherServlet에서 이루어지는 동작 흐름은 아래 그림과 같다. 아래 그림에서 Front Controller에 해당하는 부분이 DispatcherServlet이다.

![이미지 출처 : [https://docs.spring.io/spring-framework/docs/3.0.0.M4/spring-framework-reference/html/ch15s02.html](https://docs.spring.io/spring-framework/docs/3.0.0.M4/spring-framework-reference/html/ch15s02.html)](https://docs.spring.io/spring-framework/docs/3.0.0.M4/spring-framework-reference/html/images/mvc.png)

이미지 출처 : [https://docs.spring.io/spring-framework/docs/3.0.0.M4/spring-framework-reference/html/ch15s02.html](https://docs.spring.io/spring-framework/docs/3.0.0.M4/spring-framework-reference/html/ch15s02.html)

더 자세한 동작 방식은 아래 그림과 같다. 아래 그림은 RestController를 사용할 때의 동작 방식이다.

![dispatcherServlet1](https://user-images.githubusercontent.com/33615669/199640112-30907680-a697-4a77-8706-b5c3df5c4f21.png)

이미지 출처 : [https://mangkyu.tistory.com/18](https://mangkyu.tistory.com/18)

처리 과정의 단계는 아래와 같다.

1. 클라이언트의 요청을 디스패처 서블릿이 받는다.

2. 요청 정보를 통해 요청을 위임할 컨트롤러를 찾는다.

3. 요청을 컨트롤러로 위임할 핸들러 어댑터를 찾아서 전달한다.

4. 핸들러 어댑터가 컨트롤러로 요청을 위임한다.

5. Service 단에서 비지니스 로직을 처리한다.

6. 컨트롤러가 반환값을 반환한다.

7. 핸들러 어댑터가 반환 값을 처리한다.

8. 서버의 응답을 클라이언트로 반환한다.

   

RestController가 아닌 Controller일 경우 Response Body를 작성하는 대신 View Resolver가 View를 찾아 Model을 전달하고 렌더링하는 과정이 된다는 차이가 있다.

결론적으로 디스패처 서블릿이 하는 일은 **요청을 처리할 컨트롤러를 찾아서 위임하고 그 결과를 받아 클라이언트로 반환하는 것이다.**

동작 과정을  더 자세히 살펴보면 다음과 같다.



### 1. 클라이언트의 요청을 디스패처 서블릿이 받음

WAS에서 HttpServletRequest와 HttpServletResponse 객체를 생성하여 DispatcherServlet에 전달한다. DispatcherServlet은 해당 객체를 전달받아 응답을 생성하기 위해 doService() 메서드를 호출한다. 그리고 메서드 내부에서 doDispatch() 메서드를 호출한다.

```java
@Override
	protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
		// ...

		try {
			doDispatch(request, response);
		}
	}
```



### 2. 요청 정보를 통해 요청을 위임할 컨트롤러를 찾음

디스패처 서블릿은 요청을 처리할 컨트롤러를 찾고 해당 메서드를 호출해야 한다. 

doDispatch() 메서드에서는 디스패처 서블릿이 호출하는 모든 메서드가 담겨 있다. 먼저 웹 요청을 처리할 수 있는 Handler(Controller)를 찾기 위해 getHandler() 메서드를 호출한다. 이때, 해당 메서드의 반환 타입은 HandlerExecutionChain이다. 

HandlerExecutionChain으로 감싸는 이유는 컨트롤러로 요청을 넘겨주기 전에 처리해야 하는 인터셉터 등을 포함하기 위해서이다.

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
// Determine handler for the current request.
				mappedHandler = getHandler(processedRequest);
				if (mappedHandler == null) {
					noHandlerFound(processedRequest, response);
					return;
				}
}

@Nullable
	protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		if (this.handlerMappings != null) {
			for (HandlerMapping mapping : this.handlerMappings) {
				HandlerExecutionChain handler = mapping.getHandler(request);
				if (handler != null) {
					return handler;
				}
			}
		}
		return null;
	}
```



### 3. 요청을 컨트롤러로 위임할 핸들러 어댑터를 찾아서 전달한다.

디스패처 서블릿은 컨트롤러로 요청을 직접 위임하는 것이 아니라 HanlderAdapter를 통해 컨트롤러로 요청을 위임한다. 이때 어댑터 인터페이스를 통해 컨트롤러를 호출하는 이유는 컨트롤러의 구현 방식이 다양하기 때문이다. 어댑터 패턴을 적용함으로써 컨트롤러의 구현 방식에 상관 없이 요청을 위임할 수 있다.



### 4. 핸들러 어댑터가 컨트롤러로 요청을 위임한다.

핸들러 어댑터가 컨트롤러로 요청을 넘기기 전에 공통적인 전/후처리 과정이 필요하다. 대표적으로 인터셉터들을 포함해 요청 시에 @RequestParam, @RequestBody 등을 처리하기 위한 ArgumentResolver들과 응답 시에 ResponseEntity Body를 Json으로 직렬화 처리를 하는 ReturnValueHandler 등이 어댑터에서 컨트롤러로 전달되기 전에 처리된다. 



### 5. 비지니스 로직을 처리한다.

컨트롤러는 서비스를 호출하고 개발자가 작성한 비지니스 로직이 처리된다.



### 6. 컨트롤러가 반환값을 반환한다.

비지니스 로직이 처리된 후에는 컨트롤러가 반환 값을 반환한다. RestController의 경우 주로 ResponseEntity를 반환하고 Controller의 경우는 View 이름을 반환할 수도 있다.



### 7. HandlerAdapter가 반환값을 처리한다.

HandlerAdapter는 컨트롤러로부터 받은 응답을 응답 처리기인 ReturnValueHandler가 후처리한 후에 디스패처 서블릿으로 돌려준다. 만약 컨트롤러가 ResponseEntity를 반환하면 HttpEntityMethodProcessor가 MessageConverter를 사용해 응답 객체를 직렬화하고 응답 상태를 설정한다. 만약 컨트롤러가 View 이름을 반환하면 ViewResolver를 통해 View를 반환한다.



### 8. 서버의 응답을 클라이언트로 반환한다.

디스패처 서블릿을 통해 반환되는 응답은 다시 필터들을 거쳐 클라이언트에게 반환된다.



## 참고

[https://docs.spring.io/spring-framework/docs/3.0.0.M4/spring-framework-reference/html/ch15s02.html](https://docs.spring.io/spring-framework/docs/3.0.0.M4/spring-framework-reference/html/ch15s02.html)

[https://tecoble.techcourse.co.kr/post/2021-06-25-dispatcherservlet-part-1/](https://tecoble.techcourse.co.kr/post/2021-06-25-dispatcherservlet-part-1/)

[https://mangkyu.tistory.com/18](https://mangkyu.tistory.com/18)