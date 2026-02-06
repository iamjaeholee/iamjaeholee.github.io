---layout: posttitle: "[TypeScript] type vs interface, 언제 무엇을 써야 할까? (완벽 비교)"date: 2026-01-17 10:00:49 +0000
categories: [Velog]---<p>TypeScript를 처음 공부할 때 가장 헷갈리는 것이 하나 있습니다.
바로 객체의 타입을 정의할 때 쓰는 <strong>type</strong> (Type Alias) 과 <strong>interface</strong> 입니다.</p>
<pre><code class="language-typescript">// 이렇게 써도 되고...
interface User {
  name: string;
}

// 이렇게 써도 똑같이 동작하는데?
type User = {
  name: string;
};
</code></pre>
<p>&quot;기능이 똑같은데 왜 두 개나 있지?&quot; 싶으셨죠?
하지만 깊게 파고들면 <strong>확장성</strong> 과 <strong>사용 목적</strong> 에서 분명한 차이가 있습니다. 오늘은 이 둘의 차이점을 명확히 짚어드리고, 상황별로 무엇을 쓰는 게 좋은지 정리해 드립니다.</p>
<hr>
<h3 id="1-interface-객체-를-위한-설계도">1. Interface: &quot;객체&quot; 를 위한 설계도</h3>
<p><strong>interface</strong> 는 이름 그대로 객체(Object)의 구조를 정의하는 데 특화되어 있습니다. 객체 지향 프로그래밍(OOP)에서 온 개념이죠.</p>
<h4 id="✨-가장-큰-특징-선언-병합-declaration-merging">✨ 가장 큰 특징: 선언 병합 (Declaration Merging)</h4>
<p>이게 <strong>interface</strong> 의 가장 강력한 무기이자, <strong>type</strong> 과의 결정적인 차이입니다.
똑같은 이름으로 <strong>interface</strong> 를 두 번 선언하면, TypeScript가 알아서 <strong>하나로 합쳐줍니다.</strong></p>
<pre><code class="language-typescript">interface User {
  name: string;
}

interface User {
  age: number;
}

// 결과: User는 name과 age를 모두 가져야 합니다. (자동 병합됨)
const player: User = {
  name: &quot;재호&quot;,
  age: 30
};
</code></pre>
<p>이 기능 덕분에 라이브러리의 타입을 내가 원하는 대로 확장할 때 매우 유용합니다.</p>
<hr>
<h3 id="2-type-별명-붙이기-모든-타입을-위한-별명">2. Type: &quot;별명&quot; 붙이기 (모든 타입을 위한 별명)</h3>
<p><strong>Type Alias</strong> (타입 별칭)는 객체뿐만 아니라, 모든 타입에 이름을 붙일 수 있습니다. 훨씬 유연하죠.</p>
<h4 id="✨-interface가-못하는-것들">✨ Interface가 못하는 것들</h4>
<p><strong>1. 유니언 타입 (Union Type)</strong> : &quot;이거 아니면 저거&quot;</p>
<pre><code class="language-typescript">type ID = string | number; // 문자열이거나 숫자일 수 있음
</code></pre>
<p><strong>2. 튜플 (Tuple)</strong> : &quot;순서와 개수가 정해진 배열&quot;</p>
<pre><code class="language-typescript">type Coordinate = [number, number]; // [x좌표, y좌표]
</code></pre>
<p><strong>3. 원시 값 (Primitive)</strong> : 그냥 스트링에 이름 붙이기</p>
<pre><code class="language-typescript">type Name = string;
</code></pre>
<p>하지만 <strong>type</strong> 은 선언 병합이 안 됩니다. 같은 이름으로 두 번 선언하면 에러가 납니다.</p>
<hr>
<h3 id="3-정리-그래서-언제-뭘-써야-해">3. 정리: 그래서 언제 뭘 써야 해?</h3>
<p>TypeScript 공식 문서를 포함한 많은 가이드에서는 다음과 같은 <strong>Heuristic(경험 법칙)</strong> 을 추천합니다.</p>
<h4 id="✅-interface를-쓰세요-default">✅ Interface를 쓰세요 (Default)</h4>
<ul>
<li><strong>객체(Object)의 타입</strong> 을 정의할 때</li>
<li><strong>API 응답 데이터</strong> 의 형식을 정의할 때</li>
<li><strong>라이브러리</strong> 를 만들어서 배포할 때 (사용자가 확장하기 좋게)</li>
<li><strong>클래스(Class)</strong> 의 구조를 정의할 때 (implements)</li>
</ul>
<h4 id="✅-type을-쓰세요-special">✅ Type을 쓰세요 (Special)</h4>
<ul>
<li><strong>객체가 아닌 것</strong> (단순 문자열, 숫자 등)에 이름을 붙일 때</li>
<li><strong>Union 타입</strong> (<code>A | B</code>) 이나 <strong>Intersection 타입</strong> (<code>A &amp; B</code>) 이 필요할 때</li>
<li><strong>Tuple</strong> (<code>[string, number]</code>) 을 정의할 때</li>
<li><strong>함수 타입</strong> 을 간단하게 정의할 때 (<code>type Callback = () =&gt; void</code>)</li>
</ul>
<h3 id="마무리">마무리</h3>
<p><strong>&quot;확장이 필요하면 interface, 복잡한 조합이 필요하면 type&quot;</strong>
이 한 문장만 기억하시면 됩니다. 하지만 프로젝트의 일관성이 가장 중요하니, 팀원들과 상의해서 하나의 스타일을 정하는 것이 베스트입니다!</p>
