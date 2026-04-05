# Building Systems & Better Life

BBACK의 기술 블로그 저장소입니다.  
이 저장소는 **GitHub Pages + Jekyll + Chirpy** 기반으로 운영되며, 백엔드 엔지니어링, 시스템 설계, Observability, 학습 기록, Resume, 그리고 일부 실사용 기반 정보형 콘텐츠를 함께 다룹니다.

- 사이트: <https://bback-dev.github.io>
- 공개 이름: **BBACK**
- 작성 언어: **한국어 중심**

---

## 블로그 소개

이 블로그는 단순한 개인 기록 저장소가 아니라, 아래 목적을 중심으로 운영합니다.

- 백엔드 / 시스템 설계 / 운영 경험 정리
- 학습한 내용을 검색 가능한 형태로 아카이빙
- Resume / Career 정리
- Observability, OpenTelemetry, Spring Boot 같은 실무 주제 기록
- 필요 시 리뷰 / 추천형 콘텐츠 발행

콘텐츠 방향은 다음 원칙을 따릅니다.

- 검색 의도 중심의 정보형 글 작성
- 과장 없이 실무적인 설명 우선
- 장점뿐 아니라 한계와 전제조건도 함께 설명
- AI 이미지 사용 시, 업로드 전 확인을 우선

---

## 기술 스택

- **Static Site Generator**: Jekyll
- **Theme**: Chirpy
- **Hosting**: GitHub Pages
- **Deployment**: GitHub Actions
- **Analytics**: Google Analytics 4
- **Search**: Google Search Console
- **Ads**: Google AdSense

---

## 주요 디렉터리 구조

```bash
.
├── _config.yml              # 사이트 설정
├── _posts/                  # 발행된 포스트
├── _drafts/                 # 초안 포스트
├── _tabs/                   # About, Resume 등 상단/사이드 메뉴 페이지
├── _includes/               # 커스텀 레이아웃 조각
├── _plugins/                # Jekyll 플러그인
├── assets/                  # 이미지, CSS/JS 등 정적 자산
├── .github/                 # GitHub Actions 설정
└── index.html               # 홈 진입점
```

---

## 로컬 실행 방법

### 1) 의존성 설치

```bash
bundle install
```

### 2) 로컬 서버 실행

```bash
bundle exec jekyll serve
```

기본적으로 아래 주소에서 확인할 수 있습니다.

- <http://127.0.0.1:4000>
- <http://localhost:4000>

### 3) 프로덕션 빌드 확인

```bash
bundle exec jekyll build
```

---

## 새 글 작성 방법

### 발행용 포스트

`_posts/` 아래에 아래 규칙으로 파일을 추가합니다.

```bash
YYYY-MM-DD-title.md
```

예:

```bash
2026-04-05-spring-boot-4-0-important-changes.md
```

### 초안 작성

아직 공개하지 않을 글은 `_drafts/` 에 작성합니다.

```bash
_drafts/my-draft-post.md
```

초안까지 함께 미리보기 하려면:

```bash
bundle exec jekyll serve --drafts
```

---

## 카테고리 운영 기준

현재 블로그는 대략 아래 카테고리 축으로 운영합니다.

- **Backend / Springboot**
- **Observability**
- **Insights / Blog**
- **Resume / Career**
- 필요 시 **Review / Recommendation**

카테고리는 너무 잘게 쪼개기보다, 나중에 글이 쌓여도 관리 가능한 수준으로 유지합니다.

---

## 이미지 운영 원칙

- 대표 이미지는 가능한 한 **텍스트 없는 심플한 일러스트형** 우선
- AI 생성 이미지는 가능하면 **사전 확인 후 반영**
- 깨지기 쉬운 SVG나 과도한 텍스트형 썸네일은 지양
- 공유 썸네일(OG image)까지 함께 고려해서 이미지 지정

포스트 front matter 예시:

```yaml
image:
  path: /assets/img/posts/example-cover.jpg
```

---

## 페이지 운영 기준

현재 주요 페이지는 다음과 같습니다.

- **About**
- **Resume**
- **Categories**
- **Tags**
- **Archives**
- **Privacy Policy**
- **Affiliate Disclosure**

정책성 페이지는 접근 가능해야 하지만, 메인 탐색을 해치지 않도록 푸터 중심으로 노출합니다.

---

## 배포 방식

이 저장소는 `main` 브랜치에 반영된 내용을 기준으로 GitHub Actions가 사이트를 빌드하고 GitHub Pages로 배포합니다.

기본 흐름:

1. 로컬에서 글/설정 수정
2. `bundle exec jekyll build` 로 확인
3. 커밋 및 `main` 푸시
4. GitHub Actions 자동 배포

---

## 메모

- 이 저장소는 더 이상 Chirpy starter 템플릿 소개용 저장소가 아니라, 실제 운영 중인 개인 블로그 저장소입니다.
- README는 블로그 운영 방향이 바뀌면 함께 업데이트합니다.

---

## License

기본 테마인 Chirpy는 원저작권과 라이선스를 따릅니다.  
이 저장소의 실제 콘텐츠(포스트, 소개 문구, 이미지, 커스텀 설정)는 별도 관리됩니다.
