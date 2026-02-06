---layout: post
title: "[네트워크/보안] 개발자를 위한 IPsec 프로토콜 완벽 해부 (AH vs ESP, 패킷 구조)"
date: 2026-01-29 06:05:38 +0000
categories: [Velog]
original_url: https://velog.io/@iamjaeholee/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC%EB%B3%B4%EC%95%88-%EA%B0%9C%EB%B0%9C%EC%9E%90%EB%A5%BC-%EC%9C%84%ED%95%9C-IPsec-%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C-%EC%99%84%EB%B2%BD-%ED%95%B4%EB%B6%80-AH-vs-ESP-%ED%8C%A8%ED%82%B7-%EA%B5%AC%EC%A1%B0
---
<p>AWS나 네트워크 보안을 공부하다 보면 <strong>IPsec (Internet Protocol Security)</strong>이라는 단어를 무조건 마주치게 됩니다. 기술 면접 단골 질문이기도 하죠.</p>
<p>&quot;VPN이랑 IPsec은 무슨 관계인가요?&quot;
&quot;터널 모드는 뭐고 전송 모드는 뭔가요?&quot;</p>
<p>오늘은 뜬구름 잡는 비유는 빼고, <strong>가장 직관적인 &#39;택배 포장&#39; 예시</strong>와 <strong>개발자를 위한 패킷 구조</strong>를 통해 IPsec의 작동 원리를 완벽하게 파헤쳐 보겠습니다.</p>
<hr>
<h2 id="1-개념-정립-vpn이-인터페이스라면-ipsec은-구현체">1. 개념 정립: VPN이 &#39;인터페이스&#39;라면, IPsec은 &#39;구현체&#39;</h2>
<p>개발자(Java 등)라면 이 비유 하나로 둘의 관계가 정리됩니다.</p>
<ul>
<li><strong>VPN (Virtual Private Network)</strong>: <strong><code>Interface</code></strong></li>
<li>&quot;가상의 사설망을 구축한다&quot;는 <strong>목표(추상화)</strong>입니다.</li>
</ul>
<ul>
<li><strong>IPsec</strong>: <strong><code>Class</code> (Implementation)</strong></li>
<li>VPN이라는 목표를 달성하기 위해 <strong>OSI 3계층(Network Layer)에서 실제로 구현한 기술 표준</strong>입니다.</li>
<li><em>(참고: SSL VPN, WireGuard 등은 또 다른 구현체들입니다.)</em></li>
</ul>
<p>즉, IPsec을 공부한다는 건 <strong>&quot;VPN이라는 기능을 구현하기 위해 커널 레벨에서 패킷을 어떻게 포장하는가?&quot;</strong>를 배우는 것입니다.</p>
<hr>
<h2 id="2-ipsec의-첫-번째-선택-어떤-상자에-담을까-프로토콜">2. IPsec의 첫 번째 선택: 어떤 상자에 담을까? (프로토콜)</h2>
<p>데이터(Payload)를 어떤 보안 용기에 담을지 결정하는 단계입니다.</p>
<h3 id="✉️-ah-authentication-header-투명-비닐-봉투">✉️ AH (Authentication Header): &quot;투명 비닐 봉투&quot;</h3>
<ul>
<li><strong>비유</strong>: 내용물이 훤히 보이는 <strong>투명 봉투</strong>에 넣고, 입구에 <strong>&#39;봉인 도장(인증)&#39;</strong>만 찍은 상태입니다.</li>
<li><strong>특징</strong>: 누가 보냈는지(인증), 도장이 훼손됐는지(무결성)는 알 수 있지만, <strong>지나가는 해커가 내용을 다 읽을 수 있습니다.</strong> (암호화 X)</li>
<li><strong>결론</strong>: 비밀번호를 투명 봉투에 담을 순 없죠? 실무에서는 거의 안 씁니다.</li>
</ul>
<h3 id="🧳-esp-encapsulating-security-payload-철제-금고">🧳 ESP (Encapsulating Security Payload): &quot;철제 금고&quot;</h3>
<ul>
<li><strong>비유</strong>: 데이터를 튼튼한 <strong>철제 금고</strong>에 넣고 자물쇠를 채운 상태입니다.</li>
<li><strong>특징</strong>: 데이터를 <strong>암호화(Encryption)</strong>하여 아무도 못 보게 잠급니다. 인증과 무결성도 당연히 제공합니다.</li>
<li><strong>결론</strong>: <strong>보안의 왕</strong>입니다. 우리가 &quot;IPsec VPN을 쓴다&quot;라고 하면 99%는 이 ESP 금고를 의미합니다.</li>
</ul>
<hr>
<h2 id="3-ipsec의-두-번째-선택-어떻게-배송할까-전송-모드">3. IPsec의 두 번째 선택: 어떻게 배송할까? (전송 모드)</h2>
<p>데이터를 금고(ESP)에 담았다면, 이제 송장(주소)을 어디에 붙일지 정해야 합니다. <strong>여기가 가장 중요합니다.</strong></p>
<h3 id="🚗-전송-모드-transport-mode-자물쇠-가방-배송">🚗 전송 모드 (Transport Mode): &quot;자물쇠 가방 배송&quot;</h3>
<ul>
<li><strong>방식</strong>: 금고(데이터) 손잡이에 <strong>&#39;최종 목적지 주소&#39; 송장을 바로 붙여서</strong> 보냅니다.</li>
<li><strong>구조</strong>: <code>[Original IP] + [ESP Header] + [Encrypted Data]</code></li>
<li><strong>단점</strong>: 내용물은 안전하지만, 배달부(해커/라우터)가 <strong>&quot;아, 이 금고는 철수가 영희한테 보내는 거구나&quot;</strong>라고 단번에 알아챕니다. 누가 누구랑 통신하는지 경로가 노출되죠.</li>
<li><strong>용도</strong>: 주로 <strong>서버와 서버(Host-to-Host)</strong>가 직접 통신할 때 사용합니다.</li>
</ul>
<h3 id="🚛-터널-모드-tunnel-mode-이중-박스-포장-안심-소포">🚛 터널 모드 (Tunnel Mode): &quot;이중 박스 포장 (안심 소포)&quot;</h3>
<ul>
<li><strong>방식</strong>: 금고를 <strong>&#39;커다란 겉박스&#39;</strong>에 한 번 더 넣고, 박스 겉면에는 <strong>&#39;중계 센터(VPN 장비)&#39; 주소</strong>만 적습니다.</li>
<li><strong>구조</strong>: <code>[New IP] + [ESP Header] + [Encrypted Original IP + Data]</code></li>
<li><strong>장점</strong>: 배달부는 이 박스가 물류센터로 간다는 것만 알 뿐, <strong>박스를 뜯기 전까진 그 안의 금고가 실제로 누구에게 갈 물건인지 절대 알 수 없습니다.</strong></li>
<li><strong>용도</strong>: <strong>본사와 지사(Site-to-Site)</strong>를 연결할 때 필수적으로 사용합니다. (회사의 내부 IP 주소 체계까지 완벽히 숨김)</li>
</ul>
<hr>
<h2 id="4-deep-dive-패킷-까보기-packet-structure">4. [Deep Dive] 패킷 까보기 (Packet Structure)</h2>
<p>개발자라면 실제 바이트 덩어리가 어떻게 변하는지 봐야 직성이 풀리죠.
가장 많이 쓰이는 <strong>[ESP + 터널 모드]</strong> 조합일 때, 패킷은 이렇게 변합니다.</p>
<h4 id="1단계-원본-패킷-plain-text">1단계: 원본 패킷 (Plain Text)</h4>
<pre><code class="language-text">[ Original IP Header ] + [ TCP Header ] + [ Data: &quot;Hello&quot; ]
</code></pre>
<ul>
<li>해커가 스니핑(Sniffing)하면 &quot;Hello&quot;가 그대로 보입니다.</li>
</ul>
<h4 id="2단계-ipsec-캡슐화-후-encrypted">2단계: IPsec 캡슐화 후 (Encrypted)</h4>
<p>커널의 IPsec 엔진을 통과하면 <strong>L3 계층이 통째로 겉박스에 포장</strong>됩니다.</p>
<pre><code class="language-text">[ New IP Header ]           &lt;-- 1. 겉박스 송장 (출발: VPN게이트웨이 / 도착: 상대방게이트웨이)
[ ESP Header ]              &lt;-- 2. 금고 식별번호 (SPI, Sequence Number)
[ *********************** ]
[ * [ Original IP ]     * ] &lt;-- 3. 암호화된 영역 시작
[ * [ TCP Header ]      * ]     (원래 주소와 데이터가 모두 암호문으로 변했습니다)
[ * [ Data: &quot;Hello&quot; ]   * ]     (해커는 이 안을 절대 볼 수 없습니다)
[ *********************** ]
[ ESP Trailer &amp; Auth ]      &lt;-- 4. 봉인 도장 (HMAC)
</code></pre>
<p><strong>핵심 포인트:</strong></p>
<ol>
<li><strong>New IP Header</strong>: 인터넷 라우터들은 이 겉박스 주소만 보고 패킷을 배달합니다.</li>
<li><strong>Encrypted Payload</strong>: 원래의 <code>Original IP</code>까지 암호화되었기 때문에, 최종 목적지가 어디인지 외부에서는 절대 알 수 없습니다.</li>
</ol>
<hr>
<h2 id="5-faq-ipsec에-대한-오해와-진실">5. FAQ: IPsec에 대한 오해와 진실</h2>
<h3 id="q1-https를-쓰는데-굳이-ipsec-vpn을-또-써야-하나요">Q1. HTTPS를 쓰는데 굳이 IPsec VPN을 또 써야 하나요?</h3>
<p><strong>A. 네, 쓰면 더 좋습니다. (이중 암호화)</strong>
HTTPS는 <strong>내용물(L7)</strong>을 보호하고, IPsec은 <strong>배송 경로(L3)</strong> 자체를 보호합니다.
둘을 같이 쓰면 <code>[ IPsec 암호 [ HTTPS 암호 [ Data ] ] ]</code> 형태가 되어, VPN 관리자조차 여러분의 HTTPS 통신 내용을 볼 수 없는 <strong>&#39;제로 트러스트(Zero Trust)&#39;</strong> 환경이 완성됩니다.</p>
<h3 id="q2-ipsec은-소프트웨어인가요-하드웨어인가요">Q2. IPsec은 소프트웨어인가요, 하드웨어인가요?</h3>
<p><strong>A. IPsec 자체는 프로토콜(S/W)입니다.</strong>
하지만 실무에서 보는 기업용 &#39;VPN 장비&#39;는 이 IPsec 연산(AES-256 등)을 빠르게 처리하기 위해 <strong>가속 칩(ASIC)</strong>을 탑재한 특수 목적의 리눅스 컴퓨터입니다.</p>
<hr>
<h2 id="6-마치며">6. 마치며</h2>
<p>IPsec은 단순한 보안 도구가 아니라, <strong>인터넷이라는 불안한 공용망 위에 우리만의 전용 고속도로를 건설하는 토목 공사</strong>와 같습니다.</p>
<p>앞으로 AWS에서 <code>Site-to-Site VPN</code>을 설정하거나 패킷 로그를 볼 때, <strong>&quot;아, 지금 내 데이터가 금고(ESP)에 담겨 이중 박스 포장(터널 모드)이 된 상태구나!&quot;</strong>라고 이미지를 떠올려 보세요. 훨씬 이해가 빠를 겁니다.</p>
