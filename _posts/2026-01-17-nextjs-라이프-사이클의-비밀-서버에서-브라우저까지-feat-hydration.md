---layout: post
title: "[Next.js] 라이프 사이클의 비밀: 서버에서 브라우저까지 (feat. Hydration)"
date: 2026-01-17 10:10:09 +0000
categories: [Velog]
original_url: https://velog.io/@iamjaeholee/Next.js-%EB%9D%BC%EC%9D%B4%ED%94%84-%EC%82%AC%EC%9D%B4%ED%81%B4%EC%9D%98-%EB%B9%84%EB%B0%80-%EC%84%9C%EB%B2%84%EC%97%90%EC%84%9C-%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80%EA%B9%8C%EC%A7%80-feat.-Hydration
---
<p>리액트를 공부하다 보면 <strong>마운트(Mount)</strong> , <strong>업데이트(Update)</strong> , <strong>언마운트(Unmount)</strong> 라는 용어를 지겹도록 듣게 됩니다. 컴포넌트가 태어나고, 변하고, 죽는 과정이죠.</p>
<p>하지만 <strong>Next.js</strong> 는 여기에 <strong>&quot;서버&quot;</strong> 라는 단계가 하나 더 추가됩니다. 그래서 일반 리액트 앱보다 생명 주기가 조금 더 복잡하고 흥미롭습니다.</p>
<p>오늘은 Next.js App Router 환경에서 <strong>페이지가 어떻게 생성되고, 브라우저에서 어떻게 살아 움직이게 되는지</strong> 그 전체적인 일생을 정리해 보려 합니다.</p>
<hr>
<h3 id="1-큰-그림-nextjs의-탄생-과정-the-big-picture">1. 큰 그림: Next.js의 탄생 과정 (The Big Picture)</h3>
<p>Next.js 앱의 라이프 사이클은 크게 <strong>서버 단계</strong> 와 <strong>클라이언트 단계</strong> 로 나뉩니다.</p>
<h4 id="1단계-서버에서의-탄생-pre-rendering">1단계: 서버에서의 탄생 (Pre-rendering)</h4>
<ul>
<li>사용자가 URL을 입력하면, Next.js 서버(또는 빌드 타임)에서 <strong>React Server Component</strong> 들을 실행합니다.</li>
<li>이때 HTML 뼈대와 JSON 데이터를 생성합니다.</li>
<li><strong>중요:</strong> 이 단계에서는 브라우저 API(<code>window</code>, <code>document</code>)나 <code>useEffect</code> 가 전혀 동작하지 않습니다.</li>
</ul>
<h4 id="2단계-클라이언트로-전송-html-streaming">2단계: 클라이언트로 전송 (HTML Streaming)</h4>
<ul>
<li>만들어진 HTML을 브라우저로 냅다 던져줍니다.</li>
<li>사용자는 이때 &quot;아직 기능은 안 되지만 내용이 보이는&quot; <strong>FCP (First Contentful Paint)</strong> 를 경험합니다.</li>
</ul>
<h4 id="3단계-하이드레이션-hydration-✨">3단계: 하이드레이션 (Hydration) ✨</h4>
<ul>
<li>브라우저가 HTML을 다 그리고 나면, 뒤이어 도착한 JavaScript 번들을 실행합니다.</li>
<li>이때 리액트가 <strong>&quot;자, 이제 내가 이 HTML에 영혼(이벤트 리스너)을 불어넣을게!&quot;</strong> 라며 기존 HTML과 합체합니다.</li>
<li>이 과정이 끝나야 비로소 버튼을 클릭할 수 있는 <strong>상호작용(Interactive)</strong> 상태가 됩니다.</li>
</ul>
<hr>
<h3 id="2-함수형-컴포넌트의-생명-주기-with-hooks">2. 함수형 컴포넌트의 생명 주기 (with Hooks)</h3>
<p>하이드레이션이 끝나면, 이제 우리가 아는 익숙한 <strong>리액트 라이프 사이클</strong> 이 돌아갑니다.
클래스형 컴포넌트 시절의 메서드들이 훅(Hook)으로 어떻게 바뀌었는지 매칭해 볼까요?</p>
<h4 id="①-마운트-mounting--태어날-때">① 마운트 (Mounting) : 태어날 때</h4>
<p>컴포넌트가 화면에 처음 나타날 때 실행됩니다. API 호출이나 이벤트 등록을 주로 여기서 합니다.</p>
<pre><code class="language-typescript">// componentDidMount 대체
useEffect(() =&gt; {
  console.log(&#39;컴포넌트가 화면에 나타났습니다!&#39;);
}, []); // 의존성 배열(deps)을 비워두는 것이 핵심
</code></pre>
<h4 id="②-업데이트-updating--변할-때">② 업데이트 (Updating) : 변할 때</h4>
<p><code>props</code> 나 <code>state</code> 가 바뀌어서 리렌더링이 일어난 직후에 실행됩니다.</p>
<pre><code class="language-typescript">// componentDidUpdate 대체
useEffect(() =&gt; {
  console.log(&#39;count 값이 바뀌었습니다:&#39;, count);
}, [count]); // 감시하고 싶은 값을 배열에 넣음
</code></pre>
<h4 id="③-언마운트-unmounting--죽을-때">③ 언마운트 (Unmounting) : 죽을 때</h4>
<p>컴포넌트가 화면에서 사라질 때 실행됩니다. 메모리 누수를 막기 위해 타이머를 끄거나 라이브러리 연결을 끊을 때 사용합니다.</p>
<pre><code class="language-typescript">// componentWillUnmount 대체
useEffect(() =&gt; {
  // 로직...

  return () =&gt; {
    console.log(&#39;컴포넌트가 사라집니다. 청소(Cleanup) 시작!&#39;);
    // 여기에 clearInterval, removeEventListener 등을 작성
  };
}, []);
</code></pre>
<hr>
<h3 id="3-nextjs만의-특별한-라이프-사이클-layout-vs-template">3. Next.js만의 특별한 라이프 사이클: Layout vs Template</h3>
<p>App Router를 쓰다 보면 <strong>&quot;페이지를 이동했는데 왜 <code>useEffect</code> 가 다시 실행 안 되지?&quot;</strong> 하는 경험을 하게 됩니다. 바로 <code>layout.tsx</code> 의 특성 때문입니다.</p>
<h4 id="🏠-layout-불멸의-존재">🏠 Layout (불멸의 존재)</h4>
<ul>
<li><code>layout.tsx</code> 는 URL이 바뀌어도 <strong>Unmount 되지 않고 상태를 유지</strong> 합니다.</li>
<li>네비게이션 바나 사이드바처럼 계속 떠 있어야 하는 요소에 최적화되어 있죠.</li>
<li><strong>장점:</strong> 불필요한 리렌더링을 막아 성능이 좋습니다.</li>
<li><strong>단점:</strong> 페이지 이동 시마다 뭔가 초기화하고 싶다면 난감할 수 있습니다.</li>
</ul>
<h4 id="🔄-template-매번-환생하는-존재">🔄 Template (매번 환생하는 존재)</h4>
<ul>
<li><code>template.tsx</code> 는 <code>layout.tsx</code> 와 역할은 같지만, <strong>페이지를 이동할 때마다 새로 Mount</strong> 됩니다.</li>
<li>페이지 진입 애니메이션을 넣거나, 페이지마다 상태를 초기화해야 한다면 <code>template.tsx</code> 를 써야 합니다.</li>
</ul>
<pre><code class="language-typescript">// app/template.tsx
export default function Template({ children }: { children: React.ReactNode }) {
  useEffect(() =&gt; {
    console.log(&#39;페이지 이동할 때마다 로그가 찍힙니다!&#39;);
  }, []);

  return &lt;div&gt;{children}&lt;/div&gt;;
}
</code></pre>
<hr>
<h3 id="4-정리하며">4. 정리하며</h3>
<p>Next.js의 라이프 사이클을 이해한다는 것은 <strong>&quot;코드가 어디서(Server vs Client) 실행되고, 언제(Mount vs Update) 실행되는지&quot;</strong> 를 안다는 것입니다.</p>
<ol>
<li><strong>Server Phase</strong> : HTML 뼈대를 만듦 (Hooks 사용 불가).</li>
<li><strong>Hydration</strong> : 정적인 HTML에 리액트 기능을 붙임.</li>
<li><strong>Client Lifecycle</strong> : <code>useEffect</code> 로 마운트/언마운트 관리.</li>
<li><strong>Layout vs Template</strong> : 상태 유지가 필요하면 Layout, 매번 재생성이 필요하면 Template.</li>
</ol>
<p>이 흐름만 머릿속에 있어도, &quot;왜 데이터가 안 나오지?&quot; 혹은 &quot;왜 애니메이션이 안 먹히지?&quot; 같은 버그를 훨씬 빨리 잡으실 수 있을 거예요.</p>
