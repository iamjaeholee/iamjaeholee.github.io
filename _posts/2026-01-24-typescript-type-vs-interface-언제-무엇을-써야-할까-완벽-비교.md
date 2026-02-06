---layout: post
title: "[TypeScript] type vs interface, 언제 무엇을 써야 할까? (완벽 비교)"
date: 2026-01-24 23:25:54 +0000
categories: [Velog]
original_url: https://velog.io/@iamjaeholee/TypeScript-type-vs-interface-%EC%96%B8%EC%A0%9C-%EB%AC%B4%EC%97%87%EC%9D%84-%EC%8D%A8%EC%95%BC-%ED%95%A0%EA%B9%8C-%EC%99%84%EB%B2%BD-%EB%B9%84%EA%B5%90
---
<h2 id="1-들어가며-영원한-난제-무엇을-써야-할까">1. 들어가며: 영원한 난제, 무엇을 써야 할까?</h2>
<p>타입스크립트를 공부하다 보면 가장 먼저 마주하는 고민이 있습니다. 바로 객체의 타입을 정의할 때 [Type]을 쓸 것이냐, [Interface]를 쓸 것이냐 하는 문제입니다.</p>
<p>둘은 매우 비슷해 보이고, 실제로 대부분의 상황에서는 어느 것을 써도 기능적으로 동일하게 동작합니다. 하지만 TypeScript 공식 홈페이지(Handbook)에서는 이 둘의 차이를 명확히 구분하고 있으며, 상황에 따른 권장 사항도 제시하고 있습니다.</p>
<p>오늘은 공식 문서를 기준으로 이 둘의 핵심적인 차이를 정리해 보고, 언제 무엇을 써야 할지 명확한 기준을 잡아드리겠습니다.</p>
<h2 id="2-핵심-요약-공식-문서의-결론부터">2. 핵심 요약: 공식 문서의 결론부터</h2>
<p>바쁘신 분들을 위해 결론부터 말씀드립니다. 공식 문서에서는 다음과 같은 경험 법칙(Heuristic)을 제안합니다.</p>
<blockquote>
<p>&quot;특별히 Type의 기능이 필요한 경우가 아니라면, 우선적으로 Interface를 사용하십시오.&quot;</p>
</blockquote>
<p>왜일까요? 가장 큰 이유는 [확장성] 때문입니다. 라이브러리나 규모가 큰 프로젝트에서는 나중에 타입을 보강해야 할 일이 생기는데, 이때 Interface가 훨씬 유연하게 대처할 수 있기 때문입니다.</p>
<h2 id="3-가장-큰-차이-선언-병합-declaration-merging">3. 가장 큰 차이: 선언 병합 (Declaration Merging)</h2>
<p>가장 결정적인 차이는 [새로운 속성을 추가하는 방식]에 있습니다.</p>
<p><strong>Interface: 자동 병합 가능</strong></p>
<p>Interface는 같은 이름으로 여러 번 선언할 수 있습니다. 이렇게 하면 TypeScript가 알아서 이들을 하나로 합쳐줍니다. 이를 [선언 병합]이라고 합니다.</p>
<pre><code class="language-typescript">// 첫 번째 선언
interface User {
  name: string;
}

// 두 번째 선언 (같은 이름)
interface User {
  age: number;
}

// 결과: User 인터페이스는 name과 age를 모두 가짐
const jinyoung: User = {
  name: &quot;jinyoung&quot;,
  age: 30
};
</code></pre>
<p>이 기능 덕분에 외부 라이브러리의 타입을 내가 필요한 대로 확장해서 쓰기가 매우 편리합니다.</p>
<h3 id="type-변경-불가">Type: 변경 불가</h3>
<p>반면, Type Alias는 한 번 선언되면 끝입니다. 같은 이름으로 다시 선언하려고 하면 바로 에러가 발생합니다.</p>
<pre><code class="language-typescript">type User = {
  name: string;
};

// 에러 발생: Duplicate identifier &#39;User&#39;
type User = {
  age: number;
};
</code></pre>
<p>즉, Type은 닫혀 있는 구조이고, Interface는 열려 있는 구조라고 이해하시면 됩니다.</p>
<h2 id="4-확장-문법의-차이-extends-vs-intersection">4. 확장 문법의 차이 (Extends vs Intersection)</h2>
<p>기존 타입을 바탕으로 새로운 타입을 만들 때의 문법도 다릅니다.</p>
<h3 id="interface-객체-지향적인-extends">Interface: 객체 지향적인 [extends]</h3>
<pre><code class="language-typescript">interface Animal {
  name: string;
}

interface Bear extends Animal {
  honey: boolean;
}
</code></pre>
<p>우리가 흔히 아는 상속 개념과 문법이 같습니다. 가독성이 좋고 명확합니다.</p>
<h3 id="type-교차-타입-">Type: 교차 타입 [&amp;]</h3>
<pre><code class="language-typescript">type Animal = {
  name: string;
};

type Bear = Animal &amp; {
  honey: boolean;
};
</code></pre>
<p>Type은 <code>&amp;</code> 기호를 사용해 두 타입을 합치는 방식을 사용합니다.</p>
<h2 id="5-그렇다면-type은-언제-쓰는가">5. 그렇다면 Type은 언제 쓰는가?</h2>
<p>Interface는 오직 [객체]의 모양을 정의할 때만 쓸 수 있습니다. 반면 Type은 훨씬 범용적입니다.</p>
<p>Interface로 할 수 없는 다음과 같은 것들은 Type을 써야 합니다.</p>
<ul>
<li>[원시 값]의 별칭 (type Name = string;)</li>
<li>[유니언 타입] (type ID = string | number;)</li>
<li>[튜플] (type Pair = [string, number];)</li>
</ul>
<h2 id="6-마무리-및-정리">6. 마무리 및 정리</h2>
<p>정리하자면 다음과 같습니다.</p>
<ol>
<li>[기본 전략] 대부분의 객체 타입 정의에는 Interface를 사용하세요. (확장성과 코드 가독성 면에서 유리)</li>
<li>[예외 상황] 단순한 원시 값, Union 타입, Tuple 등을 정의해야 한다면 Type을 사용하세요.</li>
<li>[라이브러리 개발 시] 사용자가 타입을 확장할 수 있도록 반드시 Interface를 제공하는 것이 좋습니다.</li>
</ol>
<p>이제 프로젝트를 시작할 때 고민하지 말고, 일단 Interface로 시작해 보는 건 어떨까요?</p>
