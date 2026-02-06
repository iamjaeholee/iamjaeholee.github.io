---layout: post
title: "[TypeScript] Type vs. Interface: The Definitive Guide (Based on Official Docs)"
date: 2026-01-25 18:30:24 +0000
categories: [Velog]
original_url: https://velog.io/@iamjaeholee/TypeScript-Type-vs.-Interface-The-Definitive-Guide-Based-on-Official-Docs
---
<h4 id="1-introduction-the-eternal-dilemma">1. Introduction: The Eternal Dilemma</h4>
<p>If you are studying TypeScript, you have likely faced this question early on:
*&quot;Should I use <code>type</code> or <code>interface</code> to define my object types?&quot;*</p>
<p>They look very similar, and in 99% of cases, they function identically. However, the <strong>TypeScript Official Handbook</strong> clearly distinguishes between the two and provides recommendations based on specific scenarios.</p>
<p>Today, I will clarify the key differences based on the official documentation and give you a clear standard on when to use which.</p>
<h4 id="2-tldr-the-official-verdict">2. TL;DR: The Official Verdict</h4>
<p>For those in a hurry, here is the conclusion first. The official documentation suggests the following heuristic:</p>
<blockquote>
<p><strong>&quot;Use <code>interface</code> until you need to use features from <code>type</code>.&quot;</strong></p>
</blockquote>
<p>Why? The biggest reason is <strong>extensibility</strong>. In libraries or large-scale projects, you often need to augment types later on. Interfaces handle this much more gracefully than types.</p>
<h4 id="3-the-biggest-difference-declaration-merging">3. The Biggest Difference: Declaration Merging</h4>
<p>The most decisive difference lies in <strong>how they handle adding new properties</strong>.</p>
<p><strong>Interface: Supports Automatic Merging</strong></p>
<p>You can declare an <code>interface</code> with the same name multiple times. TypeScript will automatically merge them into a single interface. This is called <strong>Declaration Merging</strong>.</p>
<pre><code class="language-typescript">// First declaration
interface User {
  name: string;
}

// Second declaration (Same name)
interface User {
  age: number;
}

// Result: The User interface now has both &#39;name&#39; and &#39;age&#39;
const jinyoung: User = {
  name: &quot;jinyoung&quot;,
  age: 30
};
</code></pre>
<p>Thanks to this feature, it is very convenient to extend external library types to fit your specific needs.</p>
<p><strong>Type: Immutable (Cannot Change)</strong></p>
<p>On the other hand, a <code>Type Alias</code> is finished once declared. If you try to declare it again with the same name, it throws an error immediately.</p>
<pre><code class="language-typescript">type User = {
  name: string;
};

// Error: Duplicate identifier &#39;User&#39;
type User = {
  age: number;
};
</code></pre>
<p>In short, think of <strong>Type as a closed structure</strong> and <strong>Interface as an open structure</strong>.</p>
<h4 id="4-difference-in-extension-syntax-extends-vs-intersection">4. Difference in Extension Syntax (Extends vs. Intersection)</h4>
<p>The syntax for creating a new type based on an existing one is also different.</p>
<p><strong>Interface: The OOP style `extends</strong>`</p>
<pre><code class="language-typescript">interface Animal {
  name: string;
}

interface Bear extends Animal {
  honey: boolean;
}
</code></pre>
<p>This uses the inheritance concept and syntax familiar to Object-Oriented Programming (OOP). It is readable and clear.</p>
<p><strong>Type: Intersection via `&amp;</strong>`</p>
<pre><code class="language-typescript">type Animal = {
  name: string;
};

type Bear = Animal &amp; { 
  honey: boolean; 
};
</code></pre>
<p><code>Type</code> uses the ampersand (<code>&amp;</code>) symbol to combine two types (Intersection Type).</p>
<h4 id="5-so-when-should-you-use-type">5. So, When Should You Use <code>Type</code>?</h4>
<p><code>Interface</code> can only be used to define the shape of an <strong>object</strong>. <code>Type</code>, however, is much more versatile.</p>
<p>You <strong>must</strong> use <code>Type</code> for things that an Interface simply cannot do:</p>
<ul>
<li><strong>Primitive Aliases</strong> (<code>type Name = string;</code>)</li>
<li><strong>Union Types</strong> (<code>type ID = string | number;</code>)</li>
<li><strong>Tuples</strong> (<code>type Pair = [string, number];</code>)</li>
</ul>
<h4 id="6-conclusion--summary">6. Conclusion &amp; Summary</h4>
<p>To wrap it up:</p>
<ul>
<li><strong>[Default Strategy]</strong> Use <strong>Interface</strong> for defining most object types. (It offers better extensibility and readability).</li>
<li><strong>[Exceptions]</strong> Use <strong>Type</strong> if you need to define primitive aliases, Union types, or Tuples.</li>
<li><strong>[For Library Developers]</strong> You should almost always provide <strong>Interfaces</strong> so that users can extend them via declaration merging.</li>
</ul>
<p>Next time you start a project, don&#39;t hesitate—just start with an <code>interface</code>!</p>
