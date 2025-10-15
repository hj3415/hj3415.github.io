문서 수정하는 과정
1. \.md 파일 작성
2. 이미지와 비디오는 .md 파일과 같은 경로에 images/videos 폴더를 만들어서 넣는다.
3. mkdocs.yml의 nav:에 경로추가

문서 수정후 배포
```sheel
mkdocs gh-deploy
```

main 브랜치 커밋은 꼭 안해도되지만 하는걸 추천

비디오첨부는 이런식의 코드로 작성할것(동영상경로를 절대경로로)
```html
<video controls width="800" preload="metadata" playsinline>
  <source src="/GDQUEST/2D/M9_Top_Down_Movement/L1_Top_Down_Movement_Module_Overview/videos/010_overview_result.mp4" type="video/mp4">
  동영상을 보려면 브라우저가 video 태그를 지원해야 합니다.
</video>
```

markdown 구성은
```markdown
## 중제목
### 소제목
```