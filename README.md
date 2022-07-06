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