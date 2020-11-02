# SpringMVC_Lifecycle
- SpringMVC에서 request에 대한 Lifecycle에 대한 설명과 bean설정 관련 가이드 레포지토리입니다. (BCSDLab BackEnd Beginner 참고용)
- 이 문서는 김주원이 작성하였습니다. 다수의 레퍼런스를 참고하였으며, 참고 링크는 문서 중간중간에 달아두었습니다.
## Spring MVC Request Lifecycle
전체적인 구조는 다음과 같습니다.
![](https://images.velog.io/images/damiano1027/post/94f7d7a6-f4b7-4c22-97f4-35e6791b8963/image.png)
(사진 출처: 대학교 선배 정종우형)  

요소들을 하나하나 살펴보며 역할과 등록 방법을 알아봅시다.
### Filter
- 웹 애플리케이션의 전체적인 필터링(설정)을 담당합니다.
- request와 response에 대한 설정을 해줄 수 있습니다.
#### 설정 및 등록 방법
여러 방법이 있지만 그중에서도 web.xml에 등록하는 방법을 설명하겠습니다.
1. 원하는 패키지 위치에 원하는 이름으로 Filter 자바 파일을 생성합니다. 저는 filter 패키지를 만들어서 그 안에 TestFilter.java 파일을 생성하였습니다.
2. javax.servlet 패키지의 Filter 인터페이스를 implement 합니다. (javax.servlet 패키지 라이브러리를 사용하기 위해 [여기](https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api)에서 의존성 xml코드를 가져와 pom.xml에 의존성 설정을 해주도록 합시다.
3. init(), doFilter(), destroy() 메소드를 오버라이드 합니다.
    - init()
      - 웹 컨테이너(Tomcat)에서 필터 객체가 생성되면서 호출되는 메소드
      - 주로 초기화 작업
      - 인자로 들어오는 FilterConfig 객체는 정보를 필터에 전달해주기위한 객체입니다. ([참고 링크](https://tomcat.apache.org/tomcat-5.5-doc/servletapi/javax/servlet/FilterConfig.html))
    - doFilter()
      - 필터링을 수행하는 메소드
      - 첫번째와 두번째 인자로 ServletRequest 객체와 ServletResponse 객체가 들어오는데, 둘은 각각 request와 response에 관련하여 필터링을 할 수 있게 해줍니다.
        - [ServletRequest에 대한 설명 링크](https://ckddn9496.tistory.com/50?category=391769) 
        - [ServletResponse에 대한 설명 링크](https://ckddn9496.tistory.com/51?category=391769)
      - 세번째 인자로 들어오는 FilterChain 객체를 통해 doFilter() 메소드를 호출하여 다음 필터로 이동할 수 있습니다. 쉽게 생각하면 체인을 따라 다음 필터로 이동하게 해주는 것입니다. 
        - 다음 필터에서도 연속적으로 request와 response에 대해 처리할 수 있도록 인자로 ServletRequest객체와 ServletResponse 객체를 넣어줍니다.
        - doFilter() 호출 전에는 요청에 대한 필터링을 수행하고, doFilter()를 호출한 다음, doFilter() 호출 후에는 응답에 대한 필터링을 수행합니다. 
    - destroy()
      - 필터 객체가 소멸되면서 호출되는 메소드
      - 주로 자원 반납 작업
  
``` java
package filter;

import javax.servlet.*;
import java.io.IOException;

public class TestFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        // 필터 초기화 
    }
    
    @Override 
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        // 1. request를 이용하여 요청에 대한 필터링 수행
        
        // 2. 다음 필터로 이동 
        chain.doFilter(request, response);
        
        // 3. response를 이용하여 응답에 대한 필터링 수행
    }
    
    @Override
    public void destroy() {
        // 자원 반납
    }
}
``` 
4. web.xml에 filter를 등록합니다.
``` xml
<!-- filter 패키지의 TestFilter 클래스를 TestFilter라는 이름으로 등록 -->
<filter>
    <filter-name>TestFilter</filter-name>
    <filter-class>filter.TestFilter</filter-class>
</filter>
<!-- TestFilter 필터의 url 매핑 패턴을 설정 -->
<filter-mapping>
    <filter-name>TestFilter</filter-name>
    <!-- /*(모든 요청)에 대해 매핑 -->
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### DispatcherServlet
- 들어오는 모든 요청을 우선적으로 받아 처리해주는 서블릿입니다.
- HandlerMapping에게 request에 매핑할 컨트롤러 검색을 요청하고, HandlerMapping으로부터 Controller 정보를 반환받아 해당 Controller와 매핑시킵니다.
#### Bean 등록 방법
1. Spring MVC 라이브러리를 적용하면 자동으로 생성되는 dispatcher-servlet.xml이 있을 것입니다. 여기에 다음과 같이 annotation-driven 설정과 component-scan 설정을 해줍니다.
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <mvc:annotation-driven></mvc:annotation-driven>
    <!-- base-package에 입력된 패키지 아래에서 매핑될 컨트롤러를 찾게된다. -->
    <context:component-scan base-package="controller"></context:component-scan>
</beans>
```

2. dispatcher-servlet.xml을 토대로 web.xml에 servlet을 등록합니다.
``` xml
<!-- DispatcherServlet Bean 등록 -->
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
<!-- dispatcher Bean에 대한 매핑 패턴 설정 -->
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <!-- url pattern을 '/'으로 설정하게 되면 default servlet(요청에 대한 패턴을 가진 서블릿이 없을때 모든 요청을 받아들이는 servlet)이 된다. -->
    <url-pattern>/</url-pattern>
</servlet-mapping>
```
위 코드에서 servlet-name은 DispatcherServlet이 기본(default)으로 참조할 빈 설정 파일 이름의 prefix가 되는데, (servlet-name)-servlet.xml 같은 형태입니다. 위 코드와 같이 web.xml을 작성했다면 DispatcherServlet은 기본으로 /WEB-INF/dispatcher-servlet.xml을 찾게 됩니다. ([참고 링크](https://jaehun2841.github.io/2018/08/11/2018-08-11-spring-dispatcher-servlet/#%EA%B8%B0%EB%B3%B8-%EC%84%A4%EC%A0%95))

### HandlerMapping
- DispatcherServlet으로부터 검색을 요청받은 Controller를 찾아 정보를 반환해줍니다.
#### Bean 등록방법
- Spring MVC에서 기본적으로 제공하는 BeanNameUrlHandlerMapping이라는 HandlerMapping이 있습니다. 이것은 개발자가 아무것도 설정하지 않아도 자동적으로 등록됩니다. 따라서 등록 방법을 따로 살펴볼 필요는 없습니다.  
그래도 다른 HandlerMapping들의 종류나 등록 방법에 대해 공부하고싶다면 [여기](https://www.baeldung.com/spring-handler-mappings)를 참고해보시기 바랍니다.

### HandlerInterceptor
- request가 Controller에 매핑되기 전 앞단에서 부가적인 로직을 끼워넣습니다.
- 주로 세션, 쿠키, 권한 인증 로직에 많이 사용됩니다.
#### 설정 및 Bean 등록 방법
1. 원하는 패키지 위치에 원하는 이름으로 interceptor 자바 파일을 생성합니다. 저는 interceptor 패키지를 만들어서 그 안에 TestInterceptor.java 파일을 생성하였습니다.
2. org.springframework.web.servlet.handler 패키지의 HandlerInterceptorAdapter 클래스를 extends 합니다.
3. 필요에 따라 preHandle() 메소드 또는 postHandle() 메소드를 오버라이드 합니다.
    - preHandle() 
        - 컨트롤러에 요청이 매핑되기 전에 호출됩니다.
        - 세번째 인자인 Object 객체에는 HandlerMapping이 찾아준 컨트롤러의 메소드를 참조할 수 있는 객체입니다.
        - 반환형은 boolean이며, true를 반환하면 다음 단계로 넘어가고 false를 반환하면 다음 단계가 실행되지 않습니다.
    - postHandle()
        - 컨트롤러가 실행된 후에 호출됩니다.  
        
   더 자세한 내용과 추가 메소드는 [여기](https://victorydntmd.tistory.com/176)를 참고해보시기 바랍니다.
``` java
package interceptor;

import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class TestInterceptor extends HandlerInterceptorAdapter {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // do something
    }
}
```
4. dispatcher-servlet.xml에 Bean으로 등록합니다.
``` xml
<mvc:interceptors>
    <mvc:interceptor>
        <!-- 매핑 방식 설정 -->
        <mvc:mapping path="/**" />
        <!-- 클래스 절대 경로 -->
        <bean class="interceptor.TestInterceptor" />
    </mvc:interceptor>
</mvc:interceptors>
```
### Controller
- request와 매핑되는 곳입니다.
- request에 대해 어떤 비즈니스 로직(service)으로 처리할 것인지를 결정하고, 그에 맞는 Service를 호출합니다.
#### Bean 등록 방법
1. 클래스 상단에 @Controller 어노테이션을 선언합니다.
``` java
@Controller
public class TestController {
    // do something
}
```
2. dispatcher-servlet.xml에서 component-scan 설정이 잘 되었는지 확인합니다.

### Service
- 데이터 처리 및 가공을 위한 비즈니스 로직을 수행합니다.
- request에 대한 실질적인 로직을 수행합니다.
- Repository를 통해 DB에 접근하여 데이터의 CRUD를 처리합니다.
#### Bean 등록 방법
1. 클래스 상단에 @Service 어노테이션을 선언합니다.
``` java
@Service
public class TestServiceImpl implements TestService {
    // do something
}
```
2. component-scan이 잘 설정되었는지 확인합니다. Controller를 제외한 다른 클래스들에 대한 component-scan 등록은 dispatcher-servlet.xml에도 등록할 수 있고, web.xml의 contenxt-param 엘리먼트의 param-value 엘리먼트 값으로 들어간 xml 파일에 설정하여도 됩니다.   
Spring MVC 프로젝트를 생성하면 자동으로 web.xml에 이렇게 정의되어 있을 것입니다.
``` xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/applicationContext.xml</param-value>
</context-param>
```
위 코드는 앞서 살펴보았던 dispacther-servlet.xml 외에 applicationContext.xml도 스프링 컨테이너에서 참조하도록 설정하는 것입니다.
따라서 component-scan 설정을 다음과 같이 applicationContext.xml에 해주어도 좋습니다.
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- service 패키지 아래에 있는 모든 위치를 스캔하여 bean 등록 -->
    <context:component-scan base-package="service"></context:component-scan>
</beans>
```

### ViewResolver
- Controller에서 리턴한 View의 이름을 DispactherServlet으로부터 넘겨받고, 해당 view를 렌더링합니다.
- 렌더링한 View는 DispactherServlet으로 전달하고, DispatcherServlet에서는 해당 View 화면을 Response 합니다.
#### Bean 등록 방법
dispatcher-servlet.xml에 InternalResourceViewResolver 클래스에 대한 Bean을 등록한다.
``` xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <!-- value에는 파일 경로를 만들어주기위한 접두사가 들어간다. -->
    <property name="prefix" value="/WEB-INF/views/"></property>
    <!-- value에는 파일 경로를 만들어주기위한 접미사가 들어간다. -->
    <property name="suffix" value=".jsp"></property>
</bean>
```
