---
title: "Spring HTTP 통신 흐름"
excerpt: "HTTP Request에서 HTTP Response의 과정을 Spring MVC 기준으로 알아보자"

categories:
  - 자바&코틀린하린

permalink: /자바&코틀린하린/SpringMVC-HTTP/

toc: true
toc_sticky: true

date: 2024-01-14
last_modified_at: 2024-01-14
---

### 스프링 HTTP 통신 흐름

스프링 프레임워크를 사용하여 naver.com으로의 HTTP 통신 과정을 간단히 설명해보겠습니다. 이 예시에서는 스프링 MVC를 기반으로 합니다.

#### URL 입력
웹 브라우저에 `www.naver.com` 을 입력합니다.

#### DNS
HTTP는 기본적으로 TCP/IP 통신을 하기 때문에 `www.naver.com`에 대한 IP를 알아내야합니다. 이때 `www.naver.com`를 IP로 변경해줄 수 있는 DNS가 필요합니다. 

1. 웹 브라우저에 `www.naver.com`를 입력하면,
2. 컴퓨터의 hosts 파일에 `naver.com` 의 IP 주소 정보가 있는지 확인합니다. (mac os 기준 hosts 파일의 위치는 /etc/hosts.)
3. hosts 파일에 정보가 없으면, 컴퓨터에 있는 local DNS cache에 IP 주소가 있는지 확인합니다.
4. hosts 파일에서도, DNS cache 에서도 IP 주소를 알아낼 수 없다면, DNS 서버에 물어봐야 합니다. 이때는 recusive 하게 여러 네임 서버를 가쳐 IP 주소를 찾아냅니다.
![enter image description here](https://github.com/rineeee/rineeee.github.io/blob/main/assets/images/%E1%84%90%E1%85%A9%E1%86%BC%E1%84%89%E1%85%B5%E1%86%AB%E1%84%80%E1%85%AA%E1%84%8C%E1%85%A5%E1%86%BC1.png?raw=true)

#### HTTP 요청
DNS 서버를 통해 알아낸 IP주소와 입력한 URL 정보가 함께 요청으로 전달됩니다. 이는 HTTP Protocol를 통해 HTTP Request Message가 생성됩니다. 
- HTTP Request Message
![enter image description here](https://github.com/rineeee/rineeee.github.io/blob/main/assets/images/%E1%84%90%E1%85%A9%E1%86%BC%E1%84%89%E1%85%B5%E1%86%AB%E1%84%80%E1%85%AA%E1%84%8C%E1%85%A5%E1%86%BC2.png?raw=true)

이후, TCP Protocol을 사용하여 인터넷을 거쳐 해당 IP 주소의 컴퓨터 Web Server로 전송합니다. 

#### Web Server
목적지에 들어가기 전 목적 IP 컴퓨터의 Web Server 에 도착했습니다. Web Server 는 요청 URL 정보를 확인하고, 필요한 요청이 여기서 처리되면 돌려보내고,  여기서 처리할 수 없는 요청이라면 WAS 로 이동합니다.

#### WAS(Spring MVC)
![enter image description here](https://github.com/rineeee/rineeee.github.io/blob/main/assets/images/%E1%84%90%E1%85%A9%E1%86%BC%E1%84%89%E1%85%B5%E1%86%AB%E1%84%80%E1%85%AA%E1%84%8C%E1%85%A5%E1%86%BC3.png?raw=true)
**HandlerMapping**

-   HandlerMapping 을 통해 요청 URL에 매핑되는 `Handler 조회` 합니다.  
    -   @RequestMapping 기반 Controller 일 경우 `RequestMappingHandlerMapping`
    -   Spring Bean 기반 Controller 일 경우 `BeanNameUrlHandlerMapping`
    
**HandlerAdapter**
-   조회한 Handler 를 실행할 수 있는 `Handler Adapter 조회` 합니다.
    -   @RequestMapping 기반 Controller 일 경우 `RequestMappingHandlerAdapter`
    -   HttpRequestHandler 처리의 경우 `HttpRequestHandlerAdapter`
    -   Spring Bean 기반 Controller 일 경우 `SimpleControllerHandlerAdapter`
-   앞서 조회한 `Handler Adapter 를 실행`하면 Handler Adapter 가 실제 `Handler(Controller) 실행` 됩니다.
-   Handler Adapter 는 Handler(Controller) 가 반환하는 정보를 `ModelAndView로 변환해서 반환` 합니다.

**ViewResolver**
-   viewResolver 를 찾고 실행합니다.
    -   Bean 이름으로 등록된 View 의 경우 `BeanNameViewResolver`
    -   Bean 이름으로 등록된 View 가 없을 경우 `InternalResourceViewResolver`
    
**View**
-   viewResolver는 View의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는`View 객체 반환` 합니다.
    -   `BeanNameViewResolver` : `BeanNameView` 반환
    -   `InternalResourceViewResolver` : `InternalResourceView` 반환
    
**View Rendering**
-   View 객체를 통해 View Rendering합니다.
    -   view.render()

#### 정리
정리하자면, 스프링의 HTTP 통신은 사용자가 웹 브라우저에 주소를 입력하면, DNS를 통해 서버의 위치를 찾아가며, 그 후에 서버로 HTTP 요청이 이루어집니다. 스프링 MVC는 이 요청을 받아 적절한 컨트롤러를 찾아 비즈니스 로직을 처리하고, 최종적으로 결과를 뷰로 보내 웹 페이지를 완성합니다. 자세한 통신 과정에 대한 이해를 위해선 NAT, CDN, GSLB 등의 개념을 찾아보시는 것이 도움이 될 것입니다.