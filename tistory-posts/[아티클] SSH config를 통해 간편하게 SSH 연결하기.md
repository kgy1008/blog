<p>저희는 지금까지 EC2에 접속하기 위해서 터미널에 아래와 같이 명령어를 입력해야만 했습니다.</p>
<pre class="kotlin"><code>ssh -i "pem 키 위치" ubuntu@'퍼블릭 DNS 주소'
</code></pre>
<p>너무 귀찮치 않으신가요?? 오늘은 이를 간편하게 접속할 수 있는 방법에 대해 알아보고자 합니다.</p>
<h3>1. .ssh 폴더 생성</h3>
<pre class="arduino"><code>mkdir ~/.ssh (이미 존재한다면 이 단계는 넘어가셔도 됩니다!)
</code></pre>
<h3>2. 키페어 파일 가져오기</h3>
<p>EC2에 접속하기 위해 필요한 pem키!! 다들 기억하시죠? 현재 저장되어 있는 pem키의 위치를 홈 디렉터리 내의 .ssh 폴더로 이동시켜 줄게요.</p>
<pre class="arcade"><code>mv 'pem키이름.pem' ~/.ssh/
</code></pre>
<p>그 후, 해당 pem키에 대해 권한을 설정해줘야겠죠? 아래 명령어를 입력합시다.</p>
<pre class="lsl"><code>chmod 400 "pem키 이름"
</code></pre>
<h3>3. Config 파일 설정하기</h3>
<p>그럼 이제 config 파일을 만들어봅시다.</p>
<pre class="routeros"><code>vim config
</code></pre>
<p>이미 존재한다면, 존재하는 파일을 편집해주시면 됩니다. 이제 config 파일을 설정해봅시다.</p>
<pre class="routeros"><code>Host hostname
	HostName XXX.XXX.XXX.XXX
	User ubuntu
	Port 22
	IdentityFile ~/.ssh/XXXXXXX.pem
</code></pre>
<p>먼저 <b>Host</b>는 접속할 호스트의 이름을 설정하는 부분입니다. 우리가 접속할 서버의 별칭을 정하는 것이라 생각하시면 되겠네요!</p>
<p>HostName은 탄력적 IP 주소, 즉 접속할 호스트의 IP 주소를 입력해주세요.</p>
<p>User 부분은 접속할 EC2 사용자명을 입력하는 곳입니다.</p>
<p>Port는 SSH로 접속할 포트번호를 말합니다. 기본값이 22이기 때문에 기본값을 사용한다면 이 부분은 생략하셔도 됩니다.</p>
<p>IdentityFile은 호스트 접속 시 사용되는 키 경로를 입력해주는 곳입니다. 저희는 aws에서 발급받은 pem 키의 경로를 입력해주면 되겠죠</p>
<h3>4. SSH 접속하기</h3>
<p>그럼 이제 모든 단계는 끝났습니다. 간단하죠?? 이제 접속해봅시다.</p>
<pre class="nginx"><code>ssh 'hostname'
</code></pre>
<p>접속 방법은 아주아주 간단합니다. 아까 저희가 config 파일에서 hostname, 즉 별칭을 정해줬죠? ssh 뒤에 그 별칭을 적어주면 정말 간단하게 SSH에 접속할 수 있게 됩니다.</p>
<h3>Q) 여러 호스트를 작성하고 싶은데 어떻게 해야하나요?</h3>
<p>간단합니다. 그저 config 파일에 이어서 작성해주시면 됩니다!</p>
<p><figure class="imageblock alignCenter"><span><img height="501" src="https://blog.kakaocdn.net/dn/ctRD9k/btsMMyQsV0w/IJLaJrLmNaMP8sEfiML0E1/img.png" width="649" /></span></figure>
</p>
<p>이런식으로 말이죠! 간단하죵??</p>
<p>&nbsp;</p>