---layout: posttitle: "[CKA] 쿠버네티스 NetworkPolicy 완벽 이해: 파드 격리와 DNS의 함정"date: 2026-02-04 23:57:04 +0000
categories: [Velog]---<h3 id="1-들어가며-쿠버네티스의-문은-원래-활짝-열려있다">1. 들어가며: 쿠버네티스의 문은 원래 활짝 열려있다</h3>
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
<h4 id="나는-구글googlecom은-막고-dbdb-service랑만-통신할-거야">&quot;나는 구글(google.com)은 막고 DB(db-service)랑만 통신할 거야!&quot;</h4>
<p>개발자는 이렇게 생각하고 DB IP만 허용하려 합니다. 하지만 여기서 간과한 사실이 있습니다.</p>
<ol>
<li>애플리케이션은 보통 IP(<code>10.96.0.5</code>)가 아니라 <strong>도메인 이름</strong>(<code>db-service</code>, <code>google.com</code>)으로 통신을 시도합니다.</li>
<li>이 이름을 IP로 바꾸기 위해 파드는 <strong>가장 먼저 DNS 서버(CoreDNS)에게 물어봅니다.</strong> (&quot;야, db-service 주소가 어디야?&quot;)</li>
<li>이 질문은 <strong>UDP/TCP 53번 포트</strong>를 통해 나갑니다.</li>
</ol>
<h4 id="만약-egress-정책에서-53번-포트를-안-열어주면">만약 Egress 정책에서 53번 포트를 안 열어주면?</h4>
<p>파드는 <code>db-service</code>가 어디 있는지 물어보지도 못합니다. (전화번호부를 뺏긴 셈이죠.)
결국 실제 DB 연결은 시도조차 못 해보고 <strong>&quot;Name Resolution Error&quot;</strong> 혹은 <strong>&quot;Timeout&quot;</strong>이 발생하며 뻗어버립니다.</p>
<blockquote>
<p><strong>결론</strong>: Egress를 막을 거라면, <strong>숨 쉬는 구멍(DNS 53번)</strong>은 무조건 열어둬야 합니다.</p>
</blockquote>
<hr>
<h3 id="4-실전-시나리오-cka-기출-유형">4. 실전 시나리오 (CKA 기출 유형)</h3>
<p>이제 개념을 잡았으니 실제 요구사항을 구현해 봅시다.</p>
<ul>
<li><strong>상황</strong>: <code>space1</code>과 <code>space2</code> 네임스페이스 존재.</li>
<li><strong>미션 1</strong>: <code>space1</code> 파드는 <strong>오직 <code>space2</code>로만</strong> 나갈 수 있다. (나머지 차단)</li>
<li><strong>미션 2</strong>: <code>space2</code> 파드는 <strong>오직 <code>space1</code>에서 온 것만</strong> 받을 수 있다. (나머지 차단)</li>
</ul>
<hr>
<h3 id="5-해결-과정-yaml-작성">5. 해결 과정 (YAML 작성)</h3>
<h4 id="정책-1-space1의-egress-제한-dns-필수">정책 1: <code>space1</code>의 Egress 제한 (DNS 필수!)</h4>
<p><code>space1</code> 입장에서 나가는 트래픽을 제어합니다.</p>
<pre><code class="language-yaml">apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-space1
  namespace: space1
spec:
  podSelector: {} # space1의 모든 파드 적용
  policyTypes:
  - Egress        # 나가는 것 단속 시작
  egress:
  # 규칙 1: space2 동네로 가는 건 허용
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: space2
  # 규칙 2: DNS 조회(53번)는 무조건 허용 (중요!)
  - ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
</code></pre>
<h4 id="정책-2-space2의-ingress-제한">정책 2: <code>space2</code>의 Ingress 제한</h4>
<p><code>space2</code> 입장에서 들어오는 트래픽을 제어합니다. 여기선 DNS를 열 필요가 없습니다. (들어오는 요청이니까요)</p>
<pre><code class="language-yaml">apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-space2
  namespace: space2
spec:
  podSelector: {}
  policyTypes:
  - Ingress       # 들어오는 것 단속 시작
  ingress:
  # 규칙 1: space1 동네에서 온 손님만 통과
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: space1
</code></pre>
<hr>
<h3 id="6-검증-막힌-곳과-뚫린-곳-확인하기">6. 검증: 막힌 곳과 뚫린 곳 확인하기</h3>
<p>설정 후에는 반드시 테스트 파드(<code>busybox</code>)를 띄워 확인해야 합니다.</p>
<ol>
<li><strong>DNS 확인</strong>: <code>nslookup microservice1.space2</code> 가 성공하는가? (실패하면 53번 포트 문제)</li>
<li><strong>차단 확인</strong>: <code>wget -O- google.com</code> 이 타임아웃 되는가? (성공하면 정책 적용 안 된 것)</li>
<li><strong>허용 확인</strong>: <code>wget -O- microservice1.space2</code> 가 성공하는가?</li>
</ol>
<h3 id="7-마치며">7. 마치며</h3>
<p>NetworkPolicy는 쿠버네티스 보안의 기본이자 CKA 시험의 핵심 파트입니다.
<strong>&quot;기본은 모두 차단(Default Deny), 필요한 것만 하나씩 구멍 뚫어주기(Allow)&quot;</strong>라는 원칙을 기억하세요. 그리고 그 첫 번째 구멍은 언제나 <strong>DNS(53)</strong>여야 한다는 점을 잊지 마시길 바랍니다.</p>
