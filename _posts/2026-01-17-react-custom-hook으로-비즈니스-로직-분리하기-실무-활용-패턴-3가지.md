---layout: post
title: "[React] Custom Hook으로 비즈니스 로직 분리하기 (실무 활용 패턴 3가지)"
date: 2026-01-17 10:07:06 +0000
categories: [Velog]
original_url: https://velog.io/@iamjaeholee/React-Custom-Hook%EC%9C%BC%EB%A1%9C-%EB%B9%84%EC%A6%88%EB%8B%88%EC%A6%88-%EB%A1%9C%EC%A7%81-%EB%B6%84%EB%A6%AC%ED%95%98%EA%B8%B0-%EC%8B%A4%EB%AC%B4-%ED%99%9C%EC%9A%A9-%ED%8C%A8%ED%84%B4-3%EA%B0%80%EC%A7%80
---
<p>리액트 개발을 하다 보면 컴포넌트 파일이 점점 뚱뚱해지는 것을 느끼실 때가 있을 겁니다.
<code>useState</code>, <code>useEffect</code>, 핸들러 함수들이 뒤섞여서 코드가 200줄, 300줄 넘어가기 시작하면 유지보수가 정말 힘들어지죠.</p>
<p>이때 필요한 것이 바로 <strong>Custom Hook</strong> (커스텀 훅) 입니다.
오늘은 제가 실무에서 자주 사용하는, <strong>코드를 획기적으로 줄여주는 커스텀 훅 패턴</strong> 3가지를 공유합니다.</p>
<hr>
<h3 id="1-왜-custom-hook을-써야-하나요-view-vs-logic">1. 왜 Custom Hook을 써야 하나요? (View vs Logic)</h3>
<p>리액트 컴포넌트는 <strong>UI를 그리는 역할</strong> (View) 에 집중해야 합니다.
데이터를 가져오고, 가공하고, 이벤트를 처리하는 <strong>비즈니스 로직</strong> (Logic) 은 훅으로 분리해 두면 다음과 같은 장점이 있습니다.</p>
<ol>
<li><strong>재사용성</strong> : 똑같은 로직을 다른 컴포넌트에서도 가져다 쓸 수 있습니다.</li>
<li><strong>가독성</strong> : 컴포넌트 코드가 깔끔해져서 &quot;아, 이건 UI만 그리는구나&quot; 하고 바로 이해할 수 있습니다.</li>
<li><strong>테스트 용이성</strong> : UI 없이 로직만 따로 테스트하기 좋습니다.</li>
</ol>
<hr>
<h3 id="2-실무-필수-hook-①--입력-폼-제어-useinput">2. 실무 필수 Hook ① : 입력 폼 제어 (useInput)</h3>
<p>로그인, 회원가입 등 입력 폼이 많은 페이지에서 <code>onChange</code> 함수를 일일이 만드는 건 정말 귀찮은 일입니다.</p>
<h4 id="🛠️-구현-코드">🛠️ 구현 코드</h4>
<pre><code class="language-typescript">import { useState, useCallback, ChangeEvent } from &#39;react&#39;;

// 제네릭 &lt;T&gt;를 사용해 어떤 타입이든 받을 수 있게 만듭니다.
export function useInput&lt;T&gt;(initialValue: T) {
  const [value, setValue] = useState&lt;T&gt;(initialValue);

  // useCallback으로 함수 재생성을 막아 최적화합니다.
  const handler = useCallback((e: ChangeEvent&lt;HTMLInputElement&gt;) =&gt; {
    setValue(e.target.value as unknown as T);
  }, []);

  const reset = useCallback(() =&gt; setValue(initialValue), [initialValue]);

  return [value, handler, reset, setValue] as const;
}
</code></pre>
<h4 id="🚀-사용-예시">🚀 사용 예시</h4>
<pre><code class="language-typescript">const [email, onChangeEmail] = useInput(&#39;&#39;);
const [password, onChangePassword] = useInput(&#39;&#39;);

// input 태그에 바로 꽂아주면 끝!
&lt;input value={email} onChange={onChangeEmail} /&gt;
</code></pre>
<hr>
<h3 id="3-실무-필수-hook-②--모달토글-제어-usetoggle">3. 실무 필수 Hook ② : 모달/토글 제어 (useToggle)</h3>
<p>모달 창을 열고 닫거나, 드롭다운 메뉴를 펼칠 때 <code>true/false</code> 를 왔다 갔다 하는 로직은 무조건 쓰입니다.</p>
<h4 id="🛠️-구현-코드-1">🛠️ 구현 코드</h4>
<pre><code class="language-typescript">import { useState, useCallback } from &#39;react&#39;;

export function useToggle(initialValue: boolean = false) {
  const [value, setValue] = useState(initialValue);

  const onToggle = useCallback(() =&gt; {
    setValue(prev =&gt; !prev);
  }, []);

  const onOpen = useCallback(() =&gt; setValue(true), []);
  const onClose = useCallback(() =&gt; setValue(false), []);

  return { value, onToggle, onOpen, onClose };
}
</code></pre>
<h4 id="🚀-사용-예시-1">🚀 사용 예시</h4>
<pre><code class="language-typescript">// 변수명도 내 마음대로 바꿀 수 있어서 직관적입니다.
const { value: isModalOpen, onToggle: toggleModal } = useToggle();

return (
  &lt;&gt;
    &lt;button onClick={toggleModal}&gt;모달 열기&lt;/button&gt;
    {isModalOpen &amp;&amp; &lt;Modal /&gt;}
  &lt;/&gt;
);
</code></pre>
<hr>
<h3 id="4-실무-필수-hook-③--첫-렌더링-무시하기-usedidmounteffect">4. 실무 필수 Hook ③ : 첫 렌더링 무시하기 (useDidMountEffect)</h3>
<p>이건 좀 고급 스킬인데, <code>useEffect</code> 는 컴포넌트가 처음 마운트 될 때도 무조건 실행됩니다.
하지만 실무에서는 <strong>&quot;처음엔 가만히 있고, 값이 변했을 때만 실행해!&quot;</strong> 라는 요구사항이 꽤 많습니다. (예: 검색 필터 변경 시 API 호출)</p>
<h4 id="🛠️-구현-코드-2">🛠️ 구현 코드</h4>
<pre><code class="language-typescript">import { useEffect, useRef } from &#39;react&#39;;

export const useDidMountEffect = (func: () =&gt; void, deps: any[]) =&gt; {
  const didMount = useRef(false);

  useEffect(() =&gt; {
    // 첫 렌더링이면 flag만 바꾸고 함수 실행은 안 함 (return)
    if (!didMount.current) {
      didMount.current = true;
      return;
    }
    // 두 번째 렌더링부터 실행됨
    func();
  }, deps);
};
</code></pre>
<hr>
<h3 id="5-정리하며">5. 정리하며</h3>
<p>리액트 잘하는 개발자의 코드를 보면 <code>return</code> (JSX) 위쪽이 아주 깔끔합니다. 복잡한 로직들은 전부 <code>useSomething</code> 이라는 이름의 훅으로 숨겨져 있기 때문이죠.</p>
<ol>
<li><strong>반복되는 로직</strong> 이 보인다? -&gt; <strong>Custom Hook</strong> 으로 뺀다.</li>
<li><strong>UI와 상관없는 데이터 처리</strong> 가 길다? -&gt; <strong>Custom Hook</strong> 으로 뺀다.</li>
</ol>
<p>이 습관만 들이셔도 여러분의 코드는 훨씬 더 전문적이고 우아해질 겁니다.</p>
