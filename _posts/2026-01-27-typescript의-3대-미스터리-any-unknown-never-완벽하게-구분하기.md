---layout: post
title: "TypeScript의 3대 미스터리: any, unknown, never 완벽하게 구분하기"
date: 2026-01-27 09:50:13 +0000
categories: [Velog]
original_url: https://velog.io/@iamjaeholee/TypeScript%EC%9D%98-3%EB%8C%80-%EB%AF%B8%EC%8A%A4%ED%84%B0%EB%A6%AC-any-unknown-never-%EC%99%84%EB%B2%BD%ED%95%98%EA%B2%8C-%EA%B5%AC%EB%B6%84%ED%95%98%EA%B8%B0
---
<p>타입스크립트(TypeScript)를 공부하다 보면 가장 흔하게 마주치지만, 막상 설명하려면 말문이 막히는 세 가지 키워드가 있습니다. 바로 <code>any</code>, <code>unknown</code>, 그리고 <code>never</code>입니다.</p>
<p>&quot;그냥 다 <code>any</code> 쓰면 안 되나? 편하던데.&quot;
&quot;값이 없다는 건 <code>void</code> 아닌가? <code>never</code>는 또 뭐야?&quot;</p>
<p>오늘은 이 세 가지 타입의 미묘한 차이를, 우리가 일상에서 겪는 상황에 빗대어 명쾌하게 정리해 드립니다.</p>
<hr>
<h2 id="1-any-위조된-만능-프리패스-무법자">1. any: [위조된 만능 프리패스] (무법자)</h2>
<p>공항 보안 검색대를 상상해 보세요. 모두가 짐 검사를 받고 있는데, 어떤 사람이 &#39;프리패스&#39; 목걸이를 걸고 나타납니다. 보안 요원은 이 사람을 검사하지 않습니다. 가방에 물이 들었든, 폭탄이 들었든 그냥 통과시켜 줍니다. 당장은 편하고 빠르겠죠? 하지만 이 사람이 비행기에 타는 순간 대재앙이 시작될 수 있습니다.</p>
<p>타입스크립트의 <code>any</code>가 바로 이 [위조된 프리패스]입니다.</p>
<p><code>any</code>를 쓰면 컴파일러(보안 요원)에게 &quot;내가 뭘 하든 신경 쓰지 마, 내가 알아서 할게&quot;라고 입을 막아버리는 것과 같습니다. 타입 검사를 아예 포기하기 때문에, 실행 전에는 문제가 없는 것처럼 보이다가 실제 실행(런타임) 시에 프로그램이 터져버립니다.</p>
<pre><code class="language-typescript">// 1. 보안 요원(컴파일러)을 무시합니다.
let myValue: any = 10; // 분명 숫자 10을 넣었습니다.

// 2. 숫자에다가 글자 전용 명령을 내려도 막지 않습니다. (검사 패스)
// &quot;숫자를 대문자로 바꿔라&quot; -&gt; 말도 안 되는 명령이지만 에러가 안 납니다.
myValue.toUpperCase(); 

// 3. 결과: 프로그램 실행 즉시 사망 (Run-time Error)
// &quot;Uncaught TypeError: myValue.toUpperCase is not a function&quot;
</code></pre>
<p>[결론] 기존 자바스크립트 코드를 급하게 옮기는 등 &#39;어쩔 수 없는 상황&#39;이 아니라면, <code>any</code>는 쓰지 마세요. 여러분의 퇴근 시간을 지키기 위해서요.</p>
<hr>
<h2 id="2-unknown-내용물을-모르는-택배-상자-안전-제일">2. unknown: [내용물을 모르는 택배 상자] (안전 제일)</h2>
<p>이번에는 집 앞에 &#39;택배 상자&#39;가 하나 놓여 있다고 가정해 봅시다. 송장이 뜯겨 있어서 안에 뭐가 들었는지 도무지 알 수가 없습니다(unknown).</p>
<p>이때, 여러분은 상자를 뜯지도 않고 내용물을 입으로 가져가서 먹나요? 절대 그러지 않죠. 일단 상자를 열어서(검사), 사과라면 먹고 옷이라면 입어야 합니다. 확인하기 전까진 아무것도 할 수 없습니다.</p>
<p><code>unknown</code>은 바로 이런 [잠긴 상자]입니다. <code>any</code>처럼 무엇이든 받을 수는 있지만, 그 값을 사용하려면 반드시 &quot;안에 뭐가 들었는지 확인하는 절차&quot;를 거쳐야 합니다. 훨씬 안전하죠.</p>
<pre><code class="language-typescript">// 일단 무엇이든 담을 수 있는 상자(unknown)입니다.
let giftBox: unknown = &quot;사과&quot;;

// X 에러 발생! 
// 상자를 안 열어보고(검사 없이) 길이를 재려고 하면 타입스크립트가 막습니다.
// console.log(giftBox.length); // Error: Object is of type &#39;unknown&#39;.

// O 올바른 사용법: 상자를 열어보고(타입 검사) 나서 사용
if (typeof giftBox === &#39;string&#39;) {
  // 이 블록 안에서는 &#39;문자열&#39;인 게 확인되었으므로 안전하게 사용 가능!
  console.log(giftBox.length); 
}
</code></pre>
<p>[결론] 외부 API에서 데이터를 받아오는 것처럼 &#39;뭐가 들어올지 확신할 수 없는 상황&#39;이라면, <code>any</code> 대신 무조건 <code>unknown</code>을 쓰세요. 그리고 나서 검증(Type Guard) 코드를 작성하는 것이 정석입니다.</p>
<hr>
<h2 id="3-never-블랙홀과-막다른-골목-존재-불가능">3. never: [블랙홀과 막다른 골목] (존재 불가능)</h2>
<p>가장 헷갈리는 개념입니다. 많은 분들이 &quot;값이 없으면 <code>void</code> 아닌가?&quot;라고 생각하시죠. 여기서 중요한 차이가 있습니다.</p>
<ul>
<li><code>void</code> (빈손 퇴근): 직장인이 일을 다 마치고 퇴근합니다. 손에 들고 가는 건 없지만(return값 없음), 어쨌든 집에 가서 밥도 먹고 잠도 잡니다. &quot;끝이 났다&quot;는 뜻입니다.</li>
<li><code>never</code> (행방불명): 직장인이 출근을 했는데, 블랙홀에 빨려 들어갔거나 24시간 갇혀서 영원히 일만 하고 있습니다. 퇴근을 못 합니다. 그 다음 스케줄이란 존재할 수 없죠. &quot;끝나지 않는다&quot;는 뜻입니다.</li>
</ul>
<p>코드로 보면 이 차이는 [함수를 실행한 바로 다음 줄이 실행될 수 있는가?]로 구분됩니다.</p>
<pre><code class="language-typescript">// Case 1: void (정상 종료)
function sayHello(): void {
  console.log(&quot;안녕&quot;);
  // 함수가 끝나면 밖으로 나갑니다. 
  // 다음 줄 코드가 실행됩니다.
}

// Case 2: never (비정상 종료 / 블랙홀)
function crashGame(): never {
  throw new Error(&quot;서버 폭발!&quot;); 
  // 여기서 프로그램이 터지거나 흐름이 납치당합니다.
  // 이 밑으로는 절대 내려가지 못합니다. (Unreachable Code)
}
</code></pre>
<h3 id="never를-써먹는-진짜-방법-완벽함의-증명">never를 써먹는 진짜 방법: [완벽함의 증명]</h3>
<p>개발자가 실무에서 <code>never</code>를 쓰는 진짜 이유는 &quot;혹시 내가 빼먹은 게 없나?&quot;를 감시하기 위해서입니다.</p>
<p>샌드위치 가게를 예로 들어볼까요? 메뉴가 &#39;햄&#39;과 &#39;치즈&#39;뿐이라면, 이 둘을 제외한 다른 주문이 들어올 확률은 0%, 즉 <code>never</code>여야 합니다.</p>
<pre><code class="language-typescript">type Sandwich = &#39;Ham&#39; | &#39;Cheese&#39;;

function makeSandwich(menu: Sandwich) {
  if (menu === &#39;Ham&#39;) {
    // 햄 샌드위치 만들기
  } else if (menu === &#39;Cheese&#39;) {
    // 치즈 샌드위치 만들기
  } else {
    // 햄도 아니고 치즈도 아닌 주문? 그런 건 존재할 수 없습니다.
    // 이곳의 타입은 논리적으로 [never]가 됩니다.
    const _check: never = menu; 
  }
}
</code></pre>
<p>그런데 만약 사장님이 &#39;야채(Veggie)&#39; 메뉴를 추가했는데, 개발자가 이 함수를 수정하는 걸 깜빡했다면?</p>
<p><code>Veggie</code> 주문이 들어왔는데 처리하는 로직이 없으니 마지막 <code>else</code> 블록으로 흘러들어갑니다. 원래는 절대 오면 안 되는(<code>never</code>) 곳인데 손님이 와버린 거죠. 이때 타입스크립트는 &quot;야, 여기 절대 못 온다며? 야채는 뭔데?&quot; 라며 빨간 줄(에러)을 띄워줍니다.</p>
<p>이처럼 <code>never</code>는 코드의 빈틈을 원천 봉쇄하는 [강력한 안전장치] 역할을 합니다.</p>
<hr>
<h2 id="한-눈에-정리하기">한 눈에 정리하기</h2>
<table>
<thead>
<tr>
<th>타입</th>
<th>일상 비유</th>
<th>개발 의미</th>
<th>안전성</th>
</tr>
</thead>
<tbody><tr>
<td><code>any</code></td>
<td>위조된 프리패스</td>
<td>타입 검사 끄기 (사용 지양)</td>
<td>최하 (위험)</td>
</tr>
<tr>
<td><code>unknown</code></td>
<td>잠긴 택배 상자</td>
<td>검사 전 사용 불가 (안전한 미지)</td>
<td>높음 (안전)</td>
</tr>
<tr>
<td><code>never</code></td>
<td>블랙홀 / 막다른 길</td>
<td>불가능한 상태, 종료되지 않음</td>
<td>최상 (로직 검증용)</td>
</tr>
</tbody></table>
<p>[마치며]</p>
<p>편하게 막 코딩하고 싶다면 <code>any</code>를 쓰겠지만, 그 대가는 &#39;버그&#39;가 치릅니다.
일단 받고 안전하게 확인하고 싶다면 <code>unknown</code>을,
코드의 논리적 구멍을 완벽하게 막고 싶다면 <code>never</code>를 적극적으로 활용해 보세요.</p>
<p>여러분의 코드가 훨씬 더 단단해질 것입니다.</p>
