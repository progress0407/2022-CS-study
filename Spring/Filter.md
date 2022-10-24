# Spring - Filter

### 우선 필터부터 구경해봅시다.

![image](https://user-images.githubusercontent.com/66164361/197480463-ed6c84f7-bedd-4b97-acdd-a448645c9794.png)

```java
/**
 * Examples that have been identified for this design are<br>
 * 1) Authentication Filters <br>
 * 2) Logging and Auditing Filters <br>
 * 3) Image conversion Filters <br>
 * 4) Data compression Filters <br>
 * 5) Encryption Filters <br>
 * 6) Tokenizing Filters <br>
 * 7) Filters that trigger resource access events <br>
 * 8) XSL/T filters <br>
 * 9) Mime-type chain Filter <br>
 *
 * @since Servlet 2.3
 */
public interface Filter {

    public default void init(FilterConfig filterConfig) throws ServletException {}

    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;

    public default void destroy() {}
}
```

필터는 위와 같은 이유로 디자인 되었다고 합니다.

- 인증
- 로깅
- 이미지 변환
- 데이터 압축
- 암호화
- 토큰 작업
- 자원 접근 이벤트

... 등등

### 메서드들

필터는 init, doFilter, destory 메서드가 있는데, 이 중에서 가장 큰 핵심 역할을 하는 것은 doFilter 메서드이고 반드시 구현해야 합니다.

init, destory에 대한 구현 여부는 그저 선택 사항입니다.

- init 메소드
  - 필터 객체를 초기화하는 메서드, 웹 컨테이너가 1회 init 메소드를 호출하여 필터 객체를 초기화한다.
 
 
- doFilter 메소드
  - url pattern에 맞는 모든 HTTP 요청을 처리한다. 
  - FilterChain의 `doFilter` 메서드를 통해 다음 대상으로 요청 처리
 
- destroy 메소드
  - 웹 앱 종료시 웹 컨테이너에 의해 1번 호출 되는 메서드


### 우테코 크루 모두 아시다시피...

스프링은 웹 요청의 공통 처리를 위해서 존재합니다.

request 객체를 가로채서 속성값을 추가할 수도 있고 변경할 수도 있습니다.

혹은 request객체에서 세션 값을 꺼내서 인증여부를 확인할 수도 있습니다.

스프링의 공통 처리를 하는 기능은 필터와 인터셉터가 있습니다.

### 필터와 인터셉터의 차이는 ??

다음은 인터셉터(`HandlerInterceptor`)의 클래스 상단 주석이다.

![image](https://user-images.githubusercontent.com/66164361/197487842-1d93b6a1-135b-4121-8174-6de030449f1a.png)

```
HandlerInterceptor is basically similar to a Servlet Filter, but in contrast to the latter it just allows custom pre-processing with the option of prohibiting the execution of the handler itself, and custom post-processing. Filters are more powerful, for example they allow for exchanging the request and response objects that are handed down the chain. Note that a filter gets configured in web.xml, a HandlerInterceptor in the application context.

HandlerInterceptor는 기본적으로 서블릿 필터와 유사하지만, 후자와는 대조적으로 핸들러 자체의 실행을 금지하는 옵션으로 사용자 정의 사전 처리 및 사용자 정의 후처리를 허용합니다. 필터는 더 강력합니다. 예를 들어 체인으로 전달되는 요청 및 응답 개체를 교환할 수 있습니다. 필터는 응용 프로그램 컨텍스트의 HandlerInterceptor인 web.xml에서 구성됩니다.


As a basic guideline, fine-grained handler-related preprocessing tasks are candidates for HandlerInterceptor implementations, especially factored-out common handler code and authorization checks. On the other hand, a Filter is well-suited for request content and view content handling, like multipart forms and GZIP compression. This typically shows when one needs to map the filter to certain content types (e.g. images), or to all requests.

기본 지침으로 세분화된 핸들러 관련 전처리 작업은 HandlerInterceptor 구현, 특히 팩터링된 공통 핸들러 코드 및 권한 부여 검사의 후보입니다. 반면에 필터는 다중 파트 양식 및 GZIP 압축과 같은 요청 콘텐츠 및 보기 콘텐츠 처리에 적합합니다. 이는 일반적으로 필터를 특정 콘텐츠 유형(예: 이미지) 또는 모든 요청에 매핑해야 하는 경우를 보여줍니다.
```

요약하자면 아래와 같이 될 것 같다 !

- 필터의 경우 인터셉터와 다르게 Reqeust, Response 객체 자체를 바꿀 수 있다. 
- 세분화된 작업 주리 인터셉터에서 하며, 인가 확인의 경우도 이곳에서 합니다.  
  필터의 경우 mulit-part 및 gzip 압축 등을 처리하기 좋습니다.

### 밸덩에서 찾아본 Filter vs Interceptor

![image](https://user-images.githubusercontent.com/66164361/197488961-1809b55c-ce8f-4ed8-a876-7a1363201043.png)

- 필터
  - DispatcherServlet 전의 요청을 가로채므로 큰 범위의 작업을 처리하는 게 좋다
    - 인증
    - 로깅 및 감사
    - 이미지 데이터 압축
    - Spring MVC로부터 분리하고자 하는 기능

- 인터셉터
  - 컨트롤러와 DispatcherServlet 사이의 요청을 가로채기 때문에 앞에 있는 것보다는 세분화된 작업을 하는게 좋다
    - 횡단 관심사에 대한 application Logging
    - 세분화된 인가 확인
    - 스프링 bean이나 model을 다루는 것

### Filter가 스프링의 Bean을 주입받을 수 있다?!

우선 Filter 자체도 Bean으로 등록되는지 부터 알아보자 !
본래 Filter 자체는 스프링이 아닌 서블릿에서 제공하는 기술이므로 서블릿 컨테이너에 의해 생성되고 등록이 된다. 따라서 스프링 빈으로 등록이 불가했으며 빈을 주입받을 수도 없었다. 

그래서 Spring 1.2 부터 나온 것이 `DelegatingFilterProxy` 이다. `DelegatingFilterProxy`는 서블릿 컨테이너에 등록되며 요청이 왔을 때 이 클래스는 스프링 컨테이너 내부에 등록된 실제 Bean에 요청을 위임한다. 그래서 이름이 `위임하는 필터 프록시`인 것 같다! 특히 스프링 시큐리티는 이 기술에 굉장히 의존적이라고 한다 !

> https://uchupura.tistory.com/24  
> 해당 블로그의 그림이 너무 잘 표현해주었다 !  

### 스프링에 의해 관리받을 수 있는 Filter 만들기


### 부트 이전

아래와 같이 설정을 해주어야만 했었다 !

```
public class MyWebApplicationInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

   public void onStartup(ServletContext servletContext) throws ServletException {
      super.onStartup(servletContext);
      servletContext.addFilter("myFilter", DelegatingFilterProxy.class);
   }
}

// 출처: https://mangkyu.tistory.com/221 [MangKyu's Diary:티스토리]}
```



### 스프링 부트 이후

Spring Boot의 경우 내장 웹서버를 포함하면서 부트 서블릿 컨테이너도 제어가 가능해졌다.

현재는 아래와 같은 방법으로 Filter를 만들 수 있다.




```java
@Component("loggingFilter")
public class CustomFilter implements Filter {

    private static Logger LOGGER = LoggerFactory.getLogger(CustomFilter.class);

    @Override
    public void doFilter(
      ServletRequest request, ServletResponse response, 
      FilterChain chain) throws IOException, ServletException {
 
        // something to write ...
        chain.doFilter(request, response);
    }
}
```

이 때 `@Component`은 스프링 컨테이너에 등록을 해주고 `DelegatingFilterProxy`가 필터 체인을 초기화하는 동안 우리의 필터 클래스를 찾아서 등록한다고 한다.

원래 이 클래스는 `Spring Security`의 `FilterToBeanProxy`에서 영감을 받았다고 한다. (Ben Alex 작성)

```
// DelegatingFilterProxy 주석 마지막 줄
This class was originally inspired by Spring Security's FilterToBeanProxy class, written by Ben Alex.
```

