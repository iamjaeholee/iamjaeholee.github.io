---layout: posttitle: "[DevOps] Monorepo vs Multirepo: 요즘 대세는 왜 모노레포일까?"date: 2026-01-17 10:14:58 +0000
categories: [Velog]---<p>프로젝트 규모가 커지다 보면 이런 고민에 빠지게 됩니다.
&quot;관리자(Admin) 페이지랑 사용자(App) 페이지가 따로 있는데, 버튼 디자인이나 로그인 로직은 똑같네? 이걸 어떻게 공유하지?&quot;</p>
<p>그냥 <strong>복사 + 붙여넣기</strong> (Ctrl+C, V) 를 하자니 수정할 때마다 두 군데 다 고쳐야 하고, <strong>npm 라이브러리</strong> 로 만들자니 배포 과정이 너무 번거롭죠.</p>
<p>이런 고민을 해결하기 위해 등장한 개념이 바로 <strong>저장소 관리 전략</strong> (Repository Strategy) 입니다.
오늘은 전통적인 강자 <strong>Multirepo</strong> 와 신흥 강자 <strong>Monorepo</strong> 의 차이점을 비교해 보겠습니다.</p>
<hr>
<h3 id="1-multirepo-polyrepo-각자도생의-시대">1. Multirepo (Polyrepo): &quot;각자도생의 시대&quot;</h3>
<p>우리가 흔히 사용하는 방식입니다. <strong>프로젝트 하나당 저장소 하나</strong> (One Repo per Project) 를 가집니다.</p>
<ul>
<li><code>my-project-admin</code> (Git 저장소 1)</li>
<li><code>my-project-app</code> (Git 저장소 2)</li>
<li><code>my-project-server</code> (Git 저장소 3)</li>
</ul>
<h4 id="✅-장점">✅ 장점</h4>
<ol>
<li><strong>명확한 경계</strong> : 팀별로 담당하는 저장소가 다르니, 서로의 코드를 건드릴 일이 없습니다. 권한 관리도 쉽죠.</li>
<li><strong>가벼움</strong> : <code>git clone</code> 할 때 내 프로젝트만 받으면 되니 속도가 빠릅니다.</li>
<li><strong>자율성</strong> : Admin 팀은 React 16을 쓰고, App 팀은 Next.js 14를 써도 전혀 상관없습니다.</li>
</ol>
<h4 id="❌-단점">❌ 단점</h4>
<ol>
<li><strong>코드 공유의 어려움</strong> : 공통 컴포넌트나 유틸 함수를 공유하려면 별도의 npm 패키지로 만들어서 배포해야 합니다. (수정 -&gt; 배포 -&gt; 버전 업 -&gt; 재설치... 😱)</li>
<li><strong>일관성 유지 실패</strong> : ESLint, Prettier 설정을 프로젝트마다 따로 해야 해서, 나중엔 코딩 스타일이 제각각이 됩니다.</li>
<li><strong>의존성 지옥</strong> : A 프로젝트는 <code>lodash</code> v3를 쓰고, B 프로젝트는 v4를 쓰는 등 버전 파편화가 일어납니다.</li>
</ol>
<hr>
<h3 id="2-monorepo-한-지붕-대가족">2. Monorepo: &quot;한 지붕 대가족&quot;</h3>
<p><strong>하나의 거대한 저장소</strong> (Single Repository) 안에 여러 개의 프로젝트가 공존하는 형태입니다.</p>
<pre><code class="language-bash">my-workspace/
├── apps/
│   ├── web/ (사용자용 Next.js)
│   └── admin/ (관리자용 React)
└── packages/
    ├── ui/ (공통 버튼, 인풋 등 디자인 시스템)
    ├── utils/ (날짜 변환 등 공통 함수)
    └── eslint-config/ (공통 린트 설정)
</code></pre>
<h4 id="✅-장점-1">✅ 장점</h4>
<ol>
<li><strong>쉬운 코드 공유</strong> : <code>packages/ui</code> 에 있는 버튼을 고치면, <code>apps/web</code> 과 <code>apps/admin</code> 에 <strong>즉시 반영</strong> 됩니다. npm 배포? 필요 없습니다.</li>
<li><strong>단일화된 설정</strong> : ESLint, TSConfig 등을 한곳에서 관리하므로 모든 팀의 코드 스타일을 통일할 수 있습니다.</li>
<li><strong>Atomic Commits</strong> : API 서버의 응답 타입이 바뀌었을 때, 프론트엔드 코드도 <strong>같은 커밋</strong> 에서 수정할 수 있습니다. (히스토리 추적에 유리)</li>
</ol>
<h4 id="❌-단점-1">❌ 단점</h4>
<ol>
<li><strong>무거운 저장소</strong> : 프로젝트가 커질수록 <code>git clone</code> 과 <code>npm install</code> 속도가 느려집니다.</li>
<li><strong>빌드 복잡도</strong> : 하나만 고쳤는데 전체를 다 빌드하면 너무 느리겠죠? 그래서 <strong>스마트한 빌드 도구</strong> 가 필수입니다.</li>
<li><strong>권한 관리</strong> : 모든 코드가 한곳에 있다 보니, 특정 프로젝트만 접근 못하게 막기가 까다롭습니다.</li>
</ol>
<hr>
<h3 id="3-모노레포를-지탱하는-도구들-tools">3. 모노레포를 지탱하는 도구들 (Tools)</h3>
<p>그냥 폴더만 합친다고 모노레포가 아닙니다.
수십 개의 프로젝트를 효율적으로 관리하기 위해 <strong>빌드 캐싱</strong> 과 <strong>의존성 관리</strong> 를 도와주는 도구들이 필수입니다.</p>
<h4 id="①-turborepo-by-vercel">① Turborepo (by Vercel)</h4>
<ul>
<li><strong>특징</strong> : 설정이 매우 쉽고 빠릅니다. &quot;이전에 빌드했던 건 다시 안 한다&quot;는 강력한 <strong>캐싱(Caching)</strong> 기능이 핵심입니다. Next.js와 찰떡궁합입니다.</li>
</ul>
<h4 id="②-nx">② Nx</h4>
<ul>
<li><strong>특징</strong> : 기능이 엄청 방대합니다. 의존성 그래프 시각화부터 배포 자동화까지 지원하지만, 배우는 데 시간이 좀 걸립니다.</li>
</ul>
<h4 id="③-yarn-workspaces--npm-workspaces">③ Yarn Workspaces / npm workspaces</h4>
<ul>
<li><strong>특징</strong> : 패키지 매니저 차원에서 <code>node_modules</code> 를 최적화해 주는 가장 기초적인 기능입니다. 보통 위 도구들과 함께 씁니다.</li>
</ul>
<hr>
<h3 id="4-결론-그래서-뭘-써야-할까">4. 결론: 그래서 뭘 써야 할까?</h3>
<p>무조건 모노레포가 정답은 아닙니다. 상황에 맞춰 선택하세요.</p>
<h4 id="👉-이럴-때-multirepo-쓰세요">👉 이럴 때 Multirepo 쓰세요</h4>
<ul>
<li>팀 간의 기술 스택이 완전히 다를 때 (Java 백엔드 + React 프론트엔드)</li>
<li>프로젝트 간에 공유할 코드가 거의 없을 때</li>
<li>초기 스타트업이라 빠르게 치고 빠져야 할 때 (설정할 시간이 없음)</li>
</ul>
<h4 id="👉-이럴-때-monorepo-쓰세요">👉 이럴 때 Monorepo 쓰세요</h4>
<ul>
<li><strong>프론트엔드 프로젝트가 2개 이상</strong> 일 때 (App, Admin, Landing...)</li>
<li><strong>디자인 시스템</strong> (UI Kit) 을 구축하고 싶을 때</li>
<li>백엔드(NestJS)와 프론트엔드(Next.js)가 <strong>타입(TypeScript)을 공유</strong> 하고 싶을 때</li>
</ul>
<hr>
<h3 id="마무리">마무리</h3>
<p>개발 트렌드는 &quot;어떻게 하면 중복을 줄이고 효율적으로 일할까?&quot; 로 흐르고 있습니다. 모노레포는 그에 대한 훌륭한 해답 중 하나죠.</p>
<p>특히 <strong>Next.js</strong> 를 사용하신다면 <strong>Turborepo</strong> 를 도입해 보는 것을 강력 추천합니다. <code>npx create-turbo@latest</code> 한 줄이면 신세계가 열리거든요!</p>
