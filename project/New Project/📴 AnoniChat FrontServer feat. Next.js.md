# 📴 AnoniChat FrontServer feat. Next.js

#프로젝트 #개발 #프론트 #React #Nextjs #Front #언어 #프레임워크 #FRAMWORK

---

## [[🍌 React와 Next.js]] 참고


- 프로젝트의 기본적인 골격은 다음과 같다.

<br>

![[Pasted image 20250630173433.png|950]]

프로젝트 구조
```Lua
/
├── app/               # 페이지와 전역 레이아웃, 전역 CSS 위치
│   ├── favicon.ico
│   ├── globals.css
│   ├── layout.tsx
│   └── page.tsx
├── public/            # 정적 파일(이미지 등)
│   ├── file.svg
│   ├── globe.svg
│   ├── next.svg
│   ├── vercel.svg
│   └── window.svg
├── .gitignore
├── README.md
├── package.json
├── package-lock.json
├── next.config.ts
├── postcss.config.mjs
├── tsconfig.json
└── eslint.config.mjs
```

## 2. 각 파일의 역할

### package.json

- Node 프로젝트의 핵심 설정 파일이며, Maven/Gradle의 `pom.xml`과 비슷한 역할을 한다.
- 스크립트 정의(`dev`, `build`, `start`, `lint`)와 의존성(`react`, `next` 등)을 관리한다.

### next.config.ts

- Next.js 전용 설정 파일로, Spring Boot의 `application.properties`/`yml`과 유사한 환경 설정을 담당합니다.
- 현재는 기본 골격만 존재하며, `nextConfig` 객체 안에 필요한 옵션을 채워 넣을 수 있습니다

### tsconfig.json

- TypeScript 컴파일러 설정 파일로, `strict` 모드 여부, 모듈 해석 방법 등을 정의합니다.
- Next.js 프로젝트에서 타입 체크와 경로 별칭 등을 제어합니다

### postcss.config.mjs

- CSS 후처리를 위한 PostCSS 설정 파일입니다. 여기서는 TailwindCSS 플러그인만 활성화되어 있습니다

### eslint.config.mjs

- 코드 품질 관리를 위한 ESLint 설정 파일입니다. `next/core-web-vitals`와 `next/typescript` 규칙을 사용해 기본적인 린트 규칙을 적용합니다

### app/layout.tsx

- 모든 페이지에 공통 적용되는 레이아웃 컴포넌트로, 전역 CSS를 불러오며 `<html>` 과 `<body>` 태그를 정의합니다. Spring Boot에서 공통 레이아웃(예: `layout.jsp`, `layout.html`)을 정의하는 것과 비슷합니다

### app/page.tsx

- `/` 경로의 기본 페이지를 나타냅니다. 간단히 “Hello AnoniChat” 메시지를 보여주는 컴포넌트가 들어 있습니다

### app/globals.css

- 전역 스타일을 정의하는 CSS 파일입니다. TailwindCSS를 임포트하고 다크 모드 등 기본 변수를 설정합니다

### public/ 폴더

- 이미지 등의 정적 파일을 두는 곳으로, Spring Boot의 `resources/static` 폴더와 유사합니다.

### .gitignore

- Git 관리에서 제외할 파일 목록을 정의합니다. `node_modules`, `.next` 결과물 등 개발 시 불필요한 파일을 무시합니다


