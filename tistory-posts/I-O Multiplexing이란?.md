<h3>I/O 작업이란</h3>
<p>멀티플렉싱에 대한 내용에 앞서, 먼저 I/O 작업이 무엇인지 어떻게 동작하는지부터 알아야 한다. I/O란, input/output의 약자로 말 그대로 데이터의 입출력을 의미한다. I/O에도 여러 종류가 존재하는데, 대표적으로 네트워크(socket) I/O, 파일 I/O 등등이 있다.</p>
<p>I/O 작업은 사용자 공간에서 직접 수행할 수 있기 때문에 커널에 I/O 작업을 요청하고 응답을 받는 구조다. 응답을 어떤 순서로 받는지(synchronous/asynchronous), 어떤 타이밍에 받는지(blocking/non-blocking)에 따라 여러 모델로 분류된다.</p>
<p><a href="https://wing1008.tistory.com/59" rel="noopener" target="_blank">이전의 포스트</a>에서 동기와 비동기에 대해서 자세히 다루었으니, 이번 글에서는 간단하게 짚고 넘어가도록 하겠다.</p>
<h4>Synchronous, 동기</h4>
<p><figure class="imageblock widthContent"><span><img height="183" src="https://blog.kakaocdn.net/dn/cl7pQx/btsQruwlSUp/nFIuahzdYtN1pGlPvQks41/img.png" width="622" /></span></figure>
</p>
<p>모든 I/O 요청-응답이 순서대로 처리되는 것을 말한다. 작업 완료는 사용자 공간(user space)에서 판단하고 다음 작업을 언제 요청할지 결정한다. 이러한 동기 방식은 pipeline을 준수하는 구조에서 효율적이다.</p>
<blockquote>작업의 순서를 보장한다는 것은 '현재 작업의 응답'을 받는 시점과 '다음 작업을 요청'하는 시점을 맞추는 일이다. 다음 작업이 있다는 것 자체가 순서가 있다는 것을 의미하며, 이전 작업이 완료되기 전까진 다음 작업이 수행되지 않는다.</blockquote>
<h4 style="color: #000000; text-align: start;">Asynchronous, 비동기</h4>
<p><figure class="imageblock widthContent"><span><img height="180" src="https://blog.kakaocdn.net/dn/cwbwRt/btsQqJOjklF/uqwqK60L9uUgDMzrW7KHK0/img.png" width="645" /></span></figure>
</p>
<p>비동기 방식은 동기 방식과 달리 작업의 순서가 보장되지 않는다. 각 작업은 독립적으로 수행되며 작업이 완료되면 커널 공간(kernal space)에서 사용자 공간으로 콜백 함수, 이벤트, 시그널 등의 형태로 완료를 통보한다. 즉, 커널이 작업의 완료를 관리하고 사용자에게 알리는 역할을 담당한다. 이러한 특성 때문에 각 작업들이 독립적으로 동시에 요청할 수 있다.</p>
<h4>Blocking, 블로킹</h4>
<p>블로킹은 요청한 작업이 모두 완료될 때까지 기다렸다가 완료될 때 응답과 결과를 반환받는다. 즉, 요청한 작업이 완료될 때까지 해당 스레드나 프로세스가 기다리는 방식이다. 블로킹 방식은 구현이 단순하고 이해하기 쉽지만, 요청한 작업이 오래 걸릴 경우 CPU 자원이 효율적으로 사용되지 못하는 단점이 존재한다.</p>
<h4>Non-Blocking, 논블로킹</h4>
<p>논블로킹 방식은 작업 요청 후 결과를 기다리지 않고 다른 작업을 수행하다가, 나중에 필요할 때 해당 작업의 결과를 처리하는 방식이다. 요청한 스레드가 대기하지 않기 때문에, CPU 자원을 효율적으로 사용할 수 있다. 즉, 작업 완료를 기다리는 동안 스레드나 프로세스가 block되지 않고 다른 일을 수행할 수 있는 구조이다.</p>
<hr contenteditable="false" />
<p>Synchronous/Asynchronous, Blocking/Non-Blocking&nbsp; 이 2가지 개념을 보통 혼동하곤 하는데, 이 2개는 엄연히 다른 독립적인 개념이다. <b>동기/비동기</b>는 <b>작업의 순서와 완료 판단 위치 관점에서 구분</b>된다. 반면, <b>블로킹/논블로킹</b>은 스레드나 프로세스가 <b>요청한 작업 결과를 기다리는지 여부의 관점</b>에서 구분된다. 블로킹은 결과를 기다리며 스레드가 멈추지만, 논블로킹은 결과를 기다리지 않고 다른 작업을 수행하다가 나중에 결과를 처리한다.&nbsp;</p>
<p>즉, 동기/비동기와 블로킹/논블로킹은 서로 독립적인 개념이기 때문에, 동기 방식이라도 논블로킹으로 구현할 수 있고, 비동기 방식이라도 블로킹처럼 동작할 수 있다.</p>
<h3>I/O model의 종류</h3>
<p><figure class="imageblock alignCenter"><span><img height="317" src="https://blog.kakaocdn.net/dn/b6gm5f/btsQsmECENU/8k4R2w1v5sSjTQakPJpcB0/img.png" width="504" /></span><figcaption>IBM의 I/O 모델 분류</figcaption>
</figure>
</p>
<p>위 사진은 IBM에서 제시한 대표적은 I/O 모델 분류이다. 이때 I/O Mutiplexing이 Blocking모델로 분류되어 있는데, 이 부분에 대해서는 다양한 의견이 존재한다. 실제로 구현 방식이나 해석하는 관점에 따라 Blocking으로 볼 수도 있고 Non-Blocking으로 볼 수도 있다. 또한 내부적으로는 동기적인 방식으로 동작하기도 하며, 사용하는 기법에 따라 세부 로직이나 이벤트 알림 방식도 달라진다. 그렇기 때문에 Mutiplexing을 단순히 "비동기 블로킹 방식"이라고만 정의하는 것은 다소 무리가 있다고 한다.</p>
<h4>Synchronous Blocking I/O</h4>
<p>아래 사진은 가장 흔하게 생각할 수 있는 동기 블록킹 모델의 흐름도이다.</p>
<p><figure class="imageblock alignCenter"><span><img height="340" src="https://blog.kakaocdn.net/dn/cxsjgS/btsQpIblVKG/4O2kgwGthlrxGmKfLRxuM0/img.gif" width="538" /></span></figure>
</p>
<p>&nbsp;user space에 존재하는 프로세스는 커널에게 I/O를 요청하는 함수를 호출(system call)한다. 이 순간부터 프로세스는 kernel이 작업 결과를 반환할 때까지 중단된 채 대기(block)한다. 이때 대기 중인 프로세스는 cpu를 점유하지 않고 단순히 kernel의 응답만 기다린다. 이 과정에서 만약 signal이 발생하면 system call이 중단될 수는 있으나, 그렇지 않다면 kernel이 작업을 완료하는 순간 데이터는 사용자 공간의 버퍼로 전달되고 프로세스는 다시 실행(unblock)되어 반환된 데이터를 처리할 수 있다.</p>
<p>해당 방식은 <span style="color: #333333; text-align: start;"><span>&nbsp;</span>I/O 요청이 적은 서비스에는 적합할 수 있지만, Spring과 같은 멀티 스레드 환경에서는&nbsp;</span>요청이 늘어날 때마다 별도의 스레드를 생성하므로 스레드 수가 많아질수록 컨텍스트 스위칭(context switching) 비용이 커져 성능이 떨어지게 된다. 또한 블로킹된 프로세스는 CPU를 사용하지 않고 I/O 작업의 완료만을 기다리게 되는데, I/O 작업은 cpu 자원을 거의 사용하지 않기 때문에 시스템 자원 활용 효율도 좋지 않다.</p>
<h4>Synchronous Non-Blocking I/O</h4>
<p>모든 소켓은 논블로킹 모드로 전환할 수 있다. 논블로킹을 도입하면 입출력 명령이 즉시 실행되지 않는다.</p>
<p>&nbsp;</p>
<p><figure class="imageblock alignCenter"><span><img height="340" src="https://blog.kakaocdn.net/dn/bfGX42/btsQrVAvDEc/9fN8mEa6arCMggWGRLA5tK/img.gif" width="474" /></span></figure>
</p>
<p><span style="color: #333333; text-align: start;">해당 논 블로킹 소켓으로 I/O system call을 하게 되면 데이터가 없을 경우 오류를 반환한다.<span style="color: #666666;">(구현에 따라 오류 코드 EWOULDBLOCK 또는 EAGAIN와 같은 특수한 값을 반환하는 경우도 존재한다) </span></span>즉, 커널이 I/O 작업을 끝낼 때까지 기다리지 않고 현재 상태를 즉시 반환하기 때문에 프로세스가 블록되지 않는다.</p>
<p>&nbsp;</p>
<p>Busy-Waiting은 이러한 논블로킹 방식을 구현하는 가장 단순한 형태이다. 프로세스는 I/O 작업의 완료 여부를 확인하기 위해 accept(), read(), send()와 같은 시스템 콜을 반복적으로 호출하는데 이러한 반복적인 질의 과정을 폴링(Polling)이라고 부른다. 이러한 방식은 I/O 작업이 완료될 때까지 CPU를 계속 사용하게 되므로 <b>'바쁜 대기(Busy-Waiting)'</b>라고 부른다. 또한 적절한 polling 주기가 필요한데 주기가 너무 길 경우에는 실제 데이터는 다 준비되었음에도 처리가 지연될 수 있고, 반대로 주기가 너무 짧다면 kernel 입장에서는 의미 없는 return을 자주 해줘야하기 때문에 불필요한 오버헤드와 I/O 지연을 유발할 수 있다.</p>
<p>&nbsp;</p>
<p>따라서 논블로킹 I/O를 효율적으로 사용하기 위해서는 Busy-waiting 대신 <b>I/O 멀티플렉싱</b> 같은 고급 기술을 활용하여 여러 I/O 작업을 동시에 관리하고, 데이터가 준비될 때까지 기다렸다가 처리한다.</p>
<h4 style="color: #000000; text-align: start;">Asynchronous Non-Blocking I/O</h4>
<p><figure class="imageblock alignCenter"><span><img height="340" src="https://blog.kakaocdn.net/dn/bp7bsH/btsQtbbyu5k/UXzfb7giXjogDLf3sRM3H0/img.gif" width="481" /></span></figure>
</p>
<p style="color: #333333; text-align: start;">해당 환경에서 프로세스가 system call로 I/O 요청을 전달한 이후, 커널이 작업을 전적으로 책임지고 처리한다. 즉, I/O 작업의 전체 제어권이 커널에게 넘어간다. 프로세스는 I/O처리에 신경 쓰지 않고 있다가 작업이 완료되면 kernel로부터 signal, thread 기반 callback 등의 방식으로 결과를 통보받는다. 데이터 복사까지 커널이 모두 완료한 후 결과를 알려주기 때문에 프로세스는 결과를 받자마자 바로 처리할 수 있다. 또한 응답이 오기 전까지 프로세스는 다른 연산을 수행할 수 있으며, read()같은 호출이 완료될 때까지 블로킹되지 않는다.</p>
<h3>I/O Multiplexing (멀티 플렉싱)</h3>
<h4>I/O 다중화</h4>
<p><b>I/O 관점에서 다중화</b>란, <b>한 프로세스가 여러 개의 파일(또는 소켓)을 효율적으로 관리하는 기법</b>을 말한다. 여기서 '파일'은 단순히 디스크에 저장된 파일 뿐 아니라, 네트워크 소켓이나 파이프처럼 프로세스가 kernel과 데이터를 주고 받을 수 있도록 다리 역할을 하는 인터페이스를 의미한다.</p>
<p>예를 들어 server-client 환경이라면 하나의 server에서 여러 개의 소켓을 관리하면서 동시에 여러 클라이언트의 요청을&nbsp; 처리할 수 있어야 한다. 이때 프로세스는 각 소켓에 직접 접근하는 대신 파일 디스크립터(File Descriptor, FD)라는 추상적인 핸들을 사용하여 커널에 접근한다. 결국 <b>I/O Multiplexing의 핵심은 여러 파일 디스크립터를 어떻게 효율적으로 감시할 것인가</b>에 달려있다.</p>
<p>여기서 어떤 상태로 대기하냐에 따라 select, poll, epoll, kqueue 등등 다양한 기법들이 존재한다. 각각의 방식은 "어떤 FD가 읽기/쓰기 가능 상태가 되었는지"를 프로세스에 알려주는 방법과 대기 방식에서 차이를 보인다. 하지만, select와 poll의 경우에는 성능면에서 좋지 않기 때문에 잘 사용하지 않는다고 한다. 때문에 현재의 리눅스 환경에서는 주로 epoll을 사용하고 BSD 계열 시스템에서는 kqueue가 활용된다고 한다. 이런 함수들에 대해서는 다음 포스트에서 다뤄보고자 한다.</p>
<h4>Aysnchronous blocking I/O <span style="color: #666666;">(IBM 분류 기준)</span></h4>
<p><figure class="imageblock alignCenter"><span><img height="340" src="https://blog.kakaocdn.net/dn/cu6ig3/btsQp0bPYB3/khrKuu17L1fC2l6lWrqad0/img.gif" width="541" /></span></figure>
</p>
<p>위 구조는 kernel이 I/O 요청을 받아 처리를 시작함과 동시에 프로세스에게 미완료 상태를 반환하고 user process는 데이터가 준비됐다는 알람이 올 때까지 대기하는 모습을 보여주고 있다. 이때 특징을 살펴보면 아래와 같다.</p>
<p>&nbsp;</p>
<p>1. I/O Multiplexing System Call은 블로킹된다.</p>
<p>이때, 프로세스에서의 read, write 같은<b> I/O 작업 자체가 block 되는 것이 아니라</b> select, poll 같은 Mutiplexing 관련 <b>system call이 블로킹</b>된다. 쉽게 풀어서 설명하면, Busy-waiting 방식에서는 프로세스가 계속해서 read() 시스템 콜을 호출하며 "데이터가 왔니?"라고 묻지만, 멀티플렉싱 방식에서는 select(), epoll()과 같은 시스템 콜을 한 번 호출하고 "데이터가 올 때까지 알려주지 않아도 돼. 나 그냥 잠시 쉬고 있을게"라고 커널에게 말하는 것과 같다.</p>
<p>커널은 여러 소켓을 대신 감시하다가 그중 하나라도 데이터가 준비되면 프로세스를 깨우고 프로세스는 어떤 소켓에 데이터가 왔는지 확인하고 해당 소켓에 대해서만 read()를 호출한다.</p>
<p>&nbsp;</p>
<p>2. 데이터 복사(I/O 동작) 자체는 여전히 동기적이다.</p>
<p>멀티플렉싱 시스템 콜이 블로킹되는 것과 별개로, 데이터를 실제로 읽어오는 read() 시스템 콜은 여전히 동기적이고 블로킹된다. read()를 호출하는 순간, 커널은 커널 버퍼에 있는 데이터를 사용자 공간 버퍼로 복사하는데, 이 복사 작업이 끝날 때까지 프로세스는 기다려야 한다. I/O 멀티플렉싱의 목적은 read()를 호출하기 전, 데이터가 준비되었는지 확인하는 단계에서 프로세스를 효율적으로 대기시키는 것이다. 이는 마치 여러 개의 소켓을 일일이 확인하는 대신, 커널에게 데이터가 오면 알려달라고 부탁하는 것과 유사하다. 데이터가 도착했다는 알림을 받으면, 비로소 데이터를 read하는 작업을 수행하는 것이다.</p>
<hr contenteditable="false" />
<p>이러한 특성 때문에 I/O Multiplexing을 블로킹 방식으로 보기도 한다. 왜냐하면 멀티플렉싱 시스템 콜 함수 자체가 블록되기 때문이다. 반면 논블로킹 방식이라는 의견도 있는데 단일 스레드가 여러 I/O 작업을 블로킹되지 않고 효율적으로 관리할 수 있기 때문이다. Busy Waiting처럼 cpu를 낭비하지 않으면서도 여러 I/O 채널을 동시에 감시할 수 있다는 점에서 논블로킹적인 이점을 갖는다고 볼 수 있다.</p>
<h4>&nbsp;</h4>