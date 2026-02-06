---layout: post
title: "[CKA] Mastering K8s NetworkPolicy: Pod Isolation & The DNS Trap"
date: 2026-02-04 23:58:02 +0000
categories: [Velog]
original_url: https://velog.io/@iamjaeholee/CKA-Mastering-K8s-NetworkPolicy-Pod-Isolation-The-DNS-Trap-bj704iys
---
<h3 id="1-introduction-kubernetes-is-open-by-default">1. Introduction: Kubernetes is &quot;Open&quot; by Default</h3>
<p>One of the first things you realize when learning Kubernetes is that <strong>all Pods can communicate with all other Pods</strong> by default. It’s a flat network structure.</p>
<p>While this is convenient for development, it’s a security nightmare in production. Imagine your frontend web server having unrestricted access to your database, or a compromised pod having access to your entire internal network.</p>
<p>To solve this, we use <strong>NetworkPolicy</strong>. In this post, I’ll walk through how to isolate namespaces and avoid the most common &quot;gotcha&quot; in the CKA exam: <strong>Forgetting DNS.</strong></p>
<hr>
<h3 id="2-what-is-networkpolicy">2. What is NetworkPolicy?</h3>
<p>Think of NetworkPolicy as a <strong>&quot;Kubernetes-native Firewall.&quot;</strong>
Unlike traditional firewalls that use IPs, NetworkPolicy controls traffic based on <strong>Labels</strong> and <strong>Selectors</strong>.</p>
<p>Key concepts to understand:</p>
<ol>
<li><strong>Allow-List Model (Whitelist):</strong> NetworkPolicy is additive. If you create a policy that selects a Pod, that Pod is immediately <strong>isolated</strong>, and it rejects all traffic <em>except</em> what you explicitly allow.</li>
<li><strong>Traffic Direction:</strong></li>
</ol>
<ul>
<li><strong>Ingress:</strong> Incoming traffic (to the Pod).</li>
<li><strong>Egress:</strong> Outgoing traffic (from the Pod).</li>
</ul>
<ol start="3">
<li><strong>Scope:</strong> It is a namespaced resource.</li>
</ol>
<blockquote>
<p><strong>The Golden Rule:</strong> &quot;If no policy exists, everything is allowed. If a policy exists, everything else is denied (Default Deny).&quot;</p>
</blockquote>
<hr>
<h3 id="3-the-dns-trap-why-port-53-matters">3. The &quot;DNS Trap&quot;: Why Port 53 Matters</h3>
<p>This is where many developers (and CKA candidates) fail. When you restrict <strong>Egress</strong> traffic, you might think: *&quot;I only want my app to talk to the Backend Service, so I&#39;ll only allow that.&quot;*</p>
<p><strong>The Problem:</strong>
Your application doesn&#39;t talk to IP addresses directly. It uses domain names (e.g., <code>backend-svc</code>, <code>google.com</code>).
To resolve these names to IPs, your Pod <strong>must</strong> talk to the cluster&#39;s DNS server (CoreDNS).</p>
<p><strong>The Trap:</strong>
If you block all Egress traffic and only allow the Backend Service, <strong>you are also blocking the DNS query.</strong>
Your app will crash with a &quot;Name Resolution Error&quot; or &quot;Timeout&quot; because it can&#39;t ask &quot;Where is <code>backend-svc</code>?&quot;</p>
<p><strong>The Solution:</strong>
Whenever you restrict Egress, <strong>you must explicitly allow UDP/TCP port 53.</strong></p>
<hr>
<h3 id="4-the-scenario-cka-style">4. The Scenario (CKA Style)</h3>
<p>Let&#39;s look at a practical requirement:</p>
<ul>
<li><strong>Context:</strong> Two Namespaces exist: <code>space1</code> and <code>space2</code>.</li>
<li><strong>Requirement A:</strong> Pods in <code>space1</code> can <strong>only</strong> send traffic to <code>space2</code>. (All other outgoing traffic is blocked).</li>
<li><strong>Requirement B:</strong> Pods in <code>space2</code> can <strong>only</strong> receive traffic from <code>space1</code>. (All other incoming traffic is blocked).</li>
</ul>
<hr>
<h3 id="5-the-solution">5. The Solution</h3>
<h4 id="policy-1-restricting-egress-in-space1-dont-forget-dns">Policy 1: Restricting Egress in <code>space1</code> (Don&#39;t forget DNS!)</h4>
<p>We need to allow traffic to <code>space2</code> AND traffic to the DNS server.</p>
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
<p><strong>Pro Tip:</strong> Since Kubernetes v1.21, namespaces automatically have the label <code>kubernetes.io/metadata.name: &lt;ns-name&gt;</code>. You can use this instead of creating manual labels!</p>
</blockquote>
<h4 id="policy-2-restricting-ingress-in-space2">Policy 2: Restricting Ingress in <code>space2</code></h4>
<p>Here, we only need to filter who is coming <em>in</em>. We don&#39;t need to worry about DNS here, as that is an Egress concern.</p>
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
<p>Applying the YAML is easy. Verifying it is the real skill. Use a temporary <code>busybox</code> pod to test connectivity.</p>
<h4 id="test-1-verify-block-space1---external">Test 1: Verify Block (space1 -&gt; External)</h4>
<p>Try to reach Google. This should fail (timeout) because we only allowed <code>space2</code>.</p>
<pre><code class="language-bash">kubectl -n space1 run test-pod --image=busybox -it --rm -- wget -O- google.com
# Result: &quot;wget: download timed out&quot; (Pass!)
</code></pre>
<h4 id="test-2-verify-allow-space1---space2">Test 2: Verify Allow (space1 -&gt; space2)</h4>
<p>Try to reach a service in <code>space2</code>. <strong>Note:</strong> You must use the FQDN (<code>service.namespace</code>).</p>
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
