## ☕️ 이펙티브 자바 스터디

📄 **스터디 소개 문서**: [Notion 링크 바로가기](https://haneum21umm.notion.site/1fa1e00dd47980bdb436db838c64a509?pvs=4)

---

### 👥 멤버

<table> 
    <tr> 
        <td align="center">
            <img src="https://avatars.githubusercontent.com/u/103233513?v=4" width="100"/><br/> 
            <sub><b>이한음</b></sub> 
        </td> 
        <td align="center"> 
            <img src="https://avatars.githubusercontent.com/2heunxun" width="100"/><br/> 
            <sub><b>유승헌</b></sub> 
        </td>
        <td align="center"> 
            <img src="https://avatars.githubusercontent.com/u/128132449?v=4" width="100"/><br/> 
            <sub><b>장영후</b></sub> 
        </td> 
    </tr> 
</table>

---


### 🔄 발표 자료 업로드 규칙

1. **브랜치 생성**
- `itemN/깃허브닉네임` 형식으로 브랜치를 생성합니다.
   ```bash
   git checkout -b item6/leehaneum
   ```

2. **스터디 내용 정리**

    * `itemN` 폴더에 마크다운 파일 작성
    * 파일명: `itemN_아이템제목_이름.md`
   ```
   item6_불필요한 객체 생성을 피하라_한음.md
   ```

3. **이미지 업로드(필요시)**

    * 이미지가 있다면 `images` 폴더에 저장
    * 마크다운에 이미지 경로로 삽입

    ```md
      ![이미지 설명](../images/파일이름.png)
    ```

4. **변경사항 커밋 및 푸시**

   ```bash
   git add .
   git commit -m "docs: item6_불필요한 객체 생성을 피하라_한음"
   git push origin item6/leehaneum
   ```

5. **PR 생성**

    * GitHub에서 `main` 브랜치 기준으로 PR 생성
    * PR 제목은 자유롭게 작성
    * PR 본문에 간단한 요약 작성

    ```
      item6: 불필요한 객체 생성을 피하라 - 한음
    ```
    

6. **리뷰 요청 및 머지**

    * 팀원 1명 이상에게 `approve` 받기
    * 승인을 받으면 `main` 브랜치에 머지

