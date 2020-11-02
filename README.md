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
#### 등록 방법
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
<!-- TestFilter 필터의 url 매핑 패턴을 /*(모든 url)로 설정 -->
<filter-mapping>
    <filter-name>TestFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```
