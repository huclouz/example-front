# booksearch-front

> Example-FrontPage(VueJS 2.2)
## 데모페이지
- http://example.hutt.co.kr/

## Build Setup

``` bash
# 의존 다운로드
npm install

# 개발버전 프론트엔드 구동( Port: 9090 )
npm run dev

# 상용 버전 빌드
npm run build
```

## 빌드 후 단계
1. 빌드 이후 dist/static 폴더에 생성되는 js, css복사
2. backend프로젝트 내의 /src/main/resources/static/에 붙여넣기
3. src/main/webapp/WEB-INF/jsp/index.jsp 파일 내용은 index.html의 내용으로 대체
4. static resource의 경로가 다르므로 index.jsp에서 /static/ 내용을 제거 해주면 됩니다.

