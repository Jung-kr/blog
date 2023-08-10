---
title: "Github로 블로그 구축하기 (with Hugo)"
date: 2023-08-10T22:30:18+09:00
draft: false
---

## 1. Hugo 설치
---

### 1.1 [github.com/gohugoio](https://github.com/gohugoio/hugo/releases)에서 본인 OS에 맞는 버전 다운로드

- 저는 Window를 사용 중이라 'hugo_0.117.0_windows-amd64.zip'을 다운받았습니다
- `C:\Hugo\bin\` 디렉토리 생성 후 압축을 해제합니다


### 1.2 환경변수 설정

- Window 검색창에 `시스템 환경변수 편집` 검색 후 오른쪽 하단 `환경 변수`를 클릭합니다
- 위 박스 Path 클릭 후 `C:\Hugo\bin` 추가합니다
- `$ hugo version`으로 환경변수 설정이 잘됐는지 확인합니다
- git은 이미 설치되어 있다고 가정하고 진행하겠습니다


<br><br>

## 2. 프로젝트 만들기
---

- 명령어를 통해 hugo 프로젝트를 생성합니다
```
C:\Hugo> hugo new site <프로젝트 이름>
```
저는 blog라는 이름으로 프로젝트를 생성했습니다


<br><br>

## 3. 프로젝트 테마 다운로드
---
- https://themes.gohugo.io/ 에서 테마를 선택합니다.

저는 이 테마가 마음에 드네요 
README 파일에 설치 방법이 자세히 적혀있습니다

```
C:\Hugo\blog> git init
C:\Hugo\blog> git clone https://github.com/adityatelange/hugo-PaperMod themes/PaperMod --depth=1
C:\Hugo\blog> git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```
- `C:\Hugo\blog` 경로 안의 `hugo.toml` 파일에 `theme = "<테마 이름>"`를 추가합니다

<br><br>

## 4. 글 생성
---
```
C:\Hugo\blog> hugo new <파일 이름>
```
- `hugo new docs/test.md`를 입력하면 `C:\Hugo\blog\content\docs` 밑에 `test.md` 파일이 생성됩니다

```
---
title: "Test"
date: 2023-08-10T22:30:12+09:00
draft: false
---

### Test 페이지입니다.
Test 페이지

```
- `test.md` 파일을 수정합니다
```
C:\Hugo\blog> hugo server -D
```
- 로컬 http://localhost:1313로 접속하고 블로그가 실제로 어떻게 보여지는지 확인할 수 있습니다.

<br><br>

## 5. Github 작업
---
### 5.1 컨텐츠 블로그에 업로드
- Github에서 git repository 생성합니다 (저는 blog라는 이름의 repository를 생성)
- hugo.toml의 baseURL을 `http://<username>.github.io/<reop_name>/`으로 변경합니다
```
C:\Hugo\blog> git add .
C:\Hugo\blog> commit -m "initial commit"
C:\Hugo\blog> git branch -M main
C:\Hugo\blog> git remote add origin https://github.com/<username>/<reop_name>
C:\Hugo\blog> git push -u origin main
```

### 5.2 컨텐츠 배포 자동화
- `gh.pages` 브랜치 생성
- 생성한 repository -> Setting -> Actions -> General -> Read and write permissions 체크 후 저장합니다
- git bash로 가서 명령어를 입력합니다
```
$ mkdir -p .github/workflows
$ touch .github/workflows/deploy.yml
$ vi .github/workflows/deploy.yml
```
- deploy.yml 파일에 밑의 내용 삽입합니다
```
name: Publish to GH Pages
on:
  push:
    branches:
      - main
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source
        uses: actions/checkout@v3
        with:
          submodules: true

      - name: Checkout destination
        uses: actions/checkout@v3
        if: github.ref == 'refs/heads/main'
        with:
          ref: gh-pages
          path: built-site

      - name: Setup Hugo
        run: |
          curl -L -o /tmp/hugo.tar.gz 'https://github.com/gohugoio/hugo/releases/download/v0.110.0/hugo_extended_0.110.0_linux-amd64.tar.gz'
          tar -C ${RUNNER_TEMP} -zxvf /tmp/hugo.tar.gz hugo          
      - name: Build
        run: ${RUNNER_TEMP}/hugo

      - name: Deploy
        if: github.ref == 'refs/heads/main'
        run: |
          cp -R public/* ${GITHUB_WORKSPACE}/built-site/
          cd ${GITHUB_WORKSPACE}/built-site
          git add .
          git config user.name 'dhij'
          git config user.email 'davidhwang.ij@gmail.com'
          git commit -m 'Updated site'
          git push          
```
- 로컬의 내용을 git repository에 push합니다
```
C:\Hugo\blog> git add .
C:\Hugo\blog> commit -m "add the first page"
C:\Hugo\blog> git push
```
- 배포가 정상적으로 완료됐습니다!