<h2>Push&nbsp;기반&nbsp;통신을&nbsp;구현하는&nbsp;방법</h2>
<h3 style="color: #000000; text-align: start;">들아가며..</h3>
<p style="color: #333333; text-align: start;"><span style="color: #333333; text-align: start;">HTTP는 근본적으로 클라이언트가 서버에 요청을 보내기 전까지는 응답을 하지 않는다.<span>&nbsp;</span></span>이처럼 클라이언트-서버 간의 전형적인 요청-응답(Request-Response)으로 작동하는 HTTP 프로토콜은 서버가 클라이언트에게 능동적으로 업데이트를 보내야 하는 상황에서는 효율적이지 않다. HTTP는 기본적으로 클라이언트가 요청을 먼저 해야만 서버가 응답을 주는 '<b>pull-based protocol'&nbsp;</b>이기 때문이다. 즉, 클라이언트가 예측하고 요청하지 않으면 서버는 아무것도 보낼 수 없다. 이처럼 HTTP의 Pull 특성은 실시간 Push에 적합하지 않다. 그렇다면 어떻게 서버가 클라이언트에게 실시간으로 데이터를 보낼 수 있을까?</p>
<h3>Polling</h3>
<p><span style="color: #333333; text-align: start;">'<b>Polling</b>'이란, 클라이언트가 서버에게 새로운 데이터나 이벤트가 있는지 주기적으로 확인하는 요청을 전송하는 것을 의미한다.<span> Polling 방식은 응답 대기 시간에 따라 <b>'Short Polling'</b>과 <b>'Long Polling'</b>으로 나뉜다.</span></span></p>
<h4>Short Polling</h4>
<p><figure class="imageblock widthContent"><span><img height="530" src="https://blog.kakaocdn.net/dn/bbN5Ck/btsN5IJ5YdM/Lk1iZA9B3Uk2MDeNMwpcq0/img.png" width="1538" /></span></figure>
</p>
<p>Short Polling은 가장 기본적인 폴링 방식으로,<b> 클라이언트가 서버에게 일정 주기로 요청을 보내는 방식</b>이다. 서버는 응답할 데이터의 유무에 상관없이 즉시 응답한다. 즉, 클라이언트는 1~2초 간격으로 서버에 계속 요청을 보내면서 <b>서버의 상태 변화를 지속적으로 확인</b>하는 방식이다. 서버에 새로운 데이터가 생기면 다음 클라이언트 요청 시점에 응답을 내려준다.</p>
<p>Short Polling의 가장 큰 단점은 너무 잦은 HTTP 요청이 발생한다는 점이다. 이는 네트워크 대역폭을 차지하고 서버의 부하를 증가시킬 수 있다. 또한, 클라이언트가 요청을 일정 주기로 보내기 때문에, 서버에 새로운 데이터가 생겼더라도 클라이언트가 다음 요청을 보낼 때까지 최대 요청 간격만큼 기다려야 하므로 지연이 발생할 수 있다. 이 때문에 실시간성 요구가 높은 서비스에는 적합하지 않은 방식이다.</p>
<h4>Long Polling</h4>
<p><figure class="imageblock widthContent"><span><img height="512" src="https://blog.kakaocdn.net/dn/xVx8H/btsN4sodk79/LOczU1XKNHpZSFQ5gRs4O0/img.png" width="1508" /></span></figure>
</p>
<p>Long Polling은 Short Polling의 단점<span style="color: #666666;">(과도한 수의 HTTP 요청)</span>을 개선한 방식으로, <b>HTTP 요청 시 타임아웃 시간을 더 길게 설정</b>하는 방식이다. 클라이언트가 요청을 보내면 <b>서버는 데이터 변화가 있을 때까지 응답을 지연</b>시키고, 사용자가 행동을 하여 <b>데이터의 변화가 생긴다면 즉시 응답</b>한다. 만약 설정된 타임아웃 시간 내에 변화가 없다면, 서버는 타임아웃 시점에 응답을 종료하고 클라이언트는 다시 요청을 보낸다.</p>
<p>Long Polling에도 여전히 단점이 존재한다. 서버가 각 요청마다 일정 시간 동안 연결을 유지해야 하므로, 접속하는 클라이언트 수가 많아지면 서버 자원(메모리, 연결 수 등)에 부담이 커질 수 있다. Long Polling은 Short Polling의 과도한 요청 횟수의 문제를 줄이고 상대적으로 실시간성 또한 향상되었지만, 여전히 완전한 실시간성에는 부족하며 서버 부담도 여전하다.</p>
<h3>Web Socket</h3>
<p>Short Polling과 Long Polling은 QR 코드 스캔과 같은 단순한 작업에는 잘 작동한다. 하지만 온라인 게임처럼 <b>복잡하고 데이터 양이 많으며 실시간성이 중요한 작업</b>에는 더 효율적인 솔루션이 필요하다. 여기서 등장하는 것이 바로 <b>WebSocket</b>이다.</p>
<p><figure class="imageblock widthContent"><span><img height="504" src="https://blog.kakaocdn.net/dn/RWt30/btsN315yns3/TQvsE7Te318sevKeNxF5d0/img.png" width="1506" /></span></figure>
</p>
<p><b>WebSocket은 TCP 기반의 또 다른 프로토콜로</b>, 클라이언트와 서버 간 <b>하나의 연결을 통해 완전한 양방향(full-duplex) 통신을 가능하게</b> 하여 이 문제를 해결한다.</p>
<h4>Web Socket&nbsp; 연결을 설정하는 방법</h4>
<p><figure class="imageblock widthContent"><span><img height="182" src="https://blog.kakaocdn.net/dn/bXbEHF/btsN5tzSizB/QXCSbgt0GPvEIrrxljhYZ1/img.png" width="1454" /></span></figure>
</p>
<p>위 사진에서 볼 수 있듯, WebSocket 연결을 설정하기 위해서는 특정 HTTP 헤더 필드들을 포함한 요청을 보내야 한다. 클라이언트가 WebSocket 연결에 필요한 무작위로 생성된 base64 인코딩된 키(Sec-WebSocket-Key)를 서버에 전송하면 서버는 이에 대해 특정 헤더 값<span style="color: #666666;">(Sec-WebSocket-Key: 클라이언트의 키를 기반으로 계산된 응답 키 등)</span>을 포함한 응답을 보낸다. 이때 사용되는 HTTP 상태 코드 101(Switching Protocols)은 프로토콜이 WebSocket으로 전환되고 있음을 의미한다.</p>
<p><figure class="imageblock widthContent"><span><img height="836" src="https://blog.kakaocdn.net/dn/dgrRjf/btsN4dEFIMD/vdIvpABD0fRwfQ7IjCnXT1/img.png" width="1026" /></span></figure>
</p>
<h4>WebSocket Message</h4>
<p>HTTP가 WebSocket으로 업그레이드되면, 클라이언트와 서버는 프레임 단위로 데이터를 주고받게 된다.&nbsp;</p>
<p><figure class="imageblock widthContent"><span><img height="384" src="https://blog.kakaocdn.net/dn/cOKQfW/btsN41X8Qke/KNqSV2E5STimq3sSXSyg3K/img.png" width="988" /></span></figure>
</p>
<p>프레임 구조를 하나씩 간단하게 살펴보면 먼저 Opcode는 프레임 데이터의 타입을 나타내는 4비트 필드이다. 1은 텍스트 프레임을, 2는 바이너리 프레임을, 8은 연결 종료 신호를 의미한다. Payload length는 일반적으로 7비트 필드로 표현되지만, 필요에 따라 Extended payload length 필드를 통해 더 큰 데이터를 표현할 수도 있다. 이 두 필드를 모두 활용하면, 수 테라바이트에 달하는 데이터까지 전달할 수 있다.</p>
<h3>SSE (Server-Sent Events)</h3>
<p><figure class="imageblock widthContent"><span><img height="522" src="https://blog.kakaocdn.net/dn/bci788/btsN5oynJ3L/zw1K3zqpMvjRJl6mQf42MK/img.png" width="1512" /></span></figure>
</p>
<p>클라이언트가 서버에 SSE 연결을 생성하면, 서버는 이 연결을 끊지 않고 유지하면서 지속적으로 데이터를 클라이언트로 전송할 수 있다. 클라이언트가 서버에 매번 요청을 보내지 않아도, 실시간 주식 시세나 뉴스 속보 알림과 같이 서버 측에서 실시간으로 데이터를 푸시해야 하는 상황에 적합하다. 즉, SSE를 사용하면 업데이트가 있을 때마다 서버가 실시간으로 클라이언트에 데이터를 푸시할 수 있기 때문에 클라이언트가 매번 요청을 보낼 필요가 없다.</p>
<p>단, WebSocket과 달리 SSE는 양방향 통신을 지원하지 않기 때문에 서버와 클라이언트 간의 상호 작용이 자주 필요한 경우에는 적합하지 않다. <span style="color: #333333; text-align: start;">때문에 SSE 방식은<span>&nbsp;</span></span><span style="color: #333333; text-align: start;">클라이언트는 데이터를 수신만 하면 되며, 서버에 별도로 정보를 보낼 필요가 없는 경우에 특히 유용하다.<span>&nbsp;</span></span></p>
<h2>네트워크 프로토콜과 관련 기술들</h2>
<h3><span style="color: #333333; text-align: start;"><span>gRPC</span></span></h3>
<p><figure class="imageblock widthContent"><span><img height="514" src="https://blog.kakaocdn.net/dn/OlFG1/btsN4pLEnf7/Zena3HZQIDVrmN5qD4ajpk/img.png" width="1120" /></span></figure>
</p>
<p><span style="color: #333333; text-align: start;">RPC(Remote Procedure Call)는<span>&nbsp;</span></span><b>다른 서비스에 있는 함수를 호출</b>할 수 있게 해주는 기술이다.</p>
<h4>HTTP vs RPC</h4>
<p><figure class="imageblock widthContent"><span><img height="462" src="https://blog.kakaocdn.net/dn/cyoAXy/btsN43IouwQ/Zwou3RVkgq0sGEIIMb6IkK/img.png" width="1130" /></span></figure>
</p>
<p>RPC가 HTTP에 비해 가지는 가장 큰 장점은 경량화된 메시지 포맷과 향상된 성능이다.</p>
<h4>gRPC의 동작 과정</h4>
<p><figure class="imageblock widthContent"><span><img height="810" src="https://blog.kakaocdn.net/dn/T10Yy/btsN5LfUTjC/Okg39at0kncuyo85F6Tkf1/img.png" width="992" /></span></figure>
</p>
<p>1)</p>
<p>클라이언트가 REST 호출을 보낸다. 이때 요청 본문은 보통 JSON 형식이다.</p>
<p>2~4)</p>
<p>주문 서비스가 gRPC 클라이언트 역할을 하며 REST 호출을 받아 적절한 포맷으로 변환한 뒤, 결제 서비스에 RPC 호출을 시작한다. gRPC는 클라이언트 스텁(클라이언트 프록시)을 바이너리 형식으로 인코딩하여 낮은 수준의 전송 계층으로 전달한다.</p>
<p>5)</p>
<p>gRPC는 HTTP/2를 통해 패킷을 네트워크로 전송한다. <span style="color: #666666;">(바이너리 인코딩과 네트워크 최적화 덕분에 gRPC는 JSON보다 최대 5배 빠른 성능을 낸다고 한다.)</span></p>
<p>6~8)</p>
<p>결제 서비스가 gRPC 서버 역할을 하며 패킷을 수신하고 디코딩한 후 서버 애플리케이션을 실행한다.</p>
<p>9~11)</p>
<p>서버 애플리케이션의 결과를 인코딩해 전송 계층으로 보낸다.</p>
<p>12~14)</p>
<p>주문 서비스가 패킷을 받아 디코딩하고, 그 결과를 클라이언트 애플리케이션에 전달한다.</p>
<h3>Reliable UDP (신뢰성 있는 UDP)</h3>
<h4>등장 배경</h4>
<p>TCP는 여러 성능 오버헤드로 인해 실시간성이 중요한 애플리케이션에는 적합하지 않은 경우가 많다. 대표적인 단점은 다음과 같다</p>
<ul>
<li><b>3-way 핸드셰이크로 인한 연결 지연</b><br />TCP는 연결을 수립하기 위해 3단계 핸드셰이크 과정을 거친다. 이 과정은 실시간 반응이 중요한 서비스에서는 불필요한 지연 요소가 될 수 있다.</li>
<li><b>변경과 확장이 어려운 구조</b><br />TCP는 IETF(Internet Engineering Task Force)에서 표준화한 프로토콜로, 기능을 변경하거나 확장하려면 광범위한 합의와 긴 테스트 과정을 필요로 한다. 이로 인해 진화 속도가 매우 느리다.</li>
<li><b>HOL(Head-of-Line) 블로킹 문제</b><br />TCP는 패킷의 순서를 보장하기 위해 모든 데이터를 순차적으로 처리한다. 따라서 중간에 하나의 패킷이 손실되거나 지연되면, 그 이후의 모든 패킷 처리도 함께 지연되는 문제가 발생한다. 특히 패킷 손실이 잦은 네트워크 환경에서는 성능 저하가 두드러진다.</li>
<li><b>네트워크 전환 시 연결 끊김 문제</b><br />사용자가 Wi-Fi에서 LTE로 이동하는 등 IP 주소나 포트가 바뀌는 경우, 기존 TCP 연결이 끊기게 된다. 이로 인해 연결을 다시 수립해야 하며, 통신의 연속성이 깨질 수 있다.</li>
</ul>
<hr contenteditable="false" />
<p>이러한 TCP의 한계를 보완하기 위해 등장한 방식이 <b>Reliable UDP(RUDP)</b>다. UDP의 가벼움은 유지하면서, 패킷 손실에 대한 보완을 통해 보다 실시간성 있는 통신이 가능하도록 한다.</p>
<h4>Reliable UDP의 특징과 동작 과정</h4>
<p>화상 회의나 온라인 게임처럼 <b>속도가 가장 중요하고, 약간의 패킷 손실은 감내할 수 있는 상황</b>에서는 TCP보다 UDP가 더 적합한 경우가 많다. 하지만 UDP는 신뢰성이 보장되지 않는 프로토콜이기 때문에, 그대로 사용하기엔 무리가 있다. 그렇다면 TCP처럼 복잡한 기능(순서 보장, 3-way 핸드셰이크, 재전송, 흐름 제어 등)을 모두 갖추지 않으면서, UDP를 더 신뢰성 있게 만들 수 없을까? 그 해답이 바로 Reliable UDP (RUDP)다. RUDP는 기존 UDP의 가벼운 구조를 유지하면서, 다음과 같은 <b>신뢰성 기능</b>을 추가한다:</p>
<ol>
<li>수신한 패킷에 대한 <b>응답(ACK) 전송</b></li>
<li><b>패킷 손실 감지</b> 후 <b>재전송 요청</b></li>
<li><b>순서 보장</b>을 위한 시퀀스 넘버 활용</li>
</ol>
<p><figure class="imageblock widthContent"><span><img height="918" src="https://blog.kakaocdn.net/dn/beEnT9/btsN6RNeGCL/PYXxVrSeSUF4xkX9SYEkOK/img.png" width="990" /></span></figure>
</p>
<ol>
<li><b>캐릭터 A가 첫 번째로 총을 발사한다.</b><br />이 동작은 패킷 0에 담겨 클라이언트로 전송된다. 클라이언트는 패킷을 받고 서버에 ACK(수신 확인 응답)를 보낸다.</li>
<li><b>캐릭터 B가 두 번째로 총을 발사하지만, 해당 패킷 1은 전송 중에 손실된다.</b></li>
<li><b>캐릭터 C가 세 번째로 총을 발사한다.</b><br />이 동작은 패킷 2로 클라이언트에 전달된다. 클라이언트는 패킷 1이 누락된 것을 감지하고, 패킷 2를 일단 버퍼에 저장한 뒤 서버에 ACK를 보낸다.</li>
<li><b>서버는 패킷 1에 대한 ACK가 오지 않자, 일정 시간이 지난 후 재전송한다.</b><br />클라이언트는 이 패킷을 받고 ACK를 보낸 후, 버퍼에 저장해 두었던 패킷 2도 함께 처리한다.</li>
</ol>
<hr contenteditable="false" />
<p>이처럼 RUDP는 손실된 패킷을 감지하고 재전송하며, 순서를 보장한다. 하지만 TCP만큼 복잡하지 않기 때문에, 고속의 실시간 응답이 필요한 상황에 적합하다. 대표적인 구현체로는 QUIC 프로토콜이 있다. QUIC은 TCP의 여러 한계를 해결하고 RUDP 기반으로 만들어졌으며, 현재 HTTP/3의 기반 프로토콜로 자리 잡고 있다.</p>
<h3>SMTP</h3>
<p>SMTP는 이메일을 서버 간에 전송하기 위한 표준 프로토콜로 MAIL, RCPT, DATA와 같은 명령어를 통해 이메일을 전달한다.</p>
<h4>이메일 전송 흐름</h4>
<p><figure class="imageblock widthContent"><span><img height="772" src="https://blog.kakaocdn.net/dn/bDqlvm/btsN6SFogEs/F7zyS75SdSSufJLgnK0Ia1/img.png" width="978" /></span></figure>
</p>
<ol>
<li><b>Alice가 이메일을 작성해 전송한다.</b><br />Alice는 아웃룩 클라이언트에 로그인한 뒤, 이메일을 작성하고 "보내기" 버튼을 누른다. 이때 이메일은 SMTP(Simple Mail Transfer Protocol)를 사용해 아웃룩의 메일 서버로 전송된다.<span><br /></span></li>
<li><b>수신자 메일 서버 찾기<br /></b>아웃룩 메일 서버는 이메일 수신자인 Bob의 메일 서버 주소를 찾기 위해 DNS서버<span>를 조회한다. </span>이때 사용되는 정보가 MX(Mail Exchanger) 레코드<span>로 수신자의 이메일 도메인(Gmail 등)이 사용하는 SMTP 서버의 주소를 알려준다. </span>DNS 조회가 끝나면, 아웃룩 서버는 Gmail의 SMTP 서버로 이메일을 전송한다.</li>
<li><b>Gmail 서버에 이메일 저장</b><br />Bob의 Gmail SMTP 서버는 이메일을 수신한 후, 이를 Bob의 메일함에 저장한다. 이제 Bob이 이메일을 확인할 준비가 된 상태다.</li>
<li><b>Bob이 이메일 수신</b><br />Bob이 Gmail 계정에 로그인하면, 클라이언트는 IMAP 또는 POP 프로토콜을 통해 <span>Gmail 서버에서 새 이메일을 가져온다.</span><span><span><br /><span style="color: #333333;"><b>- POP (Post Office Protocol)<br /></b>원격 메일 서버에서 이메일을 수신하고 로컬 기기로 다운로드하는 방식이다. 다운로드 후 이메일은 보통 서버에서 삭제된다. 이로 인해 해당 이메일은 다운로드된 기기에서만 접근이 가능하다. 단점은 이메일을 보기 위해 전체 메시지를 포함한 첨부파일까지 모두 다운로드해야 하기 때문에, 대용량 메일일 경우 시간이 오래 걸릴 수 있다.</span></span></span><br /><span style="color: #333333;"><b>- IMAP (Internet Message Access Protocol)</b></span><br />이메일을 서버에 유지하며, 사용자가 메일을 열람할 때만 해당 데이터를 다운로드하는 방식이다. 때문에&nbsp;여러 기기에서 동일한 이메일을 확인할 수 있으며, 개인 이메일 계정에서 가장 널리 사용되는 방식이다.</li>
</ol>
<h3>Ping</h3>
<p>Ping 명령어는 한 서버에서 다른 서버까지의 응답 시간을 측정하거나 두 호스트 간의 네트워크 연결 여부를 확인하기 위해 사용한다.</p>
<h4>동작 과정</h4>
<p>ping 명령어는 <b>ICMP(Internet Control Message Protocol)</b> 위에서 동작한다. ICMP는 <b>네트워크 계층</b>에 속한 프로토콜로, 주로 네트워크 진단 및 제어를 위해 사용된다. ICMP에는 다양한 유형의 메시지가 존재하는데, ping 명령어에서는 주로 Echo Request(에코 요청) 와 Echo Reply(에코 응답) 메시지를 사용한다.</p>
<p><figure class="imageblock widthContent"><span><img height="820" src="https://blog.kakaocdn.net/dn/bgj3km/btsN4MT5Ujc/yjpk3kjv12u79RuECEReoK/img.png" width="1072" /></span></figure>
</p>
<ol>
<li>호스트 A가 ICMP Echo Request 메시지(타입 = 8)를 보낸다.<br />이 요청에는 일반적으로 1부터 시작하는 시퀀스 번호가 포함되며, IP 헤더에 출발지 및 목적지 IP 주소가 함께 캡슐화되어 전송된다.</li>
<li>호스트 B는 이 요청을 수신하면, 동일한 시퀀스 번호를 포함한 ICMP Echo Reply 메시지(타입 = 0)를 되돌려 보낸다.</li>
<li>호스트 A는 Echo Reply 메시지를 수신하면, 시퀀스 번호를 기반으로 요청과 응답을 매칭시키고, <span style="color: #333333; text-align: left;"><span style="color: #333333; text-align: left;">두 타임스탬프(T1: 요청을 보낸 시간, T2: 응답을 받은 시간)</span></span><span style="color: #333333; text-align: left;">를 이용하여<span>&nbsp;</span></span>왕복 시간(RTT, Round-Trip Time)<span style="color: #333333; text-align: left;">을 계산한다.</span></li>
</ol>
<p>&nbsp;</p>
<h2>참고 자료</h2>
<p><a href="https://blog.bytebytego.com/p/network-protocols-behind-server-push?utm_source=publication-search" rel="noopener&nbsp;noreferrer" target="_blank">https://blog.bytebytego.com/p/network-protocols-behind-server-push?utm_source=publication-search</a></p>
<figure contenteditable="false" id="og_1747733105703"><a href="https://blog.bytebytego.com/p/network-protocols-behind-server-push?utm_source=publication-search" rel="noopener" target="_blank">
<div class="og-image">&nbsp;</div>
<div class="og-text">
<p class="og-title">Network Protocols behind Server Push, Online Gaming, and Emails</p>
<p class="og-desc">Welcome to this new issue where we expand our exploration into essential network protocols and their various applications. The focus here is on understanding how different protocols shape the way we communicate and interact over the Internet. We will dive</p>
<p class="og-host">blog.bytebytego.com</p>
</div>
</a></figure>
<p>&nbsp;</p>