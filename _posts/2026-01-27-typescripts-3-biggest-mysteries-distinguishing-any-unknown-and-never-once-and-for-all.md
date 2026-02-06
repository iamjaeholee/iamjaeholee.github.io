---layout: post
title: "TypeScript's 3 Biggest Mysteries: Distinguishing `any`, `unknown`, and `never` Once and for All"
date: 2026-01-27 09:53:08 +0000
categories: [Velog]
original_url: https://velog.io/@iamjaeholee/TypeScripts-3-Biggest-Mysteries-Distinguishing-any-unknown-and-never-Once-and-for-All-lagw6lur
---
<p>When studying TypeScript, there are three keywords that you encounter most often but find hardest to explain clearly: <code>any</code>, <code>unknown</code>, and <code>never</code>.</p>
<p>&quot;Can&#39;t I just use <code>any</code> for everything? It&#39;s so convenient.&quot;
&quot;If a function returns nothing, isn&#39;t that <code>void</code>? What on earth is <code>never</code>?&quot;</p>
<p>Today, I will clarify the subtle differences between these three types using [real-life analogies] that make them easy to understand.</p>
<hr>
<h2 id="1-any-the-counterfeit-free-pass-the-outlaw">1. any: [The Counterfeit Free Pass] (The Outlaw)</h2>
<p>Imagine airport security. Everyone is getting their bags checked, but suddenly, someone walks up wearing a &#39;Free Pass&#39; badge. The security officer doesn&#39;t check this person at all. Whether their bag contains water or a bomb, they are just waved through. It might seem fast and convenient at the moment, but the second this person gets on the plane, a disaster could occur.</p>
<p>In TypeScript, <code>any</code> is this [Counterfeit Free Pass].</p>
<p>Using <code>any</code> is like telling the compiler (the security officer), &quot;Don&#39;t care about what I do; I&#39;ll handle it.&quot; Because it completely disables type checking, the code might look fine before execution, but the program will crash immediately during actual execution (runtime).</p>
<pre><code class="language-typescript">// 1. Ignoring the security officer (compiler).
let myValue: any = 10; // I clearly assigned the number 10.

// 2. Issuing a text-only command to a number. (Check passed)
// &quot;Turn this number into uppercase&quot; -&gt; Nonsense command, but no error occurs.
myValue.toUpperCase(); 

// 3. Result: Immediate crash upon execution (Run-time Error)
// &quot;Uncaught TypeError: myValue.toUpperCase is not a function&quot;
</code></pre>
<p>[Conclusion] Unless you are in an &#39;unavoidable situation&#39; like hurriedly migrating legacy JavaScript code, do not use <code>any</code>. Do it to save your future self from late-night debugging.</p>
<hr>
<h2 id="2-unknown-the-locked-mystery-box-safety-first">2. unknown: [The Locked Mystery Box] (Safety First)</h2>
<p>This time, imagine there is a &#39;delivery box&#39; in front of your house. The label is torn off, so you have absolutely no idea what is inside (<code>unknown</code>).</p>
<p>In this situation, would you just blindly put the contents into your mouth? Absolutely not. You must open the box first (check) to see if it&#39;s an apple (eat it) or a shirt (wear it). You cannot do anything until you confirm what it is.</p>
<p><code>unknown</code> is exactly this [Locked Mystery Box]. Like <code>any</code>, it can accept any value, but to use that value, you must go through a <strong>&quot;process of confirming what&#39;s inside.&quot;</strong> It is much safer.</p>
<pre><code class="language-typescript">// It&#39;s a box that can hold anything (unknown).
let giftBox: unknown = &quot;Apple&quot;;

// X Error Occurs! 
// TypeScript blocks you if you try to measure length without opening (checking) the box.
// console.log(giftBox.length); // Error: Object is of type &#39;unknown&#39;.

// O Correct Usage: Use it after opening the box (Type Check)
if (typeof giftBox === &#39;string&#39;) {
  // Inside this block, it is confirmed to be a &#39;string&#39;, so it&#39;s safe to use!
  console.log(giftBox.length); 
}
</code></pre>
<p>[Conclusion] If you are in a situation where you &#39;cannot be sure what data will come in&#39; (like receiving data from an external API), use <code>unknown</code> instead of <code>any</code>. Then, writing validation (Type Guard) code is the standard practice.</p>
<hr>
<h2 id="3-never-the-black-hole--dead-end-impossible-to-exist">3. never: [The Black Hole &amp; Dead End] (Impossible to Exist)</h2>
<p>This is the most confusing concept. Many people ask, &quot;If there is no value, isn&#39;t that <code>void</code>?&quot; There is a crucial difference here.</p>
<ul>
<li><code>void</code> (Leaving Empty-handed): An employee finishes work and goes home. They leave with nothing in their hands (no return value), but they do go home, eat dinner, and sleep. It means <strong>&quot;The task is finished.&quot;</strong></li>
<li><code>never</code> (Missing Person): An employee went to work but got sucked into a black hole or is trapped working forever. They cannot go home. A &#39;next schedule&#39; cannot exist for them. It means <strong>&quot;The task never finishes.&quot;</strong></li>
</ul>
<p>In code, this difference is distinguished by: [Can the line of code immediately following the function be executed?]</p>
<pre><code class="language-typescript">// Case 1: void (Normal Termination)
function sayHello(): void {
  console.log(&quot;Hello&quot;);
  // The function finishes and exits. 
  // The code on the next line WILL be executed.
}

// Case 2: never (Abnormal Termination / Black Hole)
function crashGame(): never {
  throw new Error(&quot;Server Exploded!&quot;); 
  // The program crashes here, or the flow is hijacked.
  // The code below this line can NEVER be reached. (Unreachable Code)
}
</code></pre>
<h3 id="the-real-way-to-use-never-proof-of-perfection">The Real Way to Use &#39;never&#39;: [Proof of Perfection]</h3>
<p>The real reason developers use <code>never</code> in production is to monitor <strong>&quot;Did I miss anything?&quot;</strong></p>
<p>Let&#39;s take a sandwich shop example. If the menu only consists of &#39;Ham&#39; and &#39;Cheese&#39;, the probability of receiving an order other than these two must be 0%, which is <code>never</code>.</p>
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
<p>But what if the owner adds a <strong>&#39;Veggie&#39;</strong> menu, and the developer forgets to update this function?</p>
<p>A <code>Veggie</code> order comes in, but there is no logic to handle it, so it flows into the final <code>else</code> block. It entered a place that should never be visited (<code>never</code>). At this moment, TypeScript throws a red line (error) saying, <strong>&quot;Hey, you said no one could come here! What is this Veggie thing?&quot;</strong></p>
<p>In this way, <code>never</code> serves as a [Powerful Safety Mechanism] that completely blocks logical holes in your code.</p>
<hr>
<h2 id="summary-at-a-glance">Summary at a Glance</h2>
<table>
<thead>
<tr>
<th>Type</th>
<th>Real-life Metaphor</th>
<th>Developer Meaning</th>
<th>Safety</th>
</tr>
</thead>
<tbody><tr>
<td><code>any</code></td>
<td>Counterfeit Free Pass</td>
<td>Turn off Type Checking (Avoid usage)</td>
<td>Lowest (Dangerous)</td>
</tr>
<tr>
<td><code>unknown</code></td>
<td>Locked Mystery Box</td>
<td>Cannot use before check (Safe Unknown)</td>
<td>High (Safe)</td>
</tr>
<tr>
<td><code>never</code></td>
<td>Black Hole / Dead End</td>
<td>Impossible state, does not terminate</td>
<td>Highest (For Logic Validation)</td>
</tr>
</tbody></table>
<p>[Final Thoughts]</p>
<p>If you want to code comfortably and carelessly, you might use <code>any</code>, but &#39;bugs&#39; will pay the price.
If you want to accept data but verify it safely, use <code>unknown</code>.
If you want to completely block logical holes in your code, actively utilize <code>never</code>.</p>
<p>Your code will become much more robust.</p>
