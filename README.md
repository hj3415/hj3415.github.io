문서 수정하는 과정
1. \.md 파일 작성
2. 이미지와 비디오는 .md 파일과 같은 경로에 images/videos 폴더를 만들어서 넣는다.
3. mkdocs.yml의 nav:에 경로추가

문서 수정후 배포
```sheel
mkdocs gh-deploy
```

main 브랜치 커밋은 꼭 안해도됨