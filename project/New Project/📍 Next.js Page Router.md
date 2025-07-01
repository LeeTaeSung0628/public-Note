# 📍 Next.js Page Router

#프로젝트 #개발 #프론트 #React #Nextjs #Front #언어 #프레임워크 #FRAMWORK #Nodejs

---

## ▶ [[🍌 React와 Next.js]] 참고

<br>

# <font color="#8db3e2">개요</font>

<br>

## Routing / Router 란? 

#### 라우팅(Routing)
- 사용자가 웹에서 URL을 입력했을 때, 어떤 페이지를 보여줄지 결정하는 동작.
- 즉, URL → 페이지 매핑 규칙이다.
- ex) `/about` 을 입력하면, `About`페이지 컴포넌트를 랜더링한다.

#### 라우터(Router)
- 라우팅을 수행하고 관리하는 시스템.
- 페이지 전환, 파라미터 처리, URL변경 감지 등을 담당.

이번에는 Next.js의 라우팅 방식의 종류를 자세하게 알아보겠다.

<br>

---

<br>

# <font color="#8db3e2">Next.Js의 라우팅 방식 (파일 기반 / 동적 / 캐치올 / 중첩 라우팅)</font>

<br>

Next.js는 **파일 시스템 기반 라우팅(file-system based routing)** 을 사용한다.

즉, <u>폴더와 파일 구조 자체</u>가 URL 라우트로 매핑된다.

<br>

#### 파일 기반 라우팅
- `pages` 디렉터리나 `app` 디렉터리에 파일을 추가하면 곧바로 동일한 경로의 페이지가 생성.
- 예를 들어 `pages/about.js` 파일을 만들면 `/about` 경로에 해당 페이지가 생성되는 식이다.
- **페이지 디렉터리**에 파일을 추가하는 것만으로 새로운 웹 페이지(라우트)가 만들어지므로, 초보자도 쉽게 URL 경로를 정의할 수 있습니다. 
    
- **동적 라우팅(Dynamic Routing)**: 경로 일부가 가변적인 경우 _동적 경로_를 사용할 수 있습니다. 파일이나 폴더 이름을 `[param]` 형태의 **대괄호**로 감싸면 동적 세그먼트로 인식되어, 요청 시 해당 부분을 변수로 처리합니다[nextjs.org](https://nextjs.org/docs/pages/building-your-application/routing/dynamic-routes#:~:text=A%20Dynamic%20Segment%20can%20be,slug)[nextjs.org](https://nextjs.org/docs/app/api-reference/file-conventions/dynamic-routes#:~:text=A%20Dynamic%20Segment%20can%20be,Dynamic%20Segment%20for%20blog%20posts). 예를 들어 `pages/blog/[slug].js`는 `/blog/hello`나 `/blog/world` 등으로 요청할 때 `[slug]` 자리에 들어온 값을 읽어 각각의 페이지를 렌더링합니다. 동적 경로 내부에서 Next.js **Router**를 통해 해당 파라미터에 접근할 수 있습니다. (Pages Router의 경우 `useRouter` 훅의 `router.query.slug`로 접근하고, App Router의 경우 컴포넌트의 `params` 프로퍼티로 접근합니다.)
    
- **캐치올 라우팅(Catch-all Routing)**: 여러 경로 세그먼트를 한 번에 캡처하고자 할 때는 `[...]` 형태로 **세 개의 점**을 사용합니다. 예를 들어 `pages/shop/[...slug].js` 파일을 만들면 `/shop` 하위의 임의의 경로에 모두 매칭됩니다. `/shop/clothes`, `/shop/clothes/tops` 등 **여러 수준의 경로**를 하나의 페이지로 처리할 수 있습니다[nextjs.org](https://nextjs.org/docs/pages/building-your-application/routing/dynamic-routes#:~:text=Dynamic%20Segments%20can%20be%20extended,segmentName)[nextjs.org](https://nextjs.org/docs/app/api-reference/file-conventions/dynamic-routes#:~:text=Catch). 이 때 `router.query.slug` (또는 App Router의 `params.slug`)는 배열 형태로 여러 세그먼트 값을 담게 됩니다.
    
- **옵셔널 캐치올(Optional Catch-all)**: 캐치올 경로를 정의하면서 해당 경로가 **없을 때도** 페이지를 매칭시키려면 대괄호를 두 번 겹친 `[[...param]]` 형태를 사용합니다. 예를 들어 `pages/shop/[[...slug]].js` 파일은 `/shop` 경로 자체와 `/shop/` 이하의 모든 하위 경로를 모두 처리합니다[nextjs.org](https://nextjs.org/docs/pages/building-your-application/routing/dynamic-routes#:~:text=Catch,segmentName)[nextjs.org](https://nextjs.org/docs/app/api-reference/file-conventions/dynamic-routes#:~:text=Optional%20Catch). 옵셔널 캐치올을 사용하면 `/shop`처럼 파라미터가 없을 경우에도 페이지가 렌더링되며, 이때 파라미터 값은 `undefined`(혹은 빈 배열)로 처리됩니다.
    
- **중첩 라우팅(Nested Routing)**: 경로를 다단계로 중첩시켜 계층적인 URL을 만들 수 있습니다. 예를 들어 `/blog/category/[slug]` 같은 경로는 **세gment**가 여러 층으로 이루어진 중첩 경로입니다. Next.js에서는 **폴더를 중첩**시켜 이러한 URL 세그먼트를 표현하고, 폴더 안에 페이지 파일을 넣어 각 세그먼트별 UI를 구성합니다[nextjs.org](https://nextjs.org/docs/app/getting-started/layouts-and-pages#:~:text=A%20nested%20route%20is%20a,is%20composed%20of%20three%20segments)[nextjs.org](https://nextjs.org/docs/app/getting-started/layouts-and-pages#:~:text=In%20Next). 예를 들어 `pages/blog/[slug].js`는 `/blog/x` 경로에 대응하고, 더 깊은 중첩 경로 `/blog/[slug]/[commentId]`를 만들고 싶다면 `pages/blog/[slug]/[commentId].js` 처럼 폴더를 계층적으로 추가하면 됩니다. App Router에서도 마찬가지로 `app` 디렉터리 내에 폴더를 나란히/계층적으로 배치하여 중첩된 경로를 정의합니다.
    

## 2. 폴더 구조 예시 및 규칙 (`pages/` vs `app/` 디렉터리)

Next.js에서는 프로젝트 폴더 내에 **`pages/` 디렉터리**와(또는) **`app/` 디렉터리**를 통해 라우트를 정의합니다. Next.js 13부터 도입된 `app/` 디렉터리는 새로운 App Router를 활용하며, 기존의 `pages/` 디렉터리는 Pages Router로 동작합니다[nextjs.org](https://nextjs.org/docs/app/guides/migrating/app-router-migration#:~:text=,a%20route%20segment%20publicly%20accessible). 두 디렉터리 모두 파일 시스템 기반으로 동작하지만, 구조와 규칙에 약간의 차이가 있습니다:

- **Pages Router (`pages/` 디렉터리)**: `pages` 폴더 아래 파일 이름이 곧 URL 경로가 됩니다. 예를 들어 `pages/index.js`는 홈 경로(`/`), `pages/about.js`는 `/about` 페이지를 의미합니다. 하위에 폴더를 만들면 URL 경로에 해당 폴더 이름이 세그먼트로 추가됩니다. 또한 파일/폴더 이름에 **대괄호 규칙**을 적용하여 동적 경로를 만들 수 있습니다[nextjs.org](https://nextjs.org/docs/pages/building-your-application/routing/dynamic-routes#:~:text=A%20Dynamic%20Segment%20can%20be,slug). 파일명을 `[param].js`로 하면 동적 라우트, `[...param].js`로 하면 캐치올 라우트, `[[...param]].js`로 하면 옵셔널 캐치올 라우트로 동작합니다[nextjs.org](https://nextjs.org/docs/app/api-reference/file-conventions/dynamic-routes#:~:text=Optional%20Catch). Pages Router에서는 파일 하나가 하나의 페이지 컴포넌트이며, **`_app.js`**, **`_document.js`** 같은 특별한 파일로 전체 레이아웃이나 `<html>` 구조를 설정합니다.
    
- **App Router (`app/` 디렉터리)**: `app` 폴더는 Next.js 13+ 버전에서 도입된 새로운 라우팅 방식으로, **폴더가 경로 세그먼트**를 나타내고 그 내부의 `page.js` (혹은 `page.tsx`) 파일이 해당 경로의 페이지 내용을 담당합니다[nextjs.org](https://nextjs.org/docs/app/guides/migrating/app-router-migration#:~:text=,a%20route%20segment%20publicly%20accessible). 즉, 각 폴더마다 반드시 `page.jsx/tsx` 파일이 있어야 그 폴더 경로가 외부에 노출됩니다. 예를 들어 `app/page.tsx`는 `/` 경로의 페이지, `app/about/page.tsx`는 `/about` 페이지를 의미합니다[frontendeng.dev](https://www.frontendeng.dev/blog/26-difference-between-app-and-pages-in-nextjs#:~:text=App%20is%20the%20new%20directory,about). 동적 경로는 폴더 이름을 `[param]`으로 만들어 표현하고, 해당 폴더 안에 `page.js`를 두는 방식입니다 (예: `app/blog/[slug]/page.tsx`는 `/blog/[임의값]` 경로)[nextjs.org](https://nextjs.org/docs/app/getting-started/layouts-and-pages#:~:text=You%20can%20continue%20nesting%20folders,file). 마찬가지로 `[...param]` 폴더를 만들어 캐치올 경로를, `[[...param]]` 폴더로 옵셔널 캐치올 경로를 구현할 수 있습니다[nextjs.org](https://nextjs.org/docs/app/api-reference/file-conventions/dynamic-routes#:~:text=Catch,folderName). App Router에서는 _각 폴더별로_ 선택적으로 `layout.js`를 정의해 그 하위 경로들에 공통 레이아웃을 적용할 수 있고, `pages/_app.js`나 `_document.js` 대신 `app/layout.js` (루트 레이아웃)을 통해 앱 전역의 레이아웃과 `<html><body>` 구조를 설정합니다.
    

아래는 `pages/`와 `app/` 디렉터리 구조 예시입니다. 폴더와 파일의 이름이 어떻게 URL로 매핑되는지 주석으로 표시했습니다:

bash

복사편집

`my-nextjs-project/ ├── pages/                  # Pages Router 사용 시 │   ├── index.js            // '/' (홈 페이지) │   ├── about.js            // '/about' │   ├── blog/ │   │   ├── index.js        // '/blog' │   │   ├── [slug].js       // '/blog/:slug' (동적 라우팅) │   │   └── [[...slug]].js  // '/blog' 및 '/blog/...'(옵셔널 캐치올) │   └── api/ │       └── hello.js        // API 라우트 예시 ('/api/hello') └── app/                    # App Router 사용 시     ├── layout.js           // 루트 레이아웃 (모든 페이지에 공통 적용)     ├── page.js             // '/' (홈 페이지)     ├── about/     │   └── page.js         // '/about'     └── blog/         ├── layout.js       // '/blog' 관련 페이지 레이아웃         ├── page.js         // '/blog'         └── [slug]/             └── page.js     // '/blog/:slug' (동적 라우팅)`

위 구조에서 볼 수 있듯, `pages` 디렉터리에서는 파일 자체가 경로이고, `app` 디렉터리에서는 각 폴더가 경로 세그먼트이며 그 안의 `page.js` 파일이 실제 페이지를 담당합니다. 폴더/파일에 대괄호(`[]`)를 사용한 규칙은 양쪽에 공통으로 적용되어 동적 라우트나 캐치올 라우트를 만들 수 있습니다[nextjs.org](https://nextjs.org/docs/app/getting-started/layouts-and-pages#:~:text=Wrapping%20a%20folder%20name%20in,blog%20posts%2C%20product%20pages%2C%20etc). 다만 App Router에서는 **페이지 컴포넌트 파일명이 항상 `page`**로 고정인 점이 다르고, 폴더 구조를 자유롭게 깊게 중첩하여 복잡한 경로도 표현할 수 있습니다 (예: `app/products/[category]/[item]/page.tsx` → `/products/:category/:item` 경로).

## 3. App Router와 Pages Router의 차이점

Next.js 13 버전에서 **App Router** (`app/` 디렉터리)가 도입되면서 기존 **Pages Router** (`pages/` 디렉터리)와 여러 측면에서 차이가 생겼습니다[frontendeng.dev](https://www.frontendeng.dev/blog/26-difference-between-app-and-pages-in-nextjs#:~:text=Nextjs%20introduced%20a%20new%20app,are%20starting%20a%20new%20project). 주요 차이점을 기능, 동작 방식 측면에서 정리하면 다음과 같습니다:

- **파일 구조 및 라우팅 방식**: Pages Router는 `pages` 폴더에 파일을 생성하면 자동으로 라우트가 생성되는 **관례 중심(convention)**의 방식입니다. 반면 App Router는 `app` 폴더 내에 **폴더와 특별한 파일명**(`page`, `layout` 등)을 사용하여 라우트를 보다 구조적으로 정의합니다[nextjs.org](https://nextjs.org/docs/app/guides/migrating/app-router-migration#:~:text=,a%20route%20segment%20publicly%20accessible). Pages Router에서는 `_app.js`, `_document.js`, `_error.js` 등을 통해 전역 설정을 했지만, App Router에서는 이러한 파일들이 **루트 레이아웃** (`app/layout.js`)이나 각 경로별 `layout.js`, 그리고 `error.js`, `loading.js` 등으로 대체되었습니다[nextjs.org](https://nextjs.org/docs/app/guides/migrating/app-router-migration#:~:text=,been%20replaced%20with%20a%20single). 즉, **App Router는 중첩 레이아웃**과 에러 처리, 로딩 상태 등을 파일 시스템에 선언적으로 구성할 수 있습니다.
    
- **렌더링 방식 (클라이언트/서버 컴포넌트)**: App Router의 도입과 함께 **React Server Components(RSC)**가 기본적으로 활용됩니다. App Router의 페이지 컴포넌트는 기본적으로 **서버 컴포넌트**로 렌더링되어, 클라이언트로 불필요한 자바스크립트를 보내지 않고도 HTML을 그려내는 방식입니다[frontendeng.dev](https://www.frontendeng.dev/blog/26-difference-between-app-and-pages-in-nextjs#:~:text=,RSC). 이는 Pages Router 시절의 페이지 컴포넌트들이 **항상 클라이언트 컴포넌트로 실행**되던 것과 큰 차이점입니다. (Pages Router에서도 SSR을 지원하지만, SSR 결과를 하이드레이션하는 형태이고 컴포넌트 자체는 클라이언트에서 동작했습니다.) App Router에서는 `"use client"` 지시어를 통해 클라이언트 컴포넌트로 전환할 수 있고, 기본은 서버에서 렌더링됩니다. 이로 인해 초기 페이지 로드 성능과 코드 분할 측면에서 이점이 있지만, 반대로 Pages Router보다 개념이 다소 복잡해졌습니다[dev.to](https://dev.to/dcs-ink/nextjs-app-router-vs-pages-router-3p57#:~:text=Simplicity%20vs,suited%20for%20complex%20routing%20scenarios).
    
- **데이터 패칭(Data Fetching) 방식**: Pages Router에서는 `getStaticProps`, `getServerSideProps`, `getStaticPaths`, `getInitialProps` 등의 **특수한 데이터 패칭 함수**를 페이지 컴포넌트에서 **export**하여 Next.js가 빌드 타임 혹은 요청 시 데이터를 공급했습니다. App Router에서는 이러한 함수를 **지원하지 않으며**, 대신 **`fetch` API와 React의 비동기 컴포넌트** 패턴을 사용한 **간결한 데이터 패칭** 방식으로 대체되었습니다[frontendeng.dev](https://www.frontendeng.dev/blog/26-difference-between-app-and-pages-in-nextjs#:~:text=The%20pages%20directory%20uses%20getServerSideProps,and%20async%20React%20Server%20Components)[nextjs.org](https://nextjs.org/docs/app/guides/migrating/app-router-migration#:~:text=,been%20replaced%20with%20a%20single). 예를 들어 Pages Router에서 정적 데이터를 가져오려면 `getStaticProps`를 구현해야 했지만, App Router의 페이지 컴포넌트에서는 그냥 `await fetch()`를 호출하면 Next.js가 해당 데이터를 빌드 시 캐싱하여 SSR해주는 식입니다. 또한 `getStaticPaths`는 App Router에서 **`generateStaticParams`**라는 함수로 대체되어, 빌드 시 생성할 동적 경로 목록을 정의합니다[nextjs.org](https://nextjs.org/docs/app/guides/migrating/app-router-migration#:~:text=,been%20replaced%20with%20a%20single). 요약하면, **Pages Router는 명시적인 데이터 패칭 함수 기반**, **App Router는 React 컴포넌트 내에서 직접 비동기 처리**를 하는 형태로 달라졌습니다. 이 변화로 인해 App Router 쪽이 데이터 패칭 코드가 간소화되고 타입스크립트 사용이 쉬워졌지만, 기존 Pages Router 함수를 사용하던 코드는 마이그레이션이 필요합니다.
    
- **레이아웃과 중첩 UI**: Pages Router에서는 페이지 간 공통 레이아웃을 구현하기 위해 `_app.js`에서 모든 페이지에 공통 레이아웃을 넣거나, 각 페이지 컴포넌트를 감싸는 식으로 처리했습니다. 반면 App Router는 **중첩 레이아웃(Nested Layout)**을 프레임워크 차원에서 지원하여, 폴더마다 `layout.js`를 추가함으로써 **해당 경로 이하의 모든 페이지에 자동으로 레이아웃이 적용**되도록 합니다[nextjs.org](https://nextjs.org/docs/app/getting-started/layouts-and-pages#:~:text=Nesting%20layouts)[nextjs.org](https://nextjs.org/docs/app/getting-started/layouts-and-pages#:~:text=If%20you%20were%20to%20combine,slug%5D%2Fpage.js). 예를 들어 `app/dashboard/layout.js`와 `app/dashboard/page.js`가 있다면 대시보드 레이아웃 안에 여러 페이지가 렌더링되며 상태도 보존됩니다. 또한 서로 다른 레이아웃을 병렬로 구성하는 **Parallel Routes**나 경로를 조건부로 바꾸는 **Intercepting Routes** 등 App Router만의 고급 라우팅 기능들도 제공됩니다. 이러한 유연성 덕분에 App Router는 **복잡한 UI 구조나 대시보드형 앱** 등에 적합합니다[dev.to](https://dev.to/dcs-ink/nextjs-app-router-vs-pages-router-3p57#:~:text=Simplicity%20vs,suited%20for%20complex%20routing%20scenarios).
    
- **권장 사용 및 마이그레이션**: Next.js 13 이후 Vercel 측은 **새로운 앱을 만들 때 App Router 사용을 권장**하고 있습니다[frontendeng.dev](https://www.frontendeng.dev/blog/26-difference-between-app-and-pages-in-nextjs#:~:text=Nextjs%20introduced%20a%20new%20app,are%20starting%20a%20new%20project). App Router가 초기에는 실험적이었지만 현재는 안정화되었고, 향후 Next.js에서는 Pages Router보다 App Router 중심으로 발전이 예상됩니다 (장기적으로 Pages Router가 **deprecated**될 가능성이 높습니다[frontendeng.dev](https://www.frontendeng.dev/blog/26-difference-between-app-and-pages-in-nextjs#:~:text=official%20documentation%20here)). 기존 Pages Router 기반 프로젝트는 점진적으로 마이그레이션할 수 있습니다. Next.js는 **두 라우터를 동시 사용**하는 것도 허용하기 때문에 (`pages/`와 `app/` 폴더를 함께 두는 방식), 일부 페이지부터 `app/` 디렉터리로 옮겨가는 **점진적 도입**도 가능합니다[nextjs.org](https://nextjs.org/docs/app/guides/migrating/app-router-migration#:~:text=Next,exists%20with%20the%20%60pages%60%20directory). 마이그레이션 시에는 기존의 `_app.js` 설정을 `app/layout.js`로 이전하고, 데이터 패칭 로직은 앞서 언급한 새로운 방식으로 변경하는 등 단계별 작업이 필요합니다[nextjs.org](https://nextjs.org/docs/app/guides/migrating/app-router-migration#:~:text=,been%20replaced%20with%20a%20single). Pages Router의 기존 페이지들은 `app` 디렉터리로 이동하지 않는 한 그대로 동작하므로, **업그레이드해도 당장 App Router로 전환할 필요는 없지만**, 장기적으로는 App Router로 전환하는 것이 권장됩니다.
    

## 4. 커스텀 라우터 설정: `next.config.js`의 `rewrites`, `redirects`, `headers`

Next.js에서는 라우팅을 세부적으로 제어하기 위해 **`next.config.js`** 파일에서 몇 가지 옵션을 제공합니다. `redirects`, `rewrites`, `headers`가 그것으로, 경로를 바꾸거나 리다이렉트하며 특정 응답 헤더를 붙이는 등의 **커스텀 라우팅 규칙**을 정의할 수 있습니다. 이 설정들은 모두 비동기 함수 형태로 배열을 반환하는 패턴이며, Next.js 서버 빌드 시 적용됩니다. 각 옵션의 역할과 사용법은 다음과 같습니다:

- **리디렉션(redirects)**: 들어온 요청 경로를 다른 경로로 **영구 혹은 일시적으로 이동**시키는 규칙입니다. 예를 들어 오래된 URL 구조를 새로운 구조로 변경한 경우나 특정 페이지를 별도의 페이지로 이동시키고 싶을 때 사용합니다[nextjs.org](https://nextjs.org/docs/app/guides/redirecting#:~:text=The%20,are%20known%20ahead%20of%20time). `next.config.js`에서 `async redirects()` 함수를 export하고, 내부에서 `source` (기존 경로 패턴), `destination` (목표 경로), `permanent` (영구여부, true면 308 Permanent Redirect / false면 307 Temporary Redirect)를 지정하는 객체 목록을 반환하면 됩니다. 지정된 경로로 요청이 오면 **브라우저가 새로운 경로로 리디렉션**되며 URL도 변경됩니다[nextjs.org](https://nextjs.org/docs/app/api-reference/config/next-config-js/rewrites#:~:text=Rewrites%20allow%20you%20to%20map,to%20a%20different%20destination%20path)[nextjs.org](https://nextjs.org/docs/app/api-reference/config/next-config-js/rewrites#:~:text=Rewrites%20act%20as%20a%20URL,and%20show%20the%20URL%20changes).
    
    예시 – `/about` 경로를 홈(`/`)으로 영구 리디렉션:
    
    js
    
    복사편집
    
    `// next.config.js module.exports = {   async redirects() {     return [       { source: '/about', destination: '/', permanent: true },     ];   }, };`
    
    위 설정에 따르면 사용자가 `/about`으로 접속하면 브라우저가 상태 코드 308과 함께 `/` 경로로 이동하게 됩니다. (`permanent: false`로 설정하면 307 Redirect로 처리됩니다[nextjs.org](https://nextjs.org/docs/app/guides/redirecting#:~:text=,redirects%20at%20scale%20for%20more).)
    
- **리라이팅(rewrites)**: 특정 경로로 들어온 요청을 **다른 경로의 콘텐츠로 매핑**하지만, **URL은 바꾸지 않고** 그대로 유지시킵니다[nextjs.org](https://nextjs.org/docs/app/api-reference/config/next-config-js/rewrites#:~:text=Rewrites%20allow%20you%20to%20map,to%20a%20different%20destination%20path). 이를 통해 사용자에게는 URL이 그대로 보이면서, 내부적으로는 다른 페이지를 보여주거나 외부 사이트의 내용을 프록시할 수도 있습니다[nextjs.org](https://nextjs.org/docs/app/api-reference/config/next-config-js/rewrites#:~:text=Rewrites%20act%20as%20a%20URL,and%20show%20the%20URL%20changes). 설정 방법은 `async rewrites()` 함수에서 `source`와 `destination`을 지정하는 것으로, `redirects`와 유사하지만 `permanent` 옵션이 없고 브라우저에 주소 변경을 알리지 않습니다.
    
    예시 – `/blog/:slug` 경로의 내용을 내부적으로 `/news/:slug` 경로에서 가져오도록 rewrite:
    
    js
    
    복사편집
    
    `// next.config.js module.exports = {   async rewrites() {     return [       { source: '/blog/:slug*', destination: '/news/:slug*' },     ];   }, };`
    
    위처럼 설정하면 사용자가 `/blog/hello`로 접속했을 때 실제로는 `/news/hello`의 페이지 내용이 렌더되지만, 브라우저 주소 창에는 여전히 `/blog/hello`로 보이게 됩니다[nextjs.org](https://nextjs.org/docs/app/api-reference/config/next-config-js/rewrites#:~:text=Rewrites%20allow%20you%20to%20map,to%20a%20different%20destination%20path). Wildcard (`*`)나 정규식 그룹 등 고급 패턴 매칭도 지원하여 유연하게 경로를 매핑할 수 있습니다[nextjs.org](https://nextjs.org/docs/app/api-reference/config/next-config-js/rewrites#:~:text=To%20match%20a%20regex%20path,blog%2Fabc)[nextjs.org](https://nextjs.org/docs/app/api-reference/config/next-config-js/rewrites#:~:text=next). 또한 `has`/`missing` 필드를 사용하면 특정 헤더나 쿠키, 쿼리 파라미터 존재 여부에 따라 조건부로 rewrites를 적용할 수도 있습니다[nextjs.org](https://nextjs.org/docs/app/api-reference/config/next-config-js/rewrites#:~:text=module.exports%20%3D%20,has%3A%20%5B)[nextjs.org](https://nextjs.org/docs/app/api-reference/config/next-config-js/rewrites#:~:text=source%3A%20%27%2F%3Apath,destination%3A%20%27%2Fhome%3Fauthorized%3D%3Aauthorized%27%2C).
    
- **헤더(headers)**: 특정 경로의 응답에 **사용자 정의 HTTP 헤더**를 추가하고 싶을 때 사용합니다. 주로 보안 헤더(CSP, X-Frame-Options 등)나 캐시 제어 헤더 등을 전역으로 적용할 때 유용합니다. `async headers()` 함수에서 `source` 경로 패턴과 `headers` 키 배열을 반환하면 되며, 각 헤더 객체는 `key` (헤더명)과 `value` (헤더값)로 구성됩니다[nextjs.org](https://nextjs.org/docs/pages/api-reference/config/next-config-js/headers#:~:text=Headers%20allow%20you%20to%20set,request%20on%20a%20given%20path)[nextjs.org](https://nextjs.org/docs/pages/api-reference/config/next-config-js/headers#:~:text=,header).
    
    예시 – `/about` 페이지에 커스텀 헤더 추가:
    
    js
    
    복사편집
    
    `// next.config.js module.exports = {   async headers() {     return [       {         source: '/about',         headers: [           { key: 'X-Custom-Header', value: 'my custom header value' },           { key: 'X-Another-Header', value: 'another value' },         ],       },     ];   }, };`
    
    위 설정은 `/about` 경로를 요청할 때 응답 헤더에 `X-Custom-Header: my custom header value`와 `X-Another-Header: another value`를 포함시킵니다. `source`에는 와일드카드 패턴 (`/docs/:slug*` 등)을 사용할 수도 있어 다수의 경로에 한꺼번에 헤더를 적용할 수 있습니다[nextjs.org](https://nextjs.org/docs/pages/api-reference/config/next-config-js/headers#:~:text=Path%20matches%20are%20allowed%2C%20for,world%60%20%28no%20nested%20paths)[nextjs.org](https://nextjs.org/docs/pages/api-reference/config/next-config-js/headers#:~:text=Wildcard%20Path%20Matching). 헤더 설정은 파일 시스템 라우팅보다 **우선 적용**되며, 중복되는 헤더 키가 여러 규칙에 걸리면 마지막에 정의된 값으로 덮어쓰입니다[nextjs.org](https://nextjs.org/docs/pages/api-reference/config/next-config-js/headers#:~:text=Headers%20are%20checked%20before%20the,files).