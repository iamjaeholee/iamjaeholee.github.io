---layout: posttitle: "[Network/Security] The Ultimate Guide to IPsec for Developers: Protocols, Modes, and Packet Structure"date: 2026-01-29 06:08:13 +0000
categories: [Velog]---<p>If you are studying AWS networking or preparing for a backend engineering interview, you will inevitably encounter the term <strong>IPsec (Internet Protocol Security)</strong>. It is also a favorite topic for technical interviewers.</p>
<p>*&quot;What is the relationship between VPN and IPsec?&quot;*
*&quot;What is the difference between Tunnel Mode and Transport Mode?&quot;*</p>
<p>Today, let&#39;s skip the abstract textbook definitions. Instead, we will break down the working principles of IPsec using <strong>intuitive analogies (Packaging &amp; Delivery)</strong> and analyze the <strong>actual packet structure</strong> from a developer&#39;s perspective.</p>
<hr>
<h2 id="1-concept-vpn-is-the-interface-ipsec-is-the-implementation">1. Concept: VPN is the &#39;Interface&#39;, IPsec is the &#39;Implementation&#39;</h2>
<p>If you are a developer (e.g., Java, TS), this analogy will make perfect sense.</p>
<ul>
<li><strong>VPN (Virtual Private Network):</strong> <strong><code>Interface</code></strong></li>
<li>It represents the <strong>Goal (Abstraction)</strong>: &quot;I want to build a private network over the public internet.&quot;</li>
</ul>
<ul>
<li><strong>IPsec:</strong> <strong><code>Class</code> (Implementation)</strong></li>
<li>It is the <strong>Implementation</strong> of that goal using standard protocols at the <strong>OSI Layer 3 (Network Layer)</strong>.</li>
<li><em>(Note: SSL VPN and WireGuard are other implementations of the VPN interface.)</em></li>
</ul>
<p>In short, studying IPsec means learning <strong>&quot;how the OS kernel manipulates and packages packets to achieve the VPN goal.&quot;</strong></p>
<hr>
<h2 id="2-choice-1-the-container-protocol---ah-vs-esp">2. Choice #1: The Container (Protocol - AH vs. ESP)</h2>
<p>The first step is deciding what kind of &quot;security container&quot; to put your data (Payload) into.</p>
<h3 id="✉️-ah-authentication-header-the-transparent-envelope">✉️ AH (Authentication Header): &quot;The Transparent Envelope&quot;</h3>
<ul>
<li><strong>Analogy:</strong> You put your letter in a <strong>transparent vinyl envelope</strong> and seal it with a <strong>wax stamp (Authentication)</strong>.</li>
<li><strong>Features:</strong> You can verify who sent it and that it hasn&#39;t been tampered with (Integrity), but <strong>anyone can read the content</strong> because it is transparent. (No Encryption).</li>
<li><strong>Verdict:</strong> Rarely used in modern production environments. You wouldn&#39;t mail a password in a clear envelope, right?</li>
</ul>
<h3 id="🧳-esp-encapsulating-security-payload-the-iron-safe">🧳 ESP (Encapsulating Security Payload): &quot;The Iron Safe&quot;</h3>
<ul>
<li><strong>Analogy:</strong> You put your data inside a sturdy <strong>Iron Safe</strong> and lock it.</li>
<li><strong>Features:</strong> The data is <strong>Encrypted</strong>, making it unreadable to outsiders. It also provides Authentication and Integrity.</li>
<li><strong>Verdict:</strong> <strong>The Industry Standard.</strong> When we say &quot;IPsec VPN,&quot; we almost always mean using ESP.</li>
</ul>
<hr>
<h2 id="3-choice-2-the-delivery-method-transport-vs-tunnel">3. Choice #2: The Delivery Method (Transport vs. Tunnel)</h2>
<p>Now that your data is inside the Safe (ESP), how will you ship it? This is the most crucial part.</p>
<h3 id="🚗-transport-mode-the-locked-briefcase-delivery">🚗 Transport Mode: &quot;The Locked Briefcase Delivery&quot;</h3>
<ul>
<li><strong>Method:</strong> You slap the <strong>shipping label (Destination IP)</strong> directly onto the handle of the Safe (Encrypted Data).</li>
<li><strong>Structure:</strong> <code>[Original IP] + [ESP Header] + [Encrypted Data]</code></li>
<li><strong>Downside:</strong> The content is secure, but the delivery guy (Hacker/Router) can see exactly <strong>who is talking to whom</strong>. Your traffic pattern is exposed.</li>
<li><strong>Use Case:</strong> Mostly used for <strong>Host-to-Host</strong> communication (e.g., Server to Server).</li>
</ul>
<h3 id="🚛-tunnel-mode-the-double-box-packaging">🚛 Tunnel Mode: &quot;The Double Box Packaging&quot;</h3>
<ul>
<li><strong>Method:</strong> You take the Safe and put it inside a <strong>larger, plain cardboard box</strong>. You then stick the label <strong>only on the outer box</strong>.</li>
<li><strong>Outer Label:</strong> &quot;To: Logistics Center (VPN Gateway)&quot;</li>
</ul>
<ul>
<li><strong>Structure:</strong> <code>[New IP] + [ESP Header] + [Encrypted Original IP + Data]</code></li>
<li><strong>Upside:</strong> The delivery guy only knows this box is going to the Logistics Center. He has <strong>no idea</strong> who the final recipient is inside the box.</li>
<li><strong>Use Case:</strong> Essential for <strong>Site-to-Site</strong> connections (e.g., Corporate HQ to Branch Office). It completely hides the internal IP topology.</li>
</ul>
<hr>
<h2 id="4-deep-dive-packet-anatomy">4. [Deep Dive] Packet Anatomy</h2>
<p>As developers, we trust code and data structures. Here is how the actual packet bytes transform.
This example uses the standard <strong>[ESP + Tunnel Mode]</strong> combination.</p>
<h4 id="step-1-original-packet-plain-text">Step 1: Original Packet (Plain Text)</h4>
<pre><code class="language-text">[ Original IP Header ] + [ TCP Header ] + [ Data: &quot;Hello&quot; ]
</code></pre>
<ul>
<li>If a hacker sniffs this, they can read &quot;Hello&quot; and see your internal IP.</li>
</ul>
<h4 id="step-2-after-ipsec-encapsulation-encrypted">Step 2: After IPsec Encapsulation (Encrypted)</h4>
<p>The OS Kernel wraps the entire Layer 3 packet inside the &quot;Outer Box.&quot;</p>
<pre><code class="language-text">[ New IP Header ]           &lt;-- 1. 겉박스 송장 (출발: VPN게이트웨이 / 도착: 상대방게이트웨이)
[ ESP Header ]              &lt;-- 2. 금고 식별번호 (SPI, Sequence Number)
[ *********************** ]
[ * [ Original IP ]     * ] &lt;-- 3. 암호화된 영역 시작
[ * [ TCP Header ]      * ]     (원래 주소와 데이터가 모두 암호문으로 변했습니다)
[ * [ Data: &quot;Hello&quot; ]   * ]     (해커는 이 안을 절대 볼 수 없습니다)
[ *********************** ]
[ ESP Trailer &amp; Auth ]      &lt;-- 4. 봉인 도장 (HMAC)
</code></pre>
<p><strong>Key Takeaways:</strong></p>
<ol>
<li><strong>New IP Header:</strong> Internet routers only look at this outer header to route the packet.</li>
<li><strong>Encrypted Payload:</strong> Since the <code>Original IP</code> is encrypted, the final destination is hidden from the public internet.</li>
</ol>
<hr>
<h2 id="5-faq-myths-and-truths">5. FAQ: Myths and Truths</h2>
<h3 id="q1-i-already-use-https-do-i-need-ipsec-vpn">Q1. I already use HTTPS. Do I need IPsec VPN?</h3>
<p><strong>A. Yes, using both is better (Defense in Depth).</strong>
HTTPS protects the <strong>Content (Layer 7)</strong>. IPsec protects the <strong>Path (Layer 3)</strong>.
When combined, you get a structure like <code>[ IPsec [ HTTPS [ Data ] ] ]</code>. Even if the VPN administrator inspects the packet, they cannot see your HTTPS data. This enables a <strong>Zero Trust</strong> architecture.</p>
<h3 id="q2-is-ipsec-software-or-hardware">Q2. Is IPsec Software or Hardware?</h3>
<p><strong>A. IPsec itself is a Protocol (Software).</strong>
However, enterprise &quot;VPN Appliances&quot; (like Cisco or Fortinet gear) are essentially Linux computers equipped with <strong>ASIC (Application-Specific Integrated Circuit) chips</strong>. These chips offload the heavy math of encryption (AES-256) from the CPU, allowing for high-speed throughput.</p>
<hr>
<h2 id="6-conclusion">6. Conclusion</h2>
<p>IPsec is not just a security tool; it is akin to <strong>civil engineering for the internet</strong>. It builds a private, secure highway on top of the public, chaotic road network.</p>
<p>The next time you configure an <code>AWS Site-to-Site VPN</code> or analyze packet logs, visualize this image: <strong>&quot;My data is locked in an Iron Safe (ESP), placed inside a Plain Cardboard Box (Tunnel Mode), and shipped across the internet.&quot;</strong></p>
<p>It makes the concept much easier to grasp.</p>
