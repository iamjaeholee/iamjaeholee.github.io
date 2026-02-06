---
layout: post
title: "[CKA] 쿠버네티스 NetworkPolicy 완벽 이해: 파드 격리와 DNS의 함정"
date: 2026-02-04 23:57:04 +0000
categories: [Velog]
---

<h3 id="1-들어가며-쿠버네티스의-문은-원래-활짝-열려있다">1. 들어가며: 쿠버네티스의 문은 원래 활짝 열려있다</h3>
<p>쿠버네티스를 처음 접할 때 가장 오해하기 쉬운 것 중 하나가 보안입니다. 기본적으로 쿠버네티스 클러스터 내의 모든 파드(Pod)는 <strong>서로 자유롭게 통신이 가능(Non-isolated)</strong>합니다.
마치 옆집, 윗집 문이 다 열려 있어서 누구든 마음대로 드나들 수 있는 아파트와 같죠. 개발할 땐 편하지만, 보안상으로는 굉장히 위험한 구조입니다.</p>
<p>이때 필요한 것이 바로 <strong>NetworkPolicy(네트워크 정책)</strong>입니다. 이번 포스팅에서는 NetworkPolicy의 개념과 실무에서 가장 많이 실수하는 <strong>DNS 설정</strong>에 대해 정리해 봅니다.</p>
<hr>
<h3 id="2-networkpolicy란-무엇인가">2. NetworkPolicy란 무엇인가?</h3>
<p>쉽게 말해 <strong>&quot;쿠버네티스 전용 방화벽&quot;</strong>입니다.
AWS의 Security Group이 IP와 포트를 기반으로 EC2를 보호한다면, NetworkPolicy는 <strong>라벨(Label)</strong>을 기반으로 파드를 보호합니다.</p>
<p>NetworkPolicy의 핵심 작동 원리는 다음과 같습니다.</p>
<ol>
<li><strong>화이트리스트(Allow List) 방식</strong>: &quot;이것만 허용해!&quot;라고 규칙을 정하면, 그 외 나머지는 모두 차단됩니다.</li>
<li><strong>방향성 제어</strong>:</li>
</ol>
<ul>
<li><strong>Ingress</strong>: 밖에서 나에게 <strong>들어오는</strong> 트래픽</li>
<li><strong>Egress</strong>: 내가 밖으로 <strong>나가는</strong> 트래픽</li>
</ul>
<ol start="3">
<li><strong>적용 대상</strong>: <code>podSelector</code>를 통해 특정 파드들에만 정책을 적용할 수 있습니다.</li>
</ol>
<blockquote>
<p><strong>💡 핵심</strong>: 아무 정책도 없으면 <strong>&quot;모두 허용&quot;</strong>이지만, 정책이 하나라도 생기는 순간 해당 파드는 <strong>&quot;고립(Isolated)&quot;</strong> 상태가 되어 명시된 친구하고만 대화할 수 있게 됩니다.</p>
</blockquote>
<hr>
<h3 id="3-왜-53번-포트dns를-꼭-열어줘야-할까">3. 왜 53번 포트(DNS)를 꼭 열어줘야 할까?</h3>
<p>NetworkPolicy를 설정할 때 가장 많이 하는 실수가 <strong>Egress(나가는 트래픽)를 제한하면서 DNS를 까먹는 경우</strong>입니다.</p>
<p>나는 구글(google.com)은 막고 DB(db-service)랑만 통신할 거야!&quot;
개발자는 이렇게 생각하고 DB IP만 허용하려 합니다. 하지만 여기서 간과한 사실이 있습니다.</p>
<ol>
<li>애플리케이션은 보통 IP(<code class="language-text">10.96.0.5</code>)가 아니라 <strong>도메인 이름</strong>(<code class="language-text">db-service</code>, <code class="language-text">google.com</code>)으로 통신을 시도합니다.</li>
<li>이 이름을 IP로 바꾸기 위해 파드는 <strong>가장 먼저 DNS 서버(CoreDNS)에게 물어봅니다.</strong> (&quot;야, db-service 주소가 어디야?&quot;)</li>
<li>이 질문은 <strong>UDP/TCP 53번 포트</strong>를 통해 나갑니다.</li>
</ol>
<p>만약 Egress 정책에서 53번 포트를 안 열어주면?</p>
<p>파드는 <code class="language-text">db-service</code>가 어디 있는지 물어보지도 못합니다. (전화번호부를 뺏긴 셈이죠.)
결국 실제 DB 연결은 시도조차 못 해보고 <strong>&quot;Name Resolution Error&quot;</strong> 혹은 <strong>&quot;Timeout&quot;</strong>이 발생하며 뻗어버립니다.</p>
<blockquote>
<p><strong>결론</strong>: Egress를 막을 거라면, <strong>숨 쉬는 구멍(DNS 53번)</strong>은 무조건 열어둬야 합니다.</p>
</blockquote>
<hr>
<h3 id="4-실전-시나리오-cka-기출-유형">4. 실전 시나리오 (CKA 기출 유형)</h3>
<p>이제 개념을 잡았으니 실제 요구사항을 구현해 봅시다.</p>
<ul>
<li><strong>상황</strong>: <code class="language-text">space1</code>과 <code class="language-text">space2</code> 네임스페이스 존재.</li>
<li><strong>미션 1</strong>: <code class="language-text">space1</code> 파드는 <strong>오직 <code class="language-text">space2</code>로만</strong> 나갈 수 있다. (나머지 차단)</li>
<li><strong>미션 2</strong>: <code class="language-text">space2</code> 파드는 <strong>오직 <code class="language-text">space1</code>에서 온 것만</strong> 받을 수 있다. (나머지 차단)</li>
</ul>
<hr>
<h3 id="5-해결-과정-yaml-작성">5. 해결 과정 (YAML 작성)</h3>
<h4 id="정책-1-space1의-egress-제한-dns-필수">정책 1: <code class="language-text">space1</code>의 Egress 제한 (DNS 필수!)</h4>
<p><code class="language-text">space1</code> 입장에서 나가는 트래픽을 제어합니다.</p>
<pre><code class="language-yaml">apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-space1
  namespace: space1
spec:
  podSelector: {} # Selects ALL pods in this namespace
  policyTypes:
  - Egress        # We are controlling outgoing traffic
  egress:
  # Rule 1: Allow traffic to &#39;space2&#39; namespace
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: space2
  # Rule 2: CRITICAL! Allow DNS resolution
  - ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
</code></pre>
<blockquote>
<p><strong>Pro Tip:</strong> Since Kubernetes v1.21, namespaces automatically have the label <code class="language-text">kubernetes.io/metadata.name: &lt;ns-name&gt;</code>. You can use this instead of creating manual labels!</p>
</blockquote>
<h4 id="policy-2-restricting-ingress-in-space2">Policy 2: Restricting Ingress in <code class="language-text">space2</code></h4>
<p>Here, we only need to filter who is coming <em class="language-text">in</em>. We don&#39;t need to worry about DNS here, as that is an Egress concern.</p>
<pre><code class="language-yaml">apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-space2
  namespace: space2
spec:
  podSelector: {}
  policyTypes:
  - Ingress       # We are controlling incoming traffic
  ingress:
  # Rule 1: Allow traffic coming FROM &#39;space1&#39;
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: space1
</code></pre>
<hr>
<h3 id="6-verification--troubleshooting">6. Verification &amp; Troubleshooting</h3>
<p>Applying the YAML is easy. Verifying it is the real skill. Use a temporary <code class="language-text">busybox</code> pod to test connectivity.</p>
<h4 id="test-1-verify-block-space1---external">Test 1: Verify Block (space1 -&gt; External)</h4>
<p>Try to reach Google. This should fail (timeout) because we only allowed <code class="language-text">space2</code>.</p>
<pre><code class="language-bash">kubectl -n space1 run test-pod --image=busybox -it --rm -- wget -O- google.com
# Result: &quot;wget: download timed out&quot; (Pass!)
</code></pre>
<h4 id="test-2-verify-allow-space1---space2">Test 2: Verify Allow (space1 -&gt; space2)</h4>
<p>Try to reach a service in <code class="language-text">space2</code>. <strong>Note:</strong> You must use the FQDN (<code class="language-text">service.namespace</code>).</p>
<pre><code class="language-bash">kubectl -n space1 run test-pod --image=busybox -it --rm -- wget -O- -T 5 microservice1.space2
# Result: HTML output or connection success (Pass!)
</code></pre>
<h4 id="test-3-debugging-dns">Test 3: Debugging DNS</h4>
<p>If Test 2 fails immediately with &quot;Bad Address,&quot; it means DNS is blocked. Check your port 53 rules!</p>
<pre><code class="language-bash">kubectl -n space1 run debug --image=busybox -it --rm -- nslookup microservice1.space2
</code></pre>
<hr>
<h3 id="7-takeaways">7. Takeaways</h3>
<ol>
<li><strong>Default Deny:</strong> Once a NetworkPolicy selects a Pod, it isolates it.</li>
<li><strong>Explicit Allow:</strong> You must list every destination you want to reach.</li>
<li><strong>Port 53 is Lifeline:</strong> Never forget to allow DNS (UDP/TCP 53) in your Egress rules, or your application will be blind.</li>
</ol>
<p>Mastering this flow is crucial not just for the CKA exam, but for securing any production Kubernetes cluster.</p>