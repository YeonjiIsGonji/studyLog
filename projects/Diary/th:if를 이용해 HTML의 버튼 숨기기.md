# th:if를 이용해 HTML의 수정/삭제 버튼을 숨기기
## 이슈현황

<img width="909" alt="image" src="https://github.com/user-attachments/assets/a3cab494-9879-48d1-9d31-0b4f33ebdb84" />

⚠️ 공유 받은 다이어리를 상세 조회할 경우, 수정/삭제 버튼이 보이는 상황<br/>
    -> 수정/삭제 버튼을 숨기고 조회만 가능하게 변경해야 함<br/><br/><br/>

## 🤔 어떻게 해결할까?

1. sharedDiary.html을 따로 만든다. → 똑같은 코드 중복이 일어남
2. diary.html에 조건문을 만들어 작성자인 경우와 아닌 경우를 구분해 수정/삭제 버튼이 보이게 함 ✅<br/><br/><br/>

### **🔍 `th:if`란? (Thymeleaf 조건문)**

`th:if`는 Thymeleaf에서 조건문을 처리하는 속성으로, 특정 조건이 `true`일 때만 해당 HTML 요소를 렌더링함

| 기능 | 문법 | 설명 |
| --- | --- | --- |
| `th:if` | `<p th:if="${user.admin}">관리자</p>` | `true`일 때만 보임 |
| `th:unless` | `<p th:unless="${user.admin}">일반 사용자</p>` | `false`일 때 보임 |

&nbsp;
## 적용

### 기존코드

- diary.html

```java
<div class="button-group">
    <button class="btn-list" th:onclick="|location.href='@{/diaries/owner}'|">목록으로</button>
    <button class="btn-edit" th:onclick="|location.href='@{/diaries/{diaryId}/edit(diaryId=${diary.id})}'|">수정
    </button>
    <form th:action="@{/diaries/{diaryId}/delete(diaryId=${diary.id})}" method="post"
          onsubmit="confirmDelete(event)">
        <button type="submit" class="btn-delete">삭제</button>
    </form>
</div>
```

- DiaryController

```java
@GetMapping("/{diaryId}")
@Operation(summary = "특정 diary 상세 조회")
public String diary(@PathVariable Long diaryId, @SessionAttribute(name = SessionConst.LOGIN_USER, required = false) User loginUser, Model model) {
    Diary diary = diaryService.findDiaryById(diaryId).orElseThrow(() ->
    new IllegalArgumentException("Invalid diary Id"));
    model.addAttribute("diary", diary);
    return "diary";
}
```

### 수정

- DiaryController에 추가

```java
boolean isOwner = diary.getAuthorId().equals(loginUser.getUserId()); 
//로그인 유저가 다이어리 작성자인지 여부 확인
model.addAttribute("isOwner", isOwner);
```

👉 **`isOwner` 변수를 뷰로 전달해서 Thymeleaf에서 버튼을 제어할 수 있도록 함**

- diary.html에 thymeleaf 조건 추가

```java
<button class="btn-edit" ❗️th:if="${isOwner}"❗️
            th:onclick="|location.href='@{/diaries/{diaryId}/edit(diaryId=${diary.id})}'|">수정
    </button>
    <form ❗️th:if="${isOwner}"❗️ th:action="@{/diaries/{diaryId}/delete(diaryId=${diary.id})}" method="post"
          onsubmit="confirmDelete(event)">
        <button type="submit" class="btn-delete">삭제</button>
    </form>
```

👉 **`isOwner`  값에 따라 수정/삭제 버튼 표시 여부 결정**

