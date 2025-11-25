# HttpMessageConverter

InputStream에서 HTTP request의 body를 읽는 역할
OutputStream에서 HTTP response의 body에 데이터를 쓰는 역할

## MIME type
text/* -> StringHttpMessageConverter
application/x-www-form-urlencoded -> FormHttpMessageConverter
application/json -> MappingJackson2HttpMessageConverter

## DispatcherServlet의 흐름
![image.png](/spring/img/dsimage.png)

Servlet.class -> GenericHttpServlet.class -> FrameworkServlet.class -> DispatcherServlet.class의 doService() 메서드 호출

스프링 어플리케이션이 실행될 때 DispatcherServlet을 자동으로 등록
@ComponentScan으로 @RequestMapping이 붙은 Handler 들을 모두 매핑

클라이언트의 요청에 의해 doService()가 호출되면 doDispatch() 메서드가 실행되고
getHandler(HttpServletRequest) 메서드를 통해 적절한 handler 탐색
@RequestMapping 의 HandlerMapping에서 적잘한 HandlerAdapter를 탐색
HandlerAdapter의 handle() 메서드를 호출하여 실제 Controller를 통한 로직 수행

참조
https://okimaru.tistory.com/entry/HttpMessageConverter
https://sedangdang.tistory.com/305