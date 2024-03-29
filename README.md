﻿# 스프링 MVC 1편(2/2) - 스프링
> 김영한 강사님 스프링 MVC 1편 스프링 MVC 내용 정리입니다.

***

# 4. 스프링 MVC
### 4.1. 스프링 MVC 구조 이해
![image](https://github.com/helloJosh/spring-servlet-study/assets/37134368/8f6d23f3-1b27-4189-82f9-91cec280713f)
**직접 만든 프레임워크 스프링 MVC 비교**
* FrontController DispatcherServlet
* handlerMappingMap HandlerMapping
* MyHandlerAdapter HandlerAdapter
* ModelView ModelAndView
* viewResolver ViewResolver
* MyView View

#### 4.1.1. DispatcherServlet 구조 살펴보기
`org.springframework.web.servlet.DispatcherServlet`
* 스프링MVC도 Front Controller 패턴으로 구현되어 있음.
* Front Controller가 DispatcherServlet이다.
* DispatcherServlet도 부모 클래스에서 `HttpServlet`을 상속 받아서 사용하고, 서블릿으로 동작한다.
    + DispatcherServlet -> FrameworkServlet -> HttpServletBean -> HttpServlet
* 스프링 부트는 DispatcherServlet을 서블릿으로 자동으로 등록하면서 urlPatterns에 대해서 매핑한다.

#### 4.1.2. 요청 흐름
* 서블릿이 호출되면 `HttpServlet`이 제공하는 Service() 호출
* 스프링 MVC는 DispatcherServlet의 부모인 FrameworkServlet에서 Service()를 오버라이드 해둠
* FrameworkServlet.service()를 시작으로 여러 메서드가 호출되면서 DispatcherServlet.doDispatch()가 호출됨

#### 4.1.3. SpringMVC 동작 순서
1. 핸들러 조회 : 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다.
2. 핸들러 어댑터 조회 : 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
3. 핸들러 어댑터 실행 : 핸들러 어댑터를 실행한다.
4. 핸들러 실행 : 핸들러 어댑터가 실제 핸들러를 실행한다.
5. ModelAndView 반환 : 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환한다.
6. ViewReSolver 호출 : 뷰 리졸버를 찾고 실행한다.
7. View 반환 : 뷰 리졸버는 뷰의 논리 이름을 물리이름으로 바꾸고, 랜더링 역할을 담당하는 뷰 객체를 반환한다.
8. 뷰 렌더링 : 뷰를 통해서 뷰를 랜더링한다.

> 스프링 MVC의 강점은 DispatcherServlet 코드의 변경 없이, 원하는 기능을 변경하거나 확장할 수 있다. <br/>
> 이러한 인터페이스를 구현하여 `DispatcherServlet`에 등록하면 나만의 컨트롤러를 만들 수 있다. <br/><br/><br/>
> **주요 인터페이스 목록** <br/>
> * 핸들러 매핑: `org.springframework.web.servlet.HandlerMapping`
> * 핸들러 어댑터: `org.springframework.web.servlet.HandlerAdapter`
> * 뷰 리졸버: `org.springframework.web.servlet.ViewResolver`
> * 뷰: `org.springframework.web.servlet.View`

### 4.2. 핸들러 매핑과 핸들러 어댑터
```java
@Component("/springmvc/old-controller")
public class OldController implements Controller {
  @Override
  public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
    System.out.println("OldController.handleRequest");
    return null;
  }
}
```
* `@Component` : 이 컨트롤러는 `/springmvc/old-controller` 라는 이름의 스프링 빈으로 등록되었다.
* **빈의 이름으로 URL을 매핑**할 것이다.

#### 4.2.1. 컨트롤러의 호출 방법
![image](https://github.com/helloJosh/spring-servlet-study/assets/37134368/0faec9a8-8661-4bc7-ad27-5c7b2542b0c0)
* HandlerMapping(핸들러매핑)
    + 핸들러 매핑에서 컨트롤러를 찾아야한다.
    + 예) 스프링 빈의 이름으로 핸들러를 찾을 수 있는 핸들러 매핑이 필요하다.
    + 0 = RequestMappingHandlerMapping : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
    + 1 = BeanNameUrlHandlerMapping : 스프링 빈의 이름으로 핸들러를 찾는다.
* HandlerAdapter(핸들러 어댑터)
    + 핸들러 매핑을 통해서 찾은 핸들러를 실행할 수 있는 핸들러 어댑터가 필요하다.
    + 0 = RequestMappingHandlerAdapter : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
    + 1 = HttpRequestHandlerAdapter : HttpRequestHandler 처리
    + 2 = SimpleControllerHandlerAdapter : Controller 인터페이스(애노테이션X, 과거에 사용) 처리
 
#### 4.2.2. Controller 인터페이스 호출 순서

* **1. 핸들러 매핑으로 핸들러 조회**
  + 1. `HandlerMapping` 을 순서대로 실행해서, 핸들러를 찾는다.
  + 2. 이 경우 빈 이름으로 핸들러를 찾아야 하기 때문에 이름 그대로 빈 이름으로 핸들러를 찾아주는 `BeanNameUrlHandlerMapping` 가 실행에 성공하고 핸들러인 `OldController` 를 반환한다.
* **2. 핸들러 어댑터 조회**
  + 1. `HandlerAdapter` 의 `supports()` 를 순서대로 호출한다.
  + 2. `SimpleControllerHandlerAdapter` 가 `Controller` 인터페이스를 지원하므로 대상이 된다.
* **3. 핸들러 어댑터 실행**
  + 1. 디스패처 서블릿이 조회한 `SimpleControllerHandlerAdapter` 를 실행하면서 핸들러 정보도 함께 넘겨준다.
  + 2. `SimpleControllerHandlerAdapter` 는 핸들러인 `OldController` 를 내부에서 실행하고, 그 결과를 반환한다.

### 4.3. View Resolver(뷰리졸버)
```java
@Component("/springmvc/old-controller")
public class OldController implements Controller {
  @Override
  public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
    System.out.println("OldController.handleRequest");
    return new ModelAndView("new-form");
  }
}
```
`application.properties` 
```
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

#### 4.3.1. View Resolver 동작 방식
![image](https://github.com/helloJosh/spring-servlet-study/assets/37134368/36922bc4-0bb7-44ea-b21d-67235e88f8e2)
* 스프링 부트가 자동 등록하는 뷰 리졸버
  + 1 = BeanNameViewResolver : 빈 이름으로 뷰를 찾아서 반환한다. (예: 엑셀 파일 생성 기능에 사용)
  + 2 = InternalResourceViewResolver : JSP를 처리할 수 있는 뷰를 반환한다.

1. 핸들러 호출
    + 핸들러 어댑터를 통해 논리뷰 이름을 획득
2. ViewResolver 호출
    + 논리 뷰 이름으로 viewResolver를 순서대로 호출
    + BeanNameViewResolver는 이름의 스프링빈으로 등록된 뷰를 찾아야한다
    + 없으면 InternalResourceViewResolver가 호출된다.
3. InternalResourceViewResolver
    + 이 뷰 리졸버는 InternalResourceView를 반환한다.
4. 뷰 - InternalResourceView
    + `InternalResourceView` 는 JSP처럼 포워드 `forward()` 를 호출해서 처리할 수 있는 경우에 사용한다.
5. view.render()
    + `view.render()` 가 호출되고 `InternalResourceView` 는 `forward()` 를 사용해서 JSP를 실행한다.
  
### 4.4. 스프링 MVC - 시작하기

#### 4.4.1 @RequestMapping
* RequestMappingHandlerMapping : 가장 우선순위가 높은 핸들러 매핑
* RequestMappingHandlerAdapter : 가장 우선순위가 높은 핸들러 어댑터

#### 4.4.2. 회원등록폼 SpringMVCV1
```java
@Controller
public class SpringMemberFormControllerV1 {
  @RequestMapping("/springmvc/v1/members/new-form")
  public ModelAndView process() {
    return new ModelAndView("new-form");
  }
}
```
* `@Controller`
    + 스프링이 자동으로 스프링 빈으로 등록한다. (내부에 `@Component` 애노테이션이 있어서 컴포넌트 스캔의 대상이 됨)
    + 스프링 MVC에서 애노테이션 기반 컨트롤러로 인식한다.
* `@RequestMapping`
    + 요청 정보를 매핑한다. 해당 URL이 호출되면 이 메서드가 호출된다. 애노테이션 기반으로 동작하기 때문에, 메서드의 이름은 임의로 지으면 된다.
* `ModelAndView`
    + 모델과 뷰 정보를 담아서 반환하면 된다.
> 스프링 부트 2.0에서는 @Controller 없이 @RequestMapping만 있으면 작동했지만, 스프링 부트 3.0에서 부터는 @Controller가 있어야 스프링 컨트롤러로 인식한다.

```java
//스프링 빈 직접 등록
@Bean
SpringMemberFormControllerV1 springMemberFormControllerV1() {
  return new SpringMemberFormControllerV1();
}
```
* 위 빈을 스프링에 직접 등록시

#### 4.4.3. 회원 저장 SpringMVCV1
```java
@Controller
public class SpringMemberSaveControllerV1 {
  private MemberRepository memberRepository = MemberRepository.getInstance();
  @RequestMapping("/springmvc/v1/members/save")
  public ModelAndView process(HttpServletRequest request, HttpServletResponse response) {
    String username = request.getParameter("username");
    int age = Integer.parseInt(request.getParameter("age"));
    Member member = new Member(username, age);
    System.out.println("member = " + member);
    memberRepository.save(member);
    ModelAndView mv = new ModelAndView("save-result");
    mv.addObject("member", member);
    return mv;
  }
}
```
* mv.addObject("member", member) : 스프링이 제공하는 ModelAndView를 통해 Model 데이터를 추가할 떄는 addObject()를 사용ㅎ안다. 이 데이터는 이후 뷰를 렌더링할때 사용된다.

#### 4.4.4. 회원 목록 SpringMVCV2
```java
@Controller
public class SpringMemberListControllerV1 {
  private MemberRepository memberRepository = MemberRepository.getInstance();
  @RequestMapping("/springmvc/v1/members")
  public ModelAndView process() {
    List<Member> members = memberRepository.findAll();
    ModelAndView mv = new ModelAndView("members");
    mv.addObject("members", members);
    return mv;
  }
}
```

### 4.5. 스프링 MVC - 컨트롤러 통합
* 메서드 단위로 컨트롤러가 적용된것을 볼수 있음
``` java
/**
* 클래스 단위 -> 메서드 단위
* @RequestMapping 클래스 레벨과 메서드 레벨 조합
*/
@Controller
@RequestMapping("/springmvc/v2/members")
public class SpringMemberControllerV2 {
  private MemberRepository memberRepository = MemberRepository.getInstance();
  @RequestMapping("/new-form")
  public ModelAndView newForm() {
    return new ModelAndView("new-form");
  }
  @RequestMapping("/save")
  public ModelAndView save(HttpServletRequest request, HttpServletResponse response) {
    String username = request.getParameter("username");
    int age = Integer.parseInt(request.getParameter("age"));
    Member member = new Member(username, age);
    memberRepository.save(member);
    ModelAndView mav = new ModelAndView("save-result");
    mav.addObject("member", member);
    return mav;
  }
  @RequestMapping
  public ModelAndView members() {
    List<Member> members = memberRepository.findAll();
    ModelAndView mav = new ModelAndView("members");
    mav.addObject("members", members);
    return mav;
  }
}
```
* URL 중복을 맨위에 @RequestMapping("/springmvc/v2/members")를 통해 중복을 피할 수 있다.
* ModelAndView를 직접 생성해서 반환해줘야하는 불편함이 있다.

### 4.6. 스프링 MVC - 실무에서 사용하는 방식
```java
/**
* v3
* Model 도입
* ViewName 직접 반환
* @RequestParam 사용
* @RequestMapping -> @GetMapping, @PostMapping
*/
@Controller
@RequestMapping("/springmvc/v3/members")
public class SpringMemberControllerV3 {
  private MemberRepository memberRepository = MemberRepository.getInstance();
  @GetMapping("/new-form")
  public String newForm() {
    return "new-form";
  }
  @PostMapping("/save")
  public String save(@RequestParam("username") String username, @RequestParam("age") int age, Model model) {
    Member member = new Member(username, age);
    memberRepository.save(member);
    model.addAttribute("member", member);
    return "save-result";
  }
  @GetMapping
  public String members(Model model) {
    List<Member> members = memberRepository.findAll();
    model.addAttribute("members", members);
    return "members";
  }
}
```
* Model 파라미터 : save(), members()는 Model을 파라미터로 받는다.
* ViewName 직접 반환한다.
* @RequestParam 사용
    + 스프링은 HTTP 요청 파라미터를 @RequestParam으로 받을 수 있다.
    + @RequestParam("username")은 request.getParameter("username)과 같은 코드이다.
    + GET, POST Form 방식 모두 지원한다.
* @RequestMapping -> @GetMapping, @PostMapping
    + `@RequestMapping` 은 URL만 매칭하는 것이 아니라, HTTP Method도 함께 구분할 수 있다.
    + 예) `@RequestMapping(value = "/new-form", method = RequestMethod.GET)`
    + 이것을 편리하게 `@GetMapping` , `@PostMapping` 으로 사용한다.

****
# 5. 스프링 MVC - 기본기능
### 5.1. 로깅 알아보기
#### 5.1.1 로그 선언과 호출
* **로그 선언**
  + `private Logger log = LoggerFactory.getLogger(getClass());`
  + `private static final Logger log = LoggerFactory.getLogger(Xxx.class)`
  + `@Slf4j` : 롬복 사용 가능
* **로그 호출**
  + `log.info("hello")`

#### 5.1.2. Log 개요
```java
//@Slf4j
@RestController
public class LogTestController {
  private final Logger log = LoggerFactory.getLogger(getClass());
  @RequestMapping("/log-test")
  public String logTest() {
    String name = "Spring";

    log.trace("trace log={}", name);
    log.debug("debug log={}", name);
    log.info(" info log={}", name);
    log.warn(" warn log={}", name);
    log.error("error log={}", name);

    //로그를 사용하지 않아도 a+b 계산 로직이 먼저 실행됨, 이런 방식으로 사용하면 X
    log.debug("String concat log=" + name);
    return "ok";
  }
}
```
* `@RestController`
    + `@Controller`는 반환 값이 String이면 뷰 이름으로 인식. 뷰를 찾고 뷰가 랜더링 된다.
    + `@RestController`는 반환 값으로 뷰를 찾는 것이 아니라, HTTP 메시지 바디에 바로 입력된다.
* 로그가 출력되는 포멧 확인
    + 시간, 로그 레벨, 프로세스ID, 쓰레드 명, 클래스 명, 로그 메시지
* 로그 레벨 설정을 변경해서 출력 결과를 볼 수 있음
    + LEVEL : `TRACE > DEBUG > INFO > WARN > ERROR`
    + 개발 서버는 debug 출력
    + 운영 서버는 info 출력
* **로그 레벨 설정**
    + `application.properties`
```
#전체 로그 레벨 설정(기본 info)
logging.level.root=info
#hello.springmvc 패키지와 그 하위 로그 레벨 설정
logging.level.hello.springmvc=debug
```

#### 5.1.3. 올바른 로그 사용법
* `log.debug("data="+data)`
  + 로그 출력 레벨을 info로 설정해도 해당 코드에 있는 "data="+data가 실제 실행이 되어 버린다. 결과적으로 문자 더하기 연산이 발생한다.
* `log.debug("data={}", data)`
  + 로그 출력 레벨을 info로 설정하면 아무일도 발생하지 않는다. 따라서 앞과 같은 의미없는 연산이 발생하지 않는다.

#### 5.1.4. 로그 사용시 장점
* 쓰레드 정보, 클래스 이름 같은 부가 정보를 함께 볼 수 있고, 출력 모양을 조정할 수 있다.
* 로그 레벨에 따라 개발 서버에서는 모든 로그를 출력하고, 운영서버에서는 출력하지 않는 등 로그를 상황에 맞게 조절할 수 있다.
* 시스템 아웃 콘솔에만 출력하는 것이 아니라, 파일이나 네트워크 등, 로그를 별도의 위치에 남길 수 있다. 특히 파일로 남길 때는 일별, 특정 용량에 따라 로그를 분할하는 것도 가능하다.
* 성능도 일반 System.out보다 좋다. (내부 버퍼링, 멀티 쓰레드 등등) 그래서 실무에서는 꼭 로그를 사용해야 한다.

***
# 6. 요청매핑
### 6.1. MappingController
```java
@RestController
public class MappingController {
  private Logger log = LoggerFactory.getLogger(getClass());
  /**
  * 기본 요청
  * 둘다 허용 /hello-basic, /hello-basic/
  * HTTP 메서드 모두 허용 GET, HEAD, POST, PUT, PATCH, DELETE
  */
  @RequestMapping("/hello-basic")
  public String helloBasic() {
    log.info("helloBasic");
    return "ok";
  }
}
```
* `@RequestMapping("/hello-basic")`
  + `/hello-basic` URL 호출이 오면 이 메서드가 실행되도록 매핑한다.
  + 대부분의 속성을 `배열[]` 로 제공하므로 다중 설정이 가능하다. `{"/hello-basic", "/hello-go"}`

> **스프링 부트 3.0 이전** <br/>
> 다음 두가지 요청은 다른 URL이지만, 스프링은 다음 URL 요청들을 같은 요청으로 매핑한다.<br/>
> 매핑: `/hello-basic`<br/>
> URL 요청: `/hello-basic` , `/hello-basic/`<br/>
> **스프링 부트 3.0 이후**<br/>
> 스프링 부트 3.0 부터는 `/hello-basic` , `/hello-basic/` 는 서로 다른 URL 요청을 사용해야 한다.<br/>

### 6.2. HTTP 메서드 매핑
```java
/**
* method 특정 HTTP 메서드 요청만 허용
* GET, HEAD, POST, PUT, PATCH, DELETE
*/
@RequestMapping(value = "/mapping-get-v1", method = RequestMethod.GET)
  public String mappingGetV1() {
  log.info("mappingGetV1");
  return "ok";
}
```
* `@RequestMapping`에 `method`속성으로 HTTP 메서드를 지정하지 않으면 HTTP 메서드와 무관하게 호출된다. 즉 모두 허용(GET,HEAD,POST,PUT,PATCH,DELETE)
* 위 코드에 POST 요청을 하면 스프링은 MVC HTTP 405 상태코드(Method Not Allowed)를 반환한다

### 6.3 HTTP 메서드 매핑 축약
```java
/**
* 편리한 축약 애노테이션 (코드보기)
* @GetMapping
* @PostMapping
* @PutMapping
* @DeleteMapping
* @PatchMapping
*/
@GetMapping(value = "/mapping-get-v2")
public String mappingGetV2() {
  log.info("mapping-get-v2");
  return "ok";
}
```
* `@RequestMapping`과 `method`를 지정한 것을 축약하여 위의 애노테이션을 사용할 수 있다.

### 6.4. PathVariable(경로 변수) 사용
```java
/**
* PathVariable 사용
* 변수명이 같으면 생략 가능
* @PathVariable("userId") String userId -> @PathVariable String userId
*/
@GetMapping("/mapping/{userId}")
public String mappingPath(@PathVariable("userId") String data) {
  log.info("mappingPath userId={}", data);
  return "ok";
}
```
* HTTP API는 리소스 경로에 식별자를 넣는 스타일을 선호한다(/mapping/userA)
* `@RequestMapping`은 URL 경로를 템플릿화 할 수 있는데, `@PathVariable`을 사용하면 매칭되는 부분을 편리하게 조회할 수 있다.
* `@PathVariabvle`의 이름과 파라미터 이름이 같으면 생략할 수 있다.

### 6.5. PathVariable 사용 - 다중
```java
/**
* PathVariable 사용 다중
*/
@GetMapping("/mapping/users/{userId}/orders/{orderId}")
public String mappingPath(@PathVariable String userId, @PathVariable Long orderId) {
  log.info("mappingPath userId={}, orderId={}", userId, orderId);
  return "ok";
}
```
* `@PathVariable` 다중으로도 사용 가능

### 6.6. 특정 파라미터 조건 매핑
```java
/**
* 파라미터로 추가 매핑
* params="mode",
* params="!mode"
* params="mode=debug"
* params="mode!=debug" (! = )
* params = {"mode=debug","data=good"}
*/
@GetMapping(value = "/mapping-param", params = "mode=debug")
public String mappingParam() {
  log.info("mappingParam");
  return "ok";
}
```
* 특정 조건의 파라미터를 매핑 가능

### 6.7. 특정 헤더 조건 매핑
```java
/**
* 특정 헤더로 추가 매핑
* headers="mode",
* headers="!mode"
* headers="mode=debug"
* headers="mode!=debug" (! = )
*/
@GetMapping(value = "/mapping-header", headers = "mode=debug")
public String mappingHeader() {
  log.info("mappingHeader");
  return "ok";
}
```
* 조건을 HTTP 헤더에서도 사용 가능

### 6.7.미디어 타입 조건 매핑 - HTTP 요청 Content-Type, consume
```java
/**
* Content-Type 헤더 기반 추가 매핑 Media Type
* consumes="application/json"
* consumes="!application/json"
* consumes="application/*"
* consumes="*\/*"
* MediaType.APPLICATION_JSON_VALUE
*/
@PostMapping(value = "/mapping-consume", consumes = "application/json")
public String mappingConsumes() {
  log.info("mappingConsumes");
  return "ok";
}
```
* HTTP 요청의 Content-Type 헤더를 기반으로 미디어 타입으로 매핑한다.
* 만약 맞지 않다면 HTTP 415 상태코드(Unsupported Media Type)을 반환한다.

### 6.8. 미디어 타입 조건 매핑 - HTTP 요청 Accept, produce
``` java
/**
* Accept 헤더 기반 Media Type
* produces = "text/html"
* produces = "!text/html"
* produces = "text/*"
* produces = "*\/*"
*/
@PostMapping(value = "/mapping-produce", produces = "text/html")
public String mappingProduces() {
  log.info("mappingProduces");
  return "ok";
}
```
* HTTP 요청의 Accept 헤더를 기반으로 미디어 타입으로 매핑한다.
* 만약 맞지 않다면 HTTP 406 상태 코드(Not Acceptable)을 반환한다.

***
# 7. 요청 매핑 - API 예시
### 7.1. 회원 관리 API
* 회원 목록 조회 : GET  `/users`
* 회원 등록 : POST `/users`
* 회원 조회 : GET `/users/{userId}`
* 회원 수정 : PATCH `/users/{userId}`
* 회원 삭제 : DELETE `/users/{userId}`

### 7.2. MappingClassController
```java
@RestController
@ReqeustMapping("/mapping/users")
public class MappingClassController{
  /**
  * GET /mapping/users
  */
  @GetMapping
  public String users(){
    return "get users"
  }
  /**
  * POST /mapping/users
  */
  @PostMapping
  public String addUser() {
    return "post user";
  }
  /**
  * GET /mapping/users/{userId}
  */
  @GetMapping("/{userId}")
  public String findUser(@PathVariable String userId) {
    return "get userId=" + userId;
  }
  /**
  * PATCH /mapping/users/{userId}
  */
  @PatchMapping("/{userId}")
  public String updateUser(@PathVariable String userId) {
    return "update userId=" + userId;
  }
  /**
  * DELETE /mapping/users/{userId}
  */
  @DeleteMapping("/{userId}")
  public String deleteUser(@PathVariable String userId) {
    return "delete userId=" + userId;
  }
}
```
* 위 API로 만든 Controller

### 7.3. HTTP 요청 - 기본, 헤더 조회
* 애노테이션 기반의 스프링 컨트롤러는 다양한 파라미터를 지원한다.
``` java
@Slf4j
@RestController
public class RequestHeaderController {
  @RequestMapping("/headers")
  public String headers(HttpServletRequest request,
                        HttpServletResponse response,
                        HttpMethod httpMethod,
                        Locale locale,
                        @RequestHeader MultiValueMap<String, String>
                        headerMap,
                        @RequestHeader("host") String host,
                        @CookieValue(value = "myCookie", required = false)
                        String cookie) {
    log.info("request={}", request);
    log.info("response={}", response);
    log.info("httpMethod={}", httpMethod);
    log.info("locale={}", locale);
    log.info("headerMap={}", headerMap);
    log.info("header host={}", host);
    log.info("myCookie={}", cookie);
    return "ok";
  }
}
```
* HttpServletRequest
* HttpServletResponse
* HttpMethod : HTTP 메서드 조회
* Locale : Locale 정보 조회
* @RequestHeader MultiValueMap<String, String> headerMap : 모든 HTTP 헤더를 MutliValueMap 형식으로 조회한다.
* @RequestHeader("host") String host
    + 특정 HTTP 헤더를 조회한다.
    + 속성
        - 필수 값 여부 : required
        - 기본 값 여부 : defaultValue
* @CookieValue(value = "myCookie", required = false) String cookie
    + 특정 쿠키를 조회한다.
    + 속성
        - 필수 값 여부 : required
        - 기본 값 : defaultValue

### 7.4. HTTP 요청 파라미터 - 쿼리 파라미터, HTML Form
HTTP 요청 데이터 조회 
* GET - 쿼리 파라미터
    + /url**?username=hello&age=20**
    + 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달
    + 예) 검색, 필터, 페이징등에서 많이 사용하는 방식
* POST - HTML Form
    + content-type: application/x-www-form-urlencoded
    + 메시지 바디에 쿼리 파리미터 형식으로 전달 username=hello&age=20
    + 예) 회원 가입, 상품 주문, HTML Form 사용
* HTTP messageBody에 데이터 직접 담아서 요청
    + HTTP API에서 주로 사용, JSON, XML, TEXT
    + 데이터 형식은 주로 JSON 사용
    + POST, PUT, PATCH

#### 7.4.1 요청 파라미터V1 - 쿼리 파라미터, HTML Form
```java
@Slf4j
@Controller
public class RequestParamController {
/**
* 반환 타입이 없으면서 이렇게 응답에 값을 직접 집어넣으면, view 조회X
*/
@RequestMapping("/request-param-v1")
  public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
    String username = request.getParameter("username");
    int age = Integer.parseInt(request.getParameter("age"));
    log.info("username={}, age={}", username, age);
    response.getWriter().write("ok");
  }
}
```
* 요청 파라미터 조회 V1
* Get 쿼리 파라미터 전송, Post HTML Form 전송 이든 둘다 형식이 같으므로 구분없이 조회가능하다.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Title</title>
  </head>
  <body>
    <form action="/request-param-v1" method="post">
      username: <input type="text" name="username" />
      age: <input type="text" name="age" />
      <button type="submit">전송</button>
    </form>
  </body>
</html>
```
### 7.4.2. 요청 파라미터V2 - @RequestParam
```java
/**
* @RequestParam 사용
* - 파라미터 이름으로 바인딩
* @ResponseBody 추가
* - View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력
*/
@ResponseBody
@RequestMapping("/request-param-v2")
public String requestParamV2(
                              @RequestParam("username") String memberName,
                              @RequestParam("age") int memberAge) {
  log.info("username={}, age={}", memberName, memberAge);
  return "ok";
}
```
* `@RequestParam` : 파라미터 이름으로 바인딩
* `@ResponseBody` : View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력
* **@RequestParam**의 `name(value)` 속성이 파라미터 이름으로 사용
  + @RequestParam("**username**") String **memberName
  + request.getParameter("**username**")
  + 두 개는 비슷한 코드

### 7.4.3. 요청 파라미터 V3 - @RequestParam
```java
/**
* @RequestParam 사용
* HTTP 파라미터 이름이 변수 이름과 같으면 @RequestParam(name="xx") 생략 가능
*/
@ResponseBody
@RequestMapping("/request-param-v3")
public String requestParamV3(
                              @RequestParam String username,
                              @RequestParam int age) {
  log.info("username={}, age={}", username, age);
  return "ok";
}
```
* HTTP 파라미터 이름이 변수 이름과 같으면 `@RequestParam(name="xx")` 생략 가능

### 7.4.4. 요청 파라미터 V4 - @RequestParam
```java
/**
* @RequestParam 사용
* String, int 등의 단순 타입이면 @RequestParam 도 생략 가능
*/
@ResponseBody
@RequestMapping("/request-param-v4")
public String requestParamV4(String username, int age) {
  log.info("username={}, age={}", username, age);
  return "ok";
}
```
* 매개변수 타입이 String, int와 같은 단순 타입일시 `@RequestParam`도 생략 가능
> 하지만 매개변수 까지 생략하는 것은 명확하지 않기 때문에 넣어주는 것이 좋다고 생각된다.<br/>

> Spring boot 3.0 이상 부터는 매개변수 이름이 생략되면 에러가 발생하기 때문에 두가지 해결방법이있다.<br/>
> 컴파일 시정에 -parameters 옵션 적용하거나 Gradle로 빌드한다.<br/>
> `@RequestParam("username") String username`,`@PathVariable("userId") String userId` 과같이 이름을 항상 적어준다 (권장)<br/>

### 7.4.5. 파라미터 필수 여부 - requestParamRequired
```java
/**
* @RequestParam.required
* /request-param-required -> username이 없으므로 예외
*
* 주의!
* /request-param-required?username= -> 빈문자로 통과
*
* 주의!
* /request-param-required
* int age -> null을 int에 입력하는 것은 불가능, 따라서 Integer 변경해야 함(또는 다음에 나오는
defaultValue 사용)
*/
@ResponseBody
@RequestMapping("/request-param-required")
public String requestParamRequired(
          @RequestParam(required = true) String username,
          @RequestParam(required = false) Integer age) {
  log.info("username={}, age={}", username, age);
  return "ok";
}
```
* `@RequestParam.required`
  + 파라미터 필수 여부
  + 기본값이 파라미터 필수(true)이다.
* **주의! - 파라미터 이름만 사용**
  + `/request-param-required?username=` 파라미터 이름만 있고 값이 없는 경우 빈문자로 통과
* 주의! - 기본형(primitive)에 null 입력
  + `@RequestParam(required = false) int age` : int에 null 입력 불가능 , 따라서 Integer로 변경하거나 `defaultValue` 사용


### 7.4.6. 파라미터 필수 기본 값 적용 - requestParamDefault
```java
/**
* @RequestParam
* - defaultValue 사용
*
* 참고: defaultValue는 빈 문자의 경우에도 적용
* /request-param-default?username=
*/
@ResponseBody
@RequestMapping("/request-param-default")
public String requestParamDefault(
        @RequestParam(required = true, defaultValue = "guest") String username,
        @RequestParam(required = false, defaultValue = "-1") int age) {
  log.info("username={}, age={}", username, age);
  return "ok";
}
```
* 파라미터에 값이 없는 경우 `defaultValue` 를 사용하면 기본 값을 적용할 수 있다.
* 이미 기본 값이 있기 때문에 `required` 는 의미가 없다.
* `defaultValue` 는 빈 문자의 경우에도 설정한 기본 값이 적용된다.
* `/request-param-default?username=`

### 7.4.7. 파라미터를 Map으로 조회하기 - requestParamMap
```java
/**
* @RequestParam Map, MultiValueMap
* Map(key=value)
* MultiValueMap(key=[value1, value2, ...]) ex) (key=userIds, value=[id1, id2])
*/
@ResponseBody
@RequestMapping("/request-param-map")
public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
  log.info("username={}, age={}", paramMap.get("username"),vparamMap.get("age"));
  return "ok";
}
```
* 파라미터의 값이 1개가 확실하다면 `Map` 을 사용해도 되지만, 그렇지 않다면 `MultiValueMap` 을 사용하자.

***
# 8. HTTP 요청 파라미터 - @ModelAttribute
```java
@RequestParam String username;
@RequestParam int age;
HelloData data = new HelloData();
data.setUsername(username);
data.setAge(age);
```
* 보통 요청 파라미터를 받아서 필요한 객체를 만들고 그 객체에 값을 넣어준다.
* 이 과정을 자동화 한 것이 @ModelAttribute이다.

### 8.1. @ModelAttribute 적용 - modelAttributeV1

```java
@Data
public class HelloData {
  private String username;
  private int age;
}
```
* 바인딩 받을 객체
```java
/**
* @ModelAttribute 사용
* 참고: model.addAttribute(helloData) 코드도 함께 자동 적용됨, 뒤에 model을 설명할 때 자세히
설명
*/
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@ModelAttribute HelloData helloData) {
  log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
  return "ok";
}
```
* HelloData 객체가 생성되고, 요청 파라미터의 값도 모두 들어가있다.
* @ModelAttribute 동작 방식
  1. `HelloData`객체를 생성한다.
  2. 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티를 찾는다.
  3. 해당 프로퍼티의 setter를 호출해서 파라미터의 값을 입력(바인딩)한다.
  + 예) 파라미터 이름이 `username`이면 `setUsername()` 메서드를 찾아서 호출하면서 값을 입력한다.

### 8.2. @ModelAttribute 생략 - modelAttributeV2
```java
/**
* @ModelAttribute 생략 가능
* String, int 같은 단순 타입 = @RequestParam
* argument resolver 로 지정해둔 타입 외 = @ModelAttribute
*/
@ResponseBody
@RequestMapping("/model-attribute-v2")
public String modelAttributeV2(HelloData helloData) {
  log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
  return "ok";
}
```
* `@ModelAttribute` 는 생략할 수 있다.
  + `@RequestParam` 도 생략할 수 있으니 혼란이 발생할 수 있다.
* 스프링은 해당 생략시 다음과 같은 규칙을 적용한다.
  + `String` , `int` , `Integer` 같은 단순 타입 = `@RequestParam`
  + 나머지 = `@ModelAttribute` (argument resolver 로 지정해둔 타입 외)

****
# 9. HTTP 요청 메시지 - 단순 텍스트
* HTTP message body**에 데이터를 직접 담아서 요청
  + HTTP API에서 주로 사용, JSON, XML, TEXT
  + 데이터 형식은 주로 JSON 사용
  + POST, PUT, PATCH

* 요청 파라미터와 다르게, HTTP 메시지 바디를 통해 데이터가 직접 넘어오는 경우는 `@RequestParam`, `@ModelAttribute`를 사용할 수 없다

### 9.1. requestBodyStringV1
```java
@Slf4j
@Controller
public class RequestBodyStringController {
  @PostMapping("/request-body-string-v1")
  public void requestBodyString(HttpServletRequest request, HttpServletResponse response) throws IOException {
    ServletInputStream inputStream = request.getInputStream();
    String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

    log.info("messageBody={}", messageBody);
    response.getWriter().write("ok");
  }
}
```
* HTTP 메시지 바디의 데이터를 `InputStream`을 사용해서 직접 읽을 수 있다.

### 9.2. Input,Output 스트림, Reader - requestBodyStringV2
```java
/**
* InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회
* OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력
*/
@PostMapping("/request-body-string-v2")
public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {
  String messageBody = StreamUtils.copyToString(inputStream,
  StandardCharsets.UTF_8);
  log.info("messageBody={}", messageBody);
  responseWriter.write("ok");
}
```
* InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회
* OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력

### 9.3. HttpEntity - requestBodyStringV3
```java
/**
* HttpEntity: HTTP header, body 정보를 편리하게 조회
* - 메시지 바디 정보를 직접 조회(@RequestParam X, @ModelAttribute X)
* - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
*
* 응답에서도 HttpEntity 사용 가능
* - 메시지 바디 정보 직접 반환(view 조회X)
* - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
*/
@PostMapping("/request-body-string-v3")
public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) {
  String messageBody = httpEntity.getBody();
  log.info("messageBody={}", messageBody);
  return new HttpEntity<>("ok");
}
```
* **HttpEntity**: HTTP header, body 정보를 편리하게 조회
  + 메시지 바디 정보를 직접 조회
  + 요청 파라미터를 조회하는 기능과 관계 없음 `@RequestParam` X, `@ModelAttribute` X
* **HttpEntity는 응답에도 사용 가능**
  + 메시지 바디 정보 직접 반환
  + 헤더 정보 포함 가능
  + view 조회X

* `HttpEntity` 를 상속받은 다음 객체들도 같은 기능을 제공한다.
  + **RequestEntity** : HttpMethod, url 정보가 추가, 요청에서 사용
  + **ResponseEntity** : HTTP 상태 코드 설정 가능, 응답에서 사용
> 예) `return new ResponseEntity<String>("Hello World", responseHeaders, HttpStatus.CREATED)`

### 9.4. @RequestBody - requestBodyStringV4
```java
/**
* @RequestBody
* - 메시지 바디 정보를 직접 조회(@RequestParam X, @ModelAttribute X)
* - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
*
* @ResponseBody
* - 메시지 바디 정보 직접 반환(view 조회X)
* - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
*/
@ResponseBody
@PostMapping("/request-body-string-v4")
public String requestBodyStringV4(@RequestBody String messageBody) {
  log.info("messageBody={}", messageBody);
  return "ok";
}
```
* `@RequestBody`를 사용하면 HTTP 메시지 바디 정보를 편리하게 조회할 수 있다.
* 헤더 정보는 `HttpEntity`를 사용하거나 `@RequestHeader`를 사용한다.

***
# 10. HTTP 요청 메시지 - JSON
### 10.1. RequestBodyJsonController
```java
/**
* {"username":"hello", "age":20}
* content-type: application/json
*/
@Slf4j
@Controller
public class RequestBodyJsonController {
  private ObjectMapper objectMapper = new ObjectMapper();
  @PostMapping("/request-body-json-v1")
  public void requestBodyJsonV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
    ServletInputStream inputStream = request.getInputStream();
    String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

    log.info("messageBody={}", messageBody);
    HelloData data = objectMapper.readValue(messageBody, HelloData.class);
    log.info("username={}, age={}", data.getUsername(), data.getAge());

    response.getWriter().write("ok");
  }
}
```
* HttpServletRequest를 사용해서 직접 HTTP 메시지 바디에서 데이터를 읽어와서 문자로 변환한다.
* 문자로 된 JSON 데이터를 Jackson 라이브러리인 objectMapper를 사용해 자바 객체로 변환한다.

### 10.2. requestBodyJsonV2 = @RequestBody 문자 변환
```java
/**
* @RequestBody
* HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
*
* @ResponseBody
* - 모든 메서드에 @ResponseBody 적용
* - 메시지 바디 정보 직접 반환(view 조회X)
* - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
*/
@ResponseBody
@PostMapping("/request-body-json-v2")
public String requestBodyJsonV2(@RequestBody String messageBody) throws IOException {
  HelloData data = objectMapper.readValue(messageBody, HelloData.class);
  log.info("username={}, age={}", data.getUsername(), data.getAge());
  return "ok";
}
```

### 10.3. requestBodyJsonV3 - @RequestBody객체 변환
```java
/**
* @RequestBody 생략 불가능(@ModelAttribute 가 적용되어 버림)
* HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter (content-type:application/json)
*
*/
@ResponseBody
@PostMapping("/request-body-json-v3")
public String requestBodyJsonV3(@RequestBody HelloData data) {
  log.info("username={}, age={}", data.getUsername(), data.getAge());
  return "ok";
}
```
* `@RequestBody` 객체 파라미터
  + `@RequestBody Hello data`
  + `@RequestBody`에 직접 만든 객체를 지정할 수 있다.
* `HttpEntity` , `@RequestBody` 를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다.
  + HTTP 컨버터는 문자 뿐만아니라 JSON도 객체로 변환해준다.
* **@RequestBody는 생략 불가능**

> 스프링은 `@ModelAttribute` , `@RequestParam` 과 같은 해당 애노테이션을 생략시 다음과 같은 규칙을 적용한다. <br/>
> `String` , `int` , `Integer` 같은 단순 타입 = `@RequestParam` <br/>
> 나머지 = `@ModelAttribute` (argument resolver 로 지정해둔 타입 외) <br/>

### 10.4. requestBodyJsonV4 - HttpEntity
```java
@ResponseBody
@PostMapping("/request-body-json-v4")
public String requestBodyJsonV4(HttpEntity<HelloData> httpEntity) {
  HelloData data = httpEntity.getBody();
  log.info("username={}, age={}", data.getUsername(), data.getAge());
  return "ok";
}
```
* HttpEntity를 사용해도 된다.

### 10.5. requestBodyJsonV5
```java
/**
* @RequestBody 생략 불가능(@ModelAttribute 가 적용되어 버림)
* HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter (content-type:
application/json)
*
* @ResponseBody 적용
* - 메시지 바디 정보 직접 반환(view 조회X)
* - HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter 적용(Accept:
application/json)
*/
@ResponseBody
@PostMapping("/request-body-json-v5")
public HelloData requestBodyJsonV5(@RequestBody HelloData data) {
  log.info("username={}, age={}", data.getUsername(), data.getAge());
  return data;
}
```
* `@ResponseBody` 응답의 경우에도 `@ResponseBody` 를 사용하면 해당 객체를 HTTP 메시지 바디에 직접 넣어줄 수 있다.
* 이 경우에도 `HttpEntity` 를 사용해도 된다.
* `@RequestBody` 요청 : JSON 요청 -> HTTP 메시지 컨버터 -> 객체
* `@ResponseBody` 응답 : 객체 -> HTTP 메시지 컨버터 -> JSON 응답

***
# 11. HTTP 응답 - 정적 리소스, 뷰 템플릿
* 스프링서버에서 응답 데이터를 만드는 방법 3가지
    + 정적 리소스 : 예) 웹 브라우저에 정적인 HTML, css, js를 제공할 때는, **정적 리소스**를 사용한다.
    + 뷰 템플릿 사용 : 예) 웹 브라우저에 동적인 HTML을 제공할 때는 뷰 템플릿을 사용한다.
    + HTTP 메시지 사용 : 예) HTTP API를 제공하는 경우에는 HTML이 아니라 데이터를 전달해야 하므로, HTTP 메시지 바디에 JSON 같은 형식으로 데이터를 실어 보낸다.

### 11.1. 정적 리소스
* 스프링 부트 지원 정적 리소스 디렉토리 : `/static` , `/public` , `/resources` , `/META-INF/resources`
* **정적 리소스 경로** `src/main/resources/static` 다음 경로에 파일이 들어있으면 `src/main/resources/static/basic/hello-form.html` 웹 브라우저에서 다음과 같이 실행하면 된다. `http://localhost:8080/basic/hello-form.html`
* 정적 리소스는 해당 파일을 변경 없이 그대로 서비스

### 11.2. 뷰 템플릿
* 뷰 템플릿을 거쳐서 HTML이 생성되고, 뷰가 응답을 만들어서 전달한다.
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
</head>
<body>
<p th:text="${data}">empty</p>
</body>
</html>
```
* 뷰 템플릿

```java
@Controller
public class ResponseViewController {
  @RequestMapping("/response-view-v1")
  public ModelAndView responseViewV1() {
    ModelAndView mav = new ModelAndView("response/hello").addObject("data", "hello!");
    return mav;
  }
  @RequestMapping("/response-view-v2")
  public String responseViewV2(Model model) {
    model.addAttribute("data", "hello!!");
    return "response/hello";
  }
  @RequestMapping("/response/hello")
  public void responseViewV3(Model model) {
    model.addAttribute("data", "hello!!");
  }
}
```
* String을 반환하는 경우 - View or HTTP 메시지
  + `@ResponseBody` 가 없으면 `response/hello` 로 뷰 리졸버가 실행되어서 뷰를 찾고, 렌더링 한다.
  + `@ResponseBody` 가 있으면 뷰 리졸버를 실행하지 않고, HTTP 메시지 바디에 직접 `response/hello` 라는 문자가 입력된다.
* Void를 반환하는 경우
  + `@Controller` 를 사용하고, `HttpServletResponse` , `OutputStream(Writer)` 같은 HTTP 메시지 바디를 처리하는 파라미터가 없으면 요청 URL을 참고해서 논리 뷰 이름으로 사용
  + 예) 요청 URL: `/response/hello` , 실행: `templates/response/hello.html
  + 명시성이 떨어지고 딱 맞는 경우가 없기 때문에 권장하지 않음
* `@ResponseBody` , `HttpEntity` 를 사용하면, 뷰 템플릿을 사용하는 것이 아니라, HTTP 메시지 바디에 직접 응답 데이터를 출력할 수 있다.

***
# 12. HTTP 응답 - HTTP API, 메시지 바디에 직접 입력
```java
@Slf4j
@Controller
//@RestController
public class ResponseBodyController {
  @GetMapping("/response-body-string-v1")
  public void responseBodyV1(HttpServletResponse response) throws IOException{
    response.getWriter().write("ok");
  }
  /**
  * HttpEntity, ResponseEntity(Http Status 추가)
  * @return
  */
  @GetMapping("/response-body-string-v2")
  public ResponseEntity<String> responseBodyV2() {
    return new ResponseEntity<>("ok", HttpStatus.OK);
  }
  @ResponseBody
  @GetMapping("/response-body-string-v3")
    public String responseBodyV3() {
    return "ok";
  }
  @GetMapping("/response-body-json-v1")
  public ResponseEntity<HelloData> responseBodyJsonV1() {
    HelloData helloData = new HelloData();
    helloData.setUsername("userA");
    helloData.setAge(20);
    return new ResponseEntity<>(helloData, HttpStatus.OK);
  }
  @ResponseStatus(HttpStatus.OK)
  @ResponseBody
  @GetMapping("/response-body-json-v2")
  public HelloData responseBodyJsonV2() {
    HelloData helloData = new HelloData();
    helloData.setUsername("userA");
    helloData.setAge(20);
    return helloData;
  }
}
```
* **responseBodyV1**
  + 서블릿을 직접 다룰 때 처럼 HttpServletResponse 객체를 통해서 HTTP 메시지 바디에 직접 `ok` 응답 메시지를 전달한다.
  + `response.getWriter().write("ok")`
* **responseBodyV2**
  + `ResponseEntity` 엔티티는 `HttpEntity` 를 상속 받았는데, HttpEntity는 HTTP 메시지의 헤더, 바디 정보를 가지고 있다.
  + `ResponseEntity` 는 여기에 더해서 HTTP 응답 코드를 설정할 수 있다.
  + `HttpStatus.CREATED` 로 변경하면 201 응답이 나가는 것을 확인할 수 있다.
* **responseBodyV3**
  + `@ResponseBody` 를 사용하면 view를 사용하지 않고, HTTP 메시지 컨버터를 통해서 HTTP 메시지를 직접 입력할수 있다.
  + `ResponseEntity` 도 동일한 방식으로 동작한다.
* **responseBodyJsonV1**
  + `ResponseEntity` 를 반환한다.
  +  HTTP 메시지 컨버터를 통해서 JSON 형식으로 변환되어서 반환된다.
* **responseBodyJsonV2**
  + `ResponseEntity` 는 HTTP 응답 코드를 설정할 수 있는데, `@ResponseBody` 를 사용하면 이런 것을 설정하기 까다롭다.
  + `@ResponseStatus(HttpStatus.OK)` 애노테이션을 사용하면 응답 코드도 설정할 수 있다.
  + 물론 애노테이션이기 때문에 응답 코드를 동적으로 변경할 수는 없다. 프로그램 조건에 따라서 동적으로 변경하려면 `ResponseEntity` 를 사용하면 된다.
* **@RestController**
  + `@Controller` 대신에 `@RestController` 애노테이션을 사용하면, 해당 컨트롤러에 모두 `@ResponseBody` 가 적용되는 효과가 있다.
  + 따라서 뷰 템플릿을 사용하는 것이 아니라, HTTP 메시지 바디에 직접 데이터를 입력한다.
  + `@ResponseBody` 는 클래스 레벨에 두면 전체 메서드에 적용되는데, `@RestController` 에노테이션 안에 `@ResponseBody` 가 적용되어 있다.
 
***
# 13. 메시지 컨버터
* 뷰 템플릿으로 HTML을 생성해서 응답하는 것이 아니라, HTTP API처럼 JSON 데이터를 HTTP 메시지 바디에서 직접 읽거나 쓰는 경우 HTTP 메시지 컨버터를 사용하면 편리하다.
![image](https://github.com/helloJosh/spring-servlet-study/assets/37134368/b2e13065-5235-469c-8ccf-6578c3e60349)
* @ResponseBody 사용 원리
  + HTTP의 BODY에 문자 내용을 직접 반환
  + `viewResolver` 대신에 `HttpMessageConverter` 가 동작
  + 기본 문자처리: `StringHttpMessageConverter`
  + 기본 객체처리: `MappingJackson2HttpMessageConverter`
  + byte 처리 등등 기타 여러 HttpMessageConverter가 기본으로 등록되어 있음

* **스프링 MVC는 다음의 경우에 HTTP 메시지 컨버터를 적용한다.**
  + HTTP 요청: `@RequestBody` , `HttpEntity(RequestEntity)`
  + HTTP 응답: `@ResponseBody` , `HttpEntity(ResponseEntity)`

### 13.1. **스프링 부트 기본 메시지 컨버터**

```
// (일부 생략)
0 = ByteArrayHttpMessageConverter
1 = StringHttpMessageConverter
2 = MappingJackson2HttpMessageConverter
```
* 스프링 부트는 다양한 메시지 컨버터를 제공하는데, 대상 클래스 타입과 미디어 타입 둘을 체크해서 사용여부를 결정한다.
* 만약 만족하지 않으면 다음 메시지 컨버터로 우선순위가 넘어간다.

### 13.2. 기본 컨버터 3가지
* `ByteArrayHttpMessageConverter` : `byte[]` 데이터를 처리한다.
  + 클래스 타입: `byte[]` , 미디어타입: `*/*` ,
  + 요청 예) `@RequestBody byte[] data`
  + 응답 예) `@ResponseBody return byte[]` 쓰기 미디어타입 `application/octet-stream`
* `StringHttpMessageConverter` : `String` 문자로 데이터를 처리한다.
  + 클래스 타입: `String` , 미디어타입: `*/*`
  + 요청 예) `@RequestBody String data`
  + 응답 예) `@ResponseBody return "ok"` 쓰기 미디어타입 `text/plain`
* `MappingJackson2HttpMessageConverter` : application/json
  + 클래스 타입: 객체 또는 `HashMap` , 미디어타입 `application/json` 관련
  + 요청 예) `@RequestBody HelloData data`
  + 응답 예) `@ResponseBody return helloData` 쓰기 미디어타입 `application/json` 관련
 
### 13.3. HTTP 요청 데이터 읽기
* HTTP 요청이 오고, 컨트롤러에서 `@RequestBody`, `HttpEntity` 파라미터를 사용한다.
* 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 `canRead()`를 호출한다.
  + 대상 클래스 타입을 지원하는지( 예 : `@RequestBody`의 대상 클래스(`byte[]`, `String`, `HelloData`)
  +HTTP 요청의 Content-Type 미디어 타입을 지원하는 (예 : `text/plain` , `application/json` , `*/*`)
* `canRead()` 조건을 만족하면 `read()` 를 호출해서 객체 생성하고, 반환한다.

### 13.4. HTTP 응답 데이터 생성
* 컨트롤러에서 `@ResponseBody` , `HttpEntity` 로 값이 반환된다.
* 메시지 컨버터가 메시지를 쓸 수 있는지  확인하기 위해 `canWrite()`를 호출한다.
  + 대상 클래스 타입을 지원하는지 확인(return의 대상 클래스 : `byte[]` , `String` , `HelloData`)
  + HTTP 요청의 Accept 미디어 타입을 지원하는가.(정확히는 `@RequestMapping`의 `produces`, 예 : `text/plain` , `application/json` , `*/*`)
* `canWrite()` 조건을 만족하면 `write()`를 호출해서 HTTP 응답 메시지 바디에 데이터를 생성한다.

### 13.3. 요청 매핑 핸들러 어댑터 구조
![image](https://github.com/helloJosh/spring-servlet-study/assets/37134368/2571d877-b60f-47ef-af1a-e562bf36075b)
* HTTP 메시지 컨버터는 `@RequestMapping` 을 처리하는 핸들러 어댑터인 `RequestMappingHandlerAdapter` (요청 매핑 헨들러 어뎁터)에 있다.

#### 13.3.1. RequestMappingHandlerAdapter 동작방식
![image](https://github.com/helloJosh/spring-servlet-study/assets/37134368/93753894-9103-43c5-819e-efa84ea89fca)
##### 13.3.1.1 **ArgumentResolver**
* 애노테이션 기반 컨트롤러를 처리하는 `RequestMappingHandlerAdapter` 는 바로 이 `ArgumentResolver` 를 호출해서 컨트롤러(핸들러)가 필요로 하는 다양한 파라미터의 값(객체)을 생성한다.
* 파리미터의 값이 모두 준비되면 컨트롤러를 호출하면서 값을 넘겨준다.
* 스프링은 30개가 넘는 `ArgumentResolver` 를 기본으로 제공한다.

##### 13.3.1.2 **ReturnValueHandler**
* ArgumentResolver` 와 비슷한데, 이것은 응답 값을 변환하고 처리한다.
* 컨트롤러에서 String으로 뷰 이름을 반환해도, 동작하는 이유가 바로 ReturnValueHandler 덕분이다.

#### 13.3.2. HTTP 메시지 컨버터
![image](https://github.com/helloJosh/spring-servlet-study/assets/37134368/4338d22c-a825-45a0-9a11-58a7b50c345b)
* HTTP 메시지 컨버터를 사용하는 `@RequestBody` 도 컨트롤러가 필요로 하는 파라미터의 값에 사용된다. `@ResponseBody` 의 경우도 컨트롤러의 반환 값을 이용한다.

##### 13.3.2.1 요청
* `@RequestBody` 를 처리하는 `ArgumentResolver` 가 있다.
* `HttpEntity` 를 처리하는 `ArgumentResolver` 가 있다.
* `ArgumentResolver` 들이 HTTP 메시지 컨버터를 사용해서 필요한 객체를 생성한다

##### 13.3.2.2 응답
* `@ResponseBody` 와 `HttpEntity` 를 처리하는 `ReturnValueHandler` 가 있다.
* `@RequestBody` `@ResponseBody` 가 있으면 `RequestResponseBodyMethodProcessor()`를 사용한다.
* HttpEntity` 가 있으면 `HttpEntityMethodProcessor()`를 사용한다.
 
