# /login @Post api 호출 후, 404 에러 뜨는 이슈


### 이슈 현황

![image](https://github.com/user-attachments/assets/38aed279-fb50-446f-a7e6-9847b464506c)


- 최초 로그인 시 404에러 발생
- url = localhost:8080/;jsessionid=2A120C930461E1D8D63C3CBEF6D2186E

&nbsp;
### 기존 코드

- LoginController

```java
@PostMapping("/login")
    public String login(@ModelAttribute User user, HttpServletRequest request) {
        User loginUser = loginService.login(user.getLoginId(), user.getPassword());

        if (loginUser == null) {
            return "loginForm";
        }

        HttpSession session = request.getSession();
        session.setAttribute("loginUser", loginUser);

        return "redirect:/";
    }
```

- HomeController

```java
@GetMapping("/")
    public String home(HttpServletRequest request, Model model) {

        HttpSession session = request.getSession(false);
        if (session == null) {
            return "home";
        }

        User loginUser = (User) session.getAttribute("loginUser");
        if (loginUser == null) {
            return "home";
        }

        model.addAttribute("user", loginUser);
        return "loginHome";
    }
```

&nbsp;
### 디버깅 모드로 이슈 원인 확인하기

#### 세팅

![image 1](https://github.com/user-attachments/assets/b409f035-9e72-4cd0-9ff9-0eaf5e00c286)

![image 2](https://github.com/user-attachments/assets/2a4898af-7d74-4eb1-88a5-5eefbcd20871)

#### 진행
1. [localhost:8080](http://localhost:8080) 호출 → home 화면 호출
2. 로그인 버튼 클릭 → [http://localhost:8080/login](http://localhost:8080/login) 호출
3. 아이디, 비밀번호 입력 후 로그인 버튼 클릭 
   
    ![image 3](https://github.com/user-attachments/assets/a40d15f4-5a0c-49cb-8feb-e8a135c0347a)
    
    `User loginUser = loginService.login(user.getLoginId(), user.getPassword());` ⭕️
    
    `return "redirect:/";` ❌

    ![image 4](https://github.com/user-attachments/assets/7b090e55-fb82-46e0-8bc1-87268a52082c)
    
    `url` = [`http://localhost:8080/;jsessionid=9ED1C04BDB764ACF3DF0B147C0255659`](http://localhost:8080/;jsessionid=9ED1C04BDB764ACF3DF0B147C0255659)
    
    JSESSIONID 생성 되면서 url에 jsessionid가 붙어서 나옴
    
4. [localhost:8080](http://localhost:8080) 호출
   
    ![image 5](https://github.com/user-attachments/assets/8569f679-227d-4b6b-ad3c-3a4bc6cee0eb)
   
    loginUser 값이 제대로 호출되는 것 확인
    
    `url` = [`http://localhost:8080`](http://localhost:8080/)  (JsessionId가 붙어서 나오지 않음)
    
    ![image 6](https://github.com/user-attachments/assets/535b0c7d-d3bf-4154-9f5e-524e90a520b2)

    
6. 로그아웃 ⭕️
    
    ![image 7](https://github.com/user-attachments/assets/7c7c1d89-8976-407e-b480-287fbf285921)

    ![image 8](https://github.com/user-attachments/assets/698aaa51-3348-41dc-b970-a6ce21e36a0f)

    JSESSIONID 값이 삭제되어있지 않음
    
    ```java
    @PostMapping("/logout")
        public String logout(HttpServletRequest request) {
            HttpSession session = request.getSession(false);
            if (session != null) {
                session.invalidate();
            }
            return "redirect:/";
        }
    ```
    
    ❓session.invalidate()를 하면 쿠키에 있는 JSESSIONID가 사라지지 않는건가?
    
7. 다시 로그인 ([http://localhost:8080/login](http://localhost:8080/login))
    
    ![image 9](https://github.com/user-attachments/assets/50d1c592-a65b-4dc2-803b-b718fccf6f2b)

    ![image 10](https://github.com/user-attachments/assets/34f56a64-a5e1-4b6d-baae-a4da0c5f6fe4)

    ![image 11](https://github.com/user-attachments/assets/5f7dea8b-b11a-4eab-b6cf-7d6d15a9bf7d)

    새로운 JSESSIONID가 생성되면서 [http://localhost:8080](http://localhost:8080/) 호출
    

#### 추측🤔
처음에 JSESSIONID가 생성되는 경우, url에 /?jsessionid=~ 가 붙어서 호출되는데, redirect:/ 가 제대로 작동되지 않는 것으로 보임

&nbsp;
### 해결

### 🔍 1. 로그인을 처음 시도할 때, URL에 jsessionid가 포함되는 이유

Spring Boot는 웹 브라우저가 쿠키를 지원하지 않을 때 세션을 유지하기 위해 쿠키 대신 url에 값을 전달함

생성되는 쿠키 이름이 jsessionid이고, 값은 추정 불가능한 랜덤 값으로 이것이 url에 전달되는 것!

404 에러가 뜨는 이유는 서버가 URL을 정확하게 인식하지 못하기 때문임

#### ✅ 해결방안

1. [`application.properties`](http://application.properties)에 추가  
    
    ```java
    server.servlet.session.tracking-modes=cookie
    ```
    
    또는 `application.yml`에  추가
    
    ```java
    server:
      servlet:
        session:
          tracking-modes: cookie
    ```
    
    ⇒ 이 설정을 하면 **쿠키만 사용하도록 강제**되므로, `jsessionid`가 URL에 포함되지 않음!
    
2. `@SessionAttributes` 활용 (대체 방법)

  #### ✍🏻 적용

  application.properties에 추가했더니, jsessionId가 생성되어도 URL에 포함되지 않고 로그인 성공 화면이 뜨는 것 확인!😆

  ![image 13](https://github.com/user-attachments/assets/7dcf67eb-f391-448d-807a-f308e0363a6e)

&nbsp;
### 🔍 2. 로그아웃 후에도 `JSESSIONID` 쿠키가 남아있는 이유

1.  `session.invalidate();`는 서버의 세션만 삭제
    
    `session.invalidate();`는 서버 측의 세션 정보를 삭제(세션 무효화)할 뿐, 클라이언트(브라우저)에서 `JSESSIONID` 쿠키를 직접 삭제하지 않음.
    
2. `JSESSIONID` 쿠키는 브라우저가 자동으로 삭제해야 함

#### ✅ 해결방안

1. 쿠키를 명시적으로 삭제

```java
@PostMapping("/logout")
public String logout(HttpServletRequest request, HttpServletResponse response) {
    HttpSession session = request.getSession(false);
    if (session != null) {
        session.invalidate();
    }

    // JSESSIONID 쿠키 삭제
    Cookie cookie = new Cookie("JSESSIONID", null);
    cookie.setMaxAge(0);  // 쿠키 즉시 만료
    cookie.setPath("/");  // 쿠키 경로 설정
    response.addCookie(cookie);

    return "redirect:/";
}
```

2. `application.properties`에 추가 → 세션 쿠키 즉시 만료
   
   ```
   server.servlet.session.cookie.max-age=0
   ``` 
    
&nbsp;
### 🔍 3. `JSESSIONID`가 URL에 추가될 때 `;`(세미콜론)가 붙는 이유

서블릿에서 `HttpSession`을 사용할 때, 쿠키가 비활성화되었거나 지원되지 않는 경우에는 URL Rewriting 방식으로 `JSESSIONID`를 전달.

이때 `JSESSIONID`는 세미콜론(`;`)을 사용하여 URL에 추가되는데, 이는 서블릿의 표준 방식

📌 서블릿 스펙에 따라 `;`이 사용됨:

`?` → 쿼리 스트링(Query String) 용도로 사용됨.

`;` → 세션 정보(URL Rewriting) 저장 용도로 사용됨.
