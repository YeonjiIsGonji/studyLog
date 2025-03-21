# [GIT] 변경 사항을 새로운 브랜치에 commit하고 싶을 때

## 상황

> 👉현재 브랜치 : main - feature/creation

- feature/creation에 스마트에디션 관련 작업을 진행하였고 git commit은 하지 않은 상태
    
    ⇒ 새로운 feature/smartedition 브랜치를 생성해서 현재 변경 사항을 반영하고 싶음
    
<br/><br/>
## 🧐 해결 방법

1️⃣ **현재 변경 사항을 그대로 둔 채 새로운 브랜치 생성**

```bash
git checkout -b feature/smarteditor
```

> 현재 변경 사항을 유지하면서 새로운 브랜치를 만듦.
> 

2️⃣ **새 브랜치에서 변경 사항을 커밋**

```bash
git add .
git commit -m "commit 메시지 작성"
```

3️⃣ **원격 저장소에도 새 브랜치를 올림**

```bash
git push origin feature/smarteditor
```
<br/><br/>
## **🛠 만약 `feature/diary-creation`을 원래 상태로 돌려놓고 싶다면?**

새 브랜치에서 커밋을 완료한 후, `feature/diary-creation`을 원래 상태로 되돌리고 싶다면:

```bash
git checkout feature/diary-creation  # 다시 원래 브랜치로 이동
git reset --hard origin/feature/diary-creation  # 원격과 동일하게 초기화
```

**⚠ 주의:** `git reset --hard`를 하면 `feature/diary-creation`에서 변경한 내용이 사라지므로, 꼭 새로운 브랜치에서 커밋한 후 실행!

<br/><br/>
## 🔍 `feature/smarteditor` 브랜치는 어디서 생성되는가?

> 👉브랜치 : main - feature/creation - feature/smartedition


💡 `feature/smartedition`은 `feature/creation`에서 파생된 브랜치!

<br/><br/>

## **📝 정리**

| 단계 | 명령어 | 설명 |
| --- | --- | --- |
| 1️⃣ | `git checkout -b feature/smarteditor` | 현재 변경 사항을 유지한 채 새 브랜치 생성 |
| 2️⃣ | `git add .` | 변경 사항을 스테이징 |
| 3️⃣ | `git commit -m "SmartEditor 적용 작업"` | 변경 사항을 커밋 |
| 4️⃣ | `git push origin feature/smarteditor` | 원격 저장소에 브랜치 업로드 |
| 5️⃣ | `git checkout feature/diary-creation` | 원래 브랜치로 돌아가기 |
| 6️⃣  | `git reset --hard origin/feature/diary-creation` | `feature/diary-creation`을 원격과 동일한 상태로 초기화 |
