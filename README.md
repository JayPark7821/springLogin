* ### 서블릿 필터
  * 필터 흐름
  > HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러
  
  참고로 스프링을 사용하는 경우 여기서 말하는 서블릿은 스프링의 디스패처 서블릿으로 생각하면 된다.

  * 필터 제한
  > HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러 // 로그인 사용자
  > HTTP 요청 -> WAS -> 필터 ( 적절하지 않은 요청이라 판단, 서블릿 호출 X) // 비 로그인 사용자
  

  ```java
  
  /*
   * Licensed to the Apache Software Foundation (ASF) under one or more
   * contributor license agreements.  See the NOTICE file distributed with
   * this work for additional information regarding copyright ownership.
   * The ASF licenses this file to You under the Apache License, Version 2.0
   * (the "License"); you may not use this file except in compliance with
   * the License.  You may obtain a copy of the License at
   *
   *     http://www.apache.org/licenses/LICENSE-2.0
   *
   * Unless required by applicable law or agreed to in writing, software
   * distributed under the License is distributed on an "AS IS" BASIS,
   * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   * See the License for the specific language governing permissions and
   * limitations under the License.
   */
  package javax.servlet;
  
  import java.io.IOException;
  
  public interface Filter {
      public default void init(FilterConfig filterConfig) throws ServletException {}
  
      public void doFilter(ServletRequest request, ServletResponse response,
              FilterChain chain) throws IOException, ServletException;
      
      public default void destroy() {}
  }
  
  ```

* ### 필터 인터페이스를 구현하고 등록하면 서블릿 컨테이너가 필터를 싱글톤 객체로 생성, 관리
* `init()` 필터 초기화 메서드, 서블릿 컨테이너가 생성될 때 호출된다.
* `doFilter()` 고객의 요청이 올 때 마다 해당 메서드가 호출됨. (필터 로직 구현)
* `destroy()` 필터 종료 메서드, 서블릿 컨테이너가 종료될 때 호출된다.

```java
  chain.doFilter(request, response);
// 다음 필터가 있으면 필터를 호출, 필터가 없으면 서블릿 호출 
// 만약 호출하지 않으면 다음 진행이 안된다.
 ```

>참고 HTTP 요청시 같은 요청의 로그에 모두 같은 식별자를 자동으로 남기는 방법은 logback mdc검색
>

* ### 스프링 인터셉터
  * 스프링 인터셉터 흐름
  > HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러

  * 스프링 인터셉터는 디스패치 서블릿과 컨트롤러 사이에서 컨트롤러 호출 직전에 호출된다.
  * 스프링 인터셉터는 스프링 MVC가 제공하는 기능이다. 결국 디스패처 서블릿 이후에 호출됨
    * (스프링 MVC의 시작점이 디스패처 서블릿 이라고 생각)
  * 스프링 인터셉터에도  URL패턴을 적용할 수 있는데, 서블릿 URL패턴과는 다르고 매우 정밀하게 설정가능 



  * 스프링 인터셉터 제한
  > HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 ->  컨트롤러 // 로그인 사용자
  > HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 ( 적절하지 않은 요청이라 판단, 컨트롤러 호출 X) // 비 로그인 사용자

![image](https://user-images.githubusercontent.com/60100532/177733328-190ee015-e1b9-41d3-80ae-1570e4284836.png)


* 정상흐름
  * `preHandle` : 컨트롤러 호출 전에 호출된다. (더 정확히는 핸들러 어댑터 호출 전에 호출된다.) 
    * `preHandle` 의 응답값이 `true` 이면 다음으로 진행하고, `false` 이면 더는 진행하지 않는다. `false`
    인 경우 나머지 인터셉터는 물론이고, 핸들러 어댑터도 호출되지 않는다. 그림에서 1번에서 끝이
    나버린다.
    
  * `postHandle` : 컨트롤러 호출 후에 호출된다. (더 정확히는 핸들러 어댑터 호출 후에 호출된다.)
  * `afterCompletion` : 뷰가 렌더링 된 이후에 호출된다


![image](https://user-images.githubusercontent.com/60100532/177734291-1a5c114d-26b1-439b-bc22-c04825b711b9.png)

* 예외 발생시
  * `preHandle` : 컨트롤러 호출 전에 호출된다.
  * `postHandle` : 컨트롤러에서 예외가 발생하면 `postHandle` 은 호출되지 않는다.
  * `afterCompletion` : `afterCompletion` 은 항상 호출된다. 이 경우 예외( ex )를 파라미터로 받아서 어떤
  예외가 발생했는지 로그로 출력할 수 있다.


* ### `afterCompletion`은 예외가 발생해도 호출된다.
  * 예외가 발생하면 `postHandle()` 는 호출되지 않으므로 예외와 무관하게 공통 처리를 하려면
  `afterCompletion()` 을 사용해야 한다.
  예외가 발생하면 `afterCompletion()` 에 예외 정보`( ex )`를 포함해서 호출된다.



```text
? 한 문자 일치
* 경로(/) 안에서 0개 이상의 문자 일치
** 경로 끝까지 0개 이상의 경로(/) 일치
{spring} 경로(/)와 일치하고 spring이라는 변수로 캡처
{spring:[a-z]+} matches the regexp [a-z]+ as a path variable named "spring"
{spring:[a-z]+} regexp [a-z]+ 와 일치하고, "spring" 경로 변수로 캡처
{*spring} 경로가 끝날 때 까지 0개 이상의 경로(/)와 일치하고 spring이라는 변수로 캡처
/pages/t?st.html — matches /pages/test.html, /pages/tXst.html but not /pages/
toast.html
/resources/*.png — matches all .png files in the resources directory
/resources/** — matches all files underneath the /resources/ path, including /
resources/image.png and /resources/css/spring.css
/resources/{*path} — matches all files underneath the /resources/ path and 
captures their relative path in a variable named "path"; /resources/image.png 
will match with "path" → "/image.png", and /resources/css/spring.css will match 
with "path" → "/css/spring.css"
/resources/{filename:\\w+}.dat will match /resources/spring.dat and assign the 
value "spring" to the filename variable
```