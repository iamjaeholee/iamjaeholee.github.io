---layout: posttitle: "[Styled-components] TypeScript + Storybook으로 완벽한 디자인 시스템 구축하기"date: 2026-01-17 10:04:54 +0000
categories: [Velog]---<p>모던 프론트엔드 개발에서 <strong>CSS-in-JS</strong> 는 이제 선택이 아닌 필수가 되어가고 있죠. 그중에서도 가장 대중적인 <strong>Styled-components</strong> 를 많이 사용하실 텐데요.</p>
<p>하지만 JavaScript에서는 편하게 썼던 기능들이 <strong>TypeScript</strong> 로 넘어오면서 빨간 줄(에러)을 뿜어내기 시작합니다. 게다가 <strong>Storybook</strong> 까지 붙이려면 설정해야 할 게 한두 가지가 아니죠.</p>
<p>오늘은 실무에서 자주 마주치는 <strong>TypeScript 타이핑 팁</strong> 과 <strong>Storybook 연동 시 필수 설정</strong> 을 정리해 드립니다.</p>
<hr>
<h3 id="1-typescript에서-props-제대로-넘기기-transient-props">1. TypeScript에서 Props 제대로 넘기기 (Transient Props)</h3>
<p>Styled-components의 가장 큰 장점은 <code>props</code> 에 따라 스타일을 다르게 줄 수 있다는 점입니다. 하지만 TS에서는 이 <code>props</code> 의 타입을 반드시 정의해야 합니다.</p>
<h4 id="❌-초보자의-실수">❌ 초보자의 실수</h4>
<p>HTML 태그에는 우리가 만든 커스텀 속성(예: <code>isActive</code>)이 존재하지 않기 때문에, 그냥 넘기면 콘솔 경고창(Warning)이 뜹니다.</p>
<h4 id="⭕-고수들의-꿀팁-transient-props-">⭕ 고수들의 꿀팁: Transient Props ($)</h4>
<p>속성 이름 앞에 달러 기호 <code>$</code> 를 붙이면, 스타일링에만 사용하고 <strong>실제 DOM 요소에는 전달되지 않습니다</strong> . (이거 면접에서 물어보면 가산점입니다!)</p>
<pre><code class="language-typescript">import styled from &#39;styled-components&#39;;

// 1. Interface로 타입 정의
interface ButtonProps {
  $primary?: boolean; // $를 붙여서 DOM 전달 방지
}

// 2. 제네릭 문법 &lt;&gt; 으로 타입 주입
const Button = styled.button&lt; ButtonProps &gt;`
  background: ${props =&gt; (props.$primary ? &#39;blue&#39; : &#39;gray&#39;)};
  color: white;
  padding: 10px 20px;
  border-radius: 4px;
`;

// 사용 예시
&lt;Button $primary&gt;메인 버튼&lt;/Button&gt;
</code></pre>
<hr>
<h3 id="2-storybook에-styled-components-연동하기">2. Storybook에 Styled-components 연동하기</h3>
<p>Storybook을 처음 설치하고 컴포넌트를 띄워보면, 폰트가 깨지거나 우리가 설정한 <code>ThemeProvider</code> 의 색상 변수를 못 찾아서 에러가 나는 경우가 많습니다.</p>
<p>Storybook은 우리 앱과는 별개의 <strong>독립적인 실행 환경</strong> 이기 때문에, 여기서도 스타일 설정을 따로 해줘야 합니다.</p>
<h4 id="🛠️-storybookpreviewtsx-설정-decorator-패턴">🛠️ .storybook/preview.tsx 설정 (Decorator 패턴)</h4>
<p>Storybook의 모든 스토리에 공통적으로 <strong>GlobalStyle</strong> 과 <strong>Theme</strong> 을 적용하려면 <code>decorators</code> 를 사용해야 합니다.</p>
<pre><code class="language-typescript">// .storybook/preview.tsx
import type { Preview } from &#39;@storybook/react&#39;;
import { ThemeProvider } from &#39;styled-components&#39;;
import { GlobalStyle } from &#39;../src/styles/GlobalStyle&#39;; // 전역 스타일
import { theme } from &#39;../src/styles/theme&#39;; // 색상 테마

const preview: Preview = {
  // decorators: 스토리를 감싸는 래퍼(Wrapper) 컴포넌트 설정
  decorators: [
    (Story) =&gt; (
      &lt;ThemeProvider theme={theme}&gt;
        &lt;GlobalStyle /&gt;
        &lt;Story /&gt; {/* 실제 컴포넌트가 여기 들어갑니다 */}
      &lt;/ThemeProvider&gt;
    ),
  ],
  parameters: {
    actions: { argTypesRegex: &#39;^on[A-Z].*&#39; },
    controls: {
      matchers: {
        color: /(background|color)$/i,
        date: /Date$/,
      },
    },
  },
};

export default preview;
</code></pre>
<p>이렇게 설정해 두면, Storybook에서도 실제 앱과 똑같은 환경(폰트, 색상 변수 등)에서 컴포넌트를 테스트할 수 있습니다.</p>
<hr>
<h3 id="3-attrs를-활용한-성능-최적화--코드-단축">3. attrs를 활용한 성능 최적화 &amp; 코드 단축</h3>
<p>자주 변하는 스타일이나, 고정된 속성을 매번 입력하기 귀찮을 때는 <code>.attrs</code> 메서드를 활용하세요.</p>
<h4 id="사용-사례-input-타입-고정하기">사용 사례: Input 타입 고정하기</h4>
<p>매번 <code>&lt;Input type=&quot;password&quot; /&gt;</code> 라고 쓰기 귀찮다면?</p>
<pre><code class="language-typescript">// attrs를 사용해 속성을 미리 주입
const PasswordInput = styled.input.attrs({
  type: &#39;password&#39;,
  placeholder: &#39;비밀번호를 입력하세요&#39;,
})`
  border: 1px solid #ccc;
  padding: 8px;
`;
</code></pre>
<p>이렇게 하면 컴포넌트를 쓸 때 <code>&lt;PasswordInput /&gt;</code> 만 써도 자동으로 타입이 적용됩니다. 코드 중복을 줄이는 아주 좋은 습관이죠.</p>
<hr>
<h3 id="4-정리하며">4. 정리하며</h3>
<p>Styled-components와 Storybook, 그리고 TypeScript는 현대 프론트엔드 개발의 <strong>삼위일체</strong> 와도 같습니다.</p>
<ol>
<li><strong>TypeScript</strong> : <code>interface</code> 와 제네릭 <code>&lt; &gt;</code> 으로 Props 타입을 확실하게 잡는다.</li>
<li><strong>Transient Props ($)</strong> : 스타일용 Props가 DOM에 묻어나지 않게 <code>$\</code> 를 붙인다.</li>
<li><strong>Storybook Decorator</strong> : <code>preview.tsx</code> 에서 <code>ThemeProvider</code> 를 감싸줘야 에러가 안 난다.</li>
</ol>
<p>이 세 가지만 기억하시면, 디자인 시스템을 만들 때 겪는 웬만한 설정 문제는 모두 해결되실 겁니다.</p>
