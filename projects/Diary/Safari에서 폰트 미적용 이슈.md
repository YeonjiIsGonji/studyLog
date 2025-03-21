# Safari에서 폰트 미적용 이슈
&nbsp;
## 이슈현황

![image](https://github.com/user-attachments/assets/b46d174f-ea20-40fb-82af-d5e8b409c117)

좌 : Chrome / 우 : Safari

⚠️ Chrome에서는 잘 적용되던 폰트가 Safari에서 실행하면 적용되지 않음<br/><br/><br/>

## 원인

- 사용자의 시스템에 폰트가 설치되어 있지 않다면 **default font**가 적용됨
    
    → 개발 시, 폰트가 적용되지 않을 것을 고려해 default font를 설정해줘야함
    
- default font는 웹 브라우저마다 다름

- but 내 코드의 문제는 ‘’를 잘못써서..😵

  기존 코드:  `font-family: 'Arial, sans-serif';`

  수정 코드: `font-family: 'Arial’, ‘sans-serif';`<br/><br/><br/>

## 해결방안

`font-family : 1순위, 2순위, 3순위, …`  

- font-family에 적힌 순서대로 폰트가 적용됨
- 1순위 폰트→ 2순위 폰트 → 3순위 폰트
- 브라우저에서 사용되는 default font를 font-family 속성의 마지막 요소에 추가해주면 된다!
    
    예시 ) `font-family: 'Arial', 'sans-serif';`
    
- default font 종류
    - `serif` , `sans-serif` , `monospace`
