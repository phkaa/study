#### SSR(Server-Side Rendering)
- 서버에서 사용자에게 보여줄 페이지를 모두 구성하여 사용자에게 페이지를 보여주는 방식이다.
- 모든 데이터가 매핑된 서비스 페이지를 클라이언트(브라우저)에게 바로 여줄 수 있다.
- 서버를 이용해 페이지를 구성하기 때문에 페이지를 구성하는 속도는 늦어지지만 전체적으로 사용자에게 보여주는 콘텐츠 구성이 완료되는 시점은 빨리지는 장점이 있다.
- SEO(Search Engine Optimization) 쉽게 구성할 수 있다.

#### SPA(Single-page Application)
- 단일 페이지 애플리케이션
- 최초 브라우저 호출시 한번 페이지를 전체 로드하고, 이후부터는 특정 부분만 fetch(ajax,axios)를 통해 데이터를 바인딩하는 방식이다.
- SEO(Search Engine Optimization) 구성이 어려울 수 있다.
- 초기에 필요한 대부분의 리소스를 다운받으므로 초기 구동속도가 느릴 수 있다.

#### Nuxt.js
- Vue 기반의 SSR 프레임워크
- SSR 모드와 SPA 모드 두가지 전부 지원한다.
- 기본 폴더 구조 및 함수([디렉토리구조](https://nuxtjs.org/docs/2.x/directory-structure/assets))
  - assets: less, sass, css, image, font 리소스를 포함한다.
  - components: 공통으로 사용할 컴포넌트(asyncData 또는 fetch 사용할 수 없음)
    - SSR인 asyncData 함수 사용 할 수 없다.
  - layouts: 애플리케이션 전체에 대한 레이아웃을 포함한다.
    - SSR인 asyncData 함수 사용 가능
  - middleware: 페이지 또는 레이아웃이 렌더링되기 전에 실행이 되며, 페이지나 레이아웃에 바인딩하였다면 해당 페이지나 레이아웃이 실행되기 전에 매번 실행된다.
  - pages: 실제 애플리케이션의 페이지 구성을 포함하여 디렉토리 구조에 따라 자동으로 router 가 생성 된다.
    - SSR인 asyncData 함수 사용 가능
    - asyncData: API를 호출하고 데이터를 하위 컴포넌트에 전달 및 data 변수에 셋팅이 가능하다. vuex store 도 가능하다.
    - fetch: API를 호출하고 데이터를 하위 컴포넌트에 전달 가능하나 data 변수에 셋팅 할 수 없다. vuex store 도 가능하다.
  - plugins: 애플리케이션에 바인딩 될 외부 혹은 내부 plugins를 포함한다. 애플리케이션이 인스턴스 화 되기 전에 실행하며 전역적으로 구성 요소를 등록하고 함수 또는 상수를 삽입할 수 있다.
  - static: 정적인 파일들을 포함한다. html, js 파일도 포함 시킬 수 있다. 혹은 서버 루트에 직접 매핑되어 바뀌지 않을 파일을 넣는다.(예. 파비콘)
  - store: 애플리케이션에서 사용될 vuex store 파일들을 포함 한다.
  - test: 테스트 코드 작성 폴더
  - nuxtServerInit: Vuex store 의 root index 에 정의되어 있을 경우 Nuxt에서 페이지 최초 접속시 한번말 실행되는 store action이다. 주로 인증과 관련된 로직을 구현한다.