<h2>TLS</h2>
<p>TLS(Transport Layer Security)는 전송 계층에서 개인 정보와 데이터 무결성을 제공하는 보안 프로토콜이다. 즉, 네트워크 상에서 데이터를 안전하게 주고 받기 위한 암호화 프로토콜이다.</p>
<p>가장 최신 버전인 TLS 1.3은 2018년도에 나왔으며 현재는, TLS 1.2와 TLS 1.3이 주로 사용되고 있다.</p>
<h3>키 교환 알고리즘</h3>
<p><figure class="imageblock alignCenter"><span><img height="240" src="https://blog.kakaocdn.net/dn/cnTl7n/btsNZ3WlCVN/doA0YzgKBHmBCFWhk1Kkd1/img.png" width="482" /></span></figure>
</p>
<h2>TLS&nbsp;1.2/TLS&nbsp;1.3&nbsp;handshake&nbsp;방식&nbsp;비교</h2>
<h3>TLS 1.2</h3>
<p><figure class="imageblock alignCenter"><span><img height="437" src="https://blog.kakaocdn.net/dn/DOSgy/btsN1iqZ35g/1DWAG7hhKAaa9f8zkjECz0/img.png" width="649" /></span></figure>
</p>
<p>&nbsp;</p>
<h4 style="color: #000000; text-align: start;">1. Client Hello</h4>
<p style="color: #333333; text-align: start;">클라이언트가 서버에 접속하는 단계이다. 다만, 이때 클라이언트는 랜덤 데이터(Clinet Random)와 클라이언트 측에서 지원하는 암호화 방식을 서버에 함께 전송한다.</p>
<h4 style="color: #000000; text-align: start;">2. Server Hello</h4>
<p style="color: #333333; text-align: start;">서버는 클라이언트의 인사에 답변한다. 서버 역시 클라이언트와 마찬가지로 랜덤 데이터(Server Random)를 생성하고 서버 측에서 지원하는 암호화 방식을 클라리언트에 전송한다. 더하여 서버 측에서 인증서도 함께 클라이언트에 전송한다.</p>
<h4 style="color: #000000; text-align: start;">3. 서버 인증서 및 키 교환 <span style="color: #9d9d9d;">(RSA 방식 기준)</span></h4>
<p><span style="color: #456771; text-align: start;">* TLS 1.2는 RSA 방식 외에도 다양한 키 교환 방식을 지원한다.</span></p>
<p style="color: #333333; text-align: start;">클라이언트는 서버의 인증서가 CA(Certificate Authority)에서 발급된 것이 맞는지 확인한다. 해당 인증서 안에는 서버가 생성한 공개 키(Pulbic Key)가 포함되어 있다. RSA 방식 기준, 클라이언트는<span>&nbsp;</span><b>Pre Master Secret</b>이라는 키를 생성하고<span>&nbsp;</span>인증서 안에 포함되어 있던 서버 키의 공개키를 사용해서 Pre Master Secret을 암호화한 뒤, 서버에 전송한다.</p>
<blockquote style="color: #666666; text-align: left;"><b><b>Pre Master Secret</b>란?</b><br />클라이언트가 생성하는 임시 비밀 값이다. 클라이언트는 이 값을 서버의 공개 키로 암호화해서 서버에 보내고 서버는 자신의 개인키로 복호화하여 값을 알아낼 수 있다. 해당 값은 최종 암호화 키(Session Key)를 만들기 위한 재료다.</blockquote>
<h4 style="color: #000000; text-align: start;">4. 공통 키 계산 및 세션 키 생성</h4>
<p style="color: #333333; text-align: start;">서버는 비공개 키(Private Key)를 가지고 있기 때문에, 비공개 키로 Pre Master Secret을 복호화한다. 이로써, 클라이언트와 서버는 Pre-Master Secret(Shared Secret) 값을 공유할 수 있다. 서버는 해당 값과 추가 정보<span style="color: #666666;">("Client Hello 단계에서 만든 클라이언트 측 랜덤 데이터"와 "Server Hello 단계에서 만든 서버 측 랜덤 데이터" 등)</span>을 조합하여<span>&nbsp;</span><b>Master Secret</b>이라는 중간 키를 생성한다. 클라이언트 역시 서버와 동일한 정보를 가지고 있기 때문에 같은 중간 키를 생성할 수 있다.<span style="color: #666666;">(즉, 키를 교환하는 과정 없이 각자 계산하여 암호화에 사용할 키를 생성할 수 있음)</span> 해당 Master Secret은 실제로 데이터를 암호화하고 복호화할 때 사용되는 대칭키인<span>&nbsp;</span><b>Session Key</b>를 만드는데 사용이 된다. 이 후 통신은 해당 Session Key를 사용해 대칭키 방식으로 이루어진다.</p>
<h4 style="color: #000000; text-align: start;">5. ChangeCipherSpec &amp; Finished</h4>
<p>클라이언트는 세션 키를 생성한 후, 앞으로의 통신을 대칭 암호화 방식으로 전환할 때 "Client Finished" 메세지와 함께 "Change Cipher Spec"이라는 메세지를 보낸다. "Finished" 메세지는 지금까지의 모든 handshake 메세지를 기반으로 무결성(MAC)을 검증하는 역할을 한다. 서버 역시 클라이언트와 동일 작업을 수행하고 보안 상태를 대칭 암호화로 전환하며 Handshake를 마친다.</p>
<h3>TLS 1.3</h3>
<p><figure class="imageblock alignCenter"><span><img height="412" src="https://blog.kakaocdn.net/dn/lQzY4/btsN1d4no8C/Co8aWq8vkJkaVkj3QZg6Z0/img.png" width="605" /></span></figure>
</p>
<h4>1. Client Hello</h4>
<p style="color: #333333; text-align: start;">TLS 1.2 버전과 마찬가지로 클라이언트가 서버에 접속하는 단계이다. 다만, TLS 1.3 버전은 키 교환 알고리즘으로 ECDHE(Diffie-Hellman)만을 사용한다. 때문에, 클라이언트는 서버에게 랜덤 데이터(Client Random)와 클라이언트의 임시 Diffie-Hellman 공개키를 전송한다. 즉, 해당 단계에서 TLS 1.2와 다르게 이미 키 공유가 이루어진다.</p>
<h4 style="color: #000000; text-align: start;">2. Server Hello</h4>
<p style="color: #333333; text-align: start;">서버는 "Server Hello" 메세지를 보내며, 서버의 ECDHE 공개키도 함께 전송한다. 이후, 서버는 자신의 인증서와 함께, 인증서의 소유권을 입증하는 CertificateVerify 메시지와 Finished 메세지를 순차적으로 보낸다. 해당 시점에서 클라이언트와 서버는 서로의 키로 공통의 Shared Secret을 계산할 수 있다. <span style="color: #666666;">(TLS 1.2와 달리 Server Random은 보내지 않는다.)</span></p>
<blockquote>클라이언트와 서버는 ECDHE를 이용해 Shared Secret을 계산하고, 이를 기반으로 HKDF(HMAC-based Key Derivation Function)을 통해 다양한 키를 생성할 수 있다.</blockquote>
<h4 style="color: #000000; text-align: start;">3. Client Finished</h4>
<p style="color: #333333; text-align: start;">클라이언트도 서버로부터 받은 메세지의 무결성을 검증한 후, 자신의 Finished 메세지를 보낸다. 해당 메세지는 세션 키로 암호화되어 있으며, 클라이언트 측에서 handshake가 완료되었음을 의미한다.</p>
<h3 style="color: #333333; text-align: start;">정리</h3>
<p>TLS 1.2 버전에서는 handshake를 완료하기 위해서 클라이언트와 서버 사이 3번의 왕복이 필요했다. 하지만 TLS 1.3에서의 handshake 프로세스는 단 1회의 왕복만 포함되기 때문에 대기 시간이 단축되게 된다.</p>
<p><figure class="imageblock widthContent"><span><img height="797" src="https://blog.kakaocdn.net/dn/oyopj/btsN1A6imxj/0wqVAuJzcNlGuBhMixSC4k/img.png" width="999" /></span></figure>
</p>
<p>실제로 handshake에서 소요되는 시간을 확인해보면 TLS 1.3은 TLS 1.2보다 평균적으로 56% 더 효율적이다.&nbsp;</p>
<h2>참고자료</h2>
<p><a href="https://www.go-euc.com/tls12-vs-tlsv13-a-deep-dive-into-handshake-efficiency/" rel="noopener&nbsp;noreferrer" target="_blank">https://www.go-euc.com/tls12-vs-tlsv13-a-deep-dive-into-handshake-efficiency/</a></p>
<figure contenteditable="false" id="og_1747385825481"><a href="https://www.go-euc.com/tls12-vs-tlsv13-a-deep-dive-into-handshake-efficiency/" rel="noopener" target="_blank">
<div class="og-image">&nbsp;</div>
<div class="og-text">
<p class="og-title">TLSv1.2 vs TLSv1.3: A Deep Dive into Handshake Efficiency | GO-EUC</p>
<p class="og-desc">Table of Contents Cloud Providers, Load Balancers, Security Experts, Architects, etc., all have one thing in common. At some point, they will advise you to utilize TLSv1.2 and/or TLSv1.3 protocols and abandon legacy protocols like TLSv1.0 and TLSv1.1. Lega</p>
<p class="og-host">www.go-euc.com</p>
</div>
</a></figure>
<p><a href="https://www.wolfssl.com/tls-1-3-performance-part-2-full-handshake-2/" rel="noopener&nbsp;noreferrer" target="_blank">https://www.wolfssl.com/tls-1-3-performance-part-2-full-handshake-2/</a></p>
<figure contenteditable="false" id="og_1747385835823"><a href="https://www.wolfssl.com/tls-1-3-performance-part-2-full-handshake-2/" rel="noopener" target="_blank">
<div class="og-image">&nbsp;</div>
<div class="og-text">
<p class="og-title">TLS 1.3 Performance Analysis &ndash; Full Handshake &ndash; wolfSSL</p>
<p class="og-desc">Significant changes from TLS 1.2 have been made in TLS 1.3 that are targeted at performance. This is the second part of six blogs discussing the performance differences observed between TLS 1.2 and TLS 1.3 in wolfSSL and how to make the most of them in you</p>
<p class="og-host">www.wolfssl.com</p>
</div>
</a></figure>
<p>&nbsp;</p>