<h2>외부 연동 서비스에, 장애가 발생했어요!</h2>
<p>외부 서비스에 과부하가 발생해 응답을 제대로 주지 못하고 있는 상황이라고 생각해보자. 연동 서비스가 정상화되기 전까지는 요청을 보내도 계속 에러만 발생하게 될 것이다. 또한, 읽기 타임아웃이 발생할 때까지 대기하느라 응답 시간도 길어지게 될 것이다.</p>
<p><figure class="imageblock widthContent"><span><img height="580" src="https://blog.kakaocdn.net/dn/clCfL6/btsPzTDPb0y/Oz5R7ptHS27lzP4FyjB8mk/img.png" width="1834" /></span></figure>
</p>
<p>그림과 같은 상황일 때, A 서비스는 B 서비스에 요청을 보내지 않고 바로 에러를 응답하는 것이 낫다. 이렇게 하면 B 서비스의 문제가 A 서비스에 주는 영향(응답 시간 증가, 처리량 감소)을 줄일 수 있다. 또한 사용자 입장에서도 수 초를 대기하다가 에러 화면을 보는 것보다는, 빠르게 에러 화면을 보는 편이 낫다. 즉, <b>연동 서비스가 장애 상황일 때는, 연동 대신 바로 에러를 응답하고, 정상화되었을 때 연동을 재개하면 연동 서비스의 장애가 주는 영향을 줄일 수 있다.</b></p>
<hr contenteditable="false" />
<p>이처럼, 우리는 외부 서비스의 장애가 우리 시스템으로 전파되지 않도록 장애를 격리할 필요가 있다. 장애가 발생한 서비스를 탐지하고 요청을 보내지 않도록 차단하려면 어떻게 해야할까?</p>
<h2>서킷 브레이커 (Circuit Breaker)</h2>
<p>외부 서비스에 의한 문제를 방지하기 위해 등장한 것이 바로 <b>서킷 브레이커 패턴</b>이다. 서킷 브레이커 패턴은, 클라이언트 측면에서 장애를 방지하기 위한 도구로, <b>실패할 수 있는 작업을 계속 시도하지 않도록 방지</b>한다.</p>
<h3>동작 원리</h3>
<p>서킷 브레이커는 누전 차단기와 비슷하게 동작한다. 과전류가 흐르면 차단기가 내려가 전기를 끊는 것처럼, 서킷 브레이커도 과도한 오류가 발생하면 연동을 중지시키고 바로 에러를 응답한다.&nbsp;</p>
<h4>서킷 브레이커의 3가지 상태</h4>
<p><figure class="imageblock widthContent"><span><img height="706" src="https://blog.kakaocdn.net/dn/eA9seX/btsPAB3yW31/ubG4mzPyEsS723vaAl0Lik/img.png" width="1430" /></span><figcaption>서킷 브레이커의 3가지 상태</figcaption>
</figure>
</p>
<p>서킷 브레이커는 닫힘(closed), 열림(Open), 반 열림(Half-Open)의 3가지 상태를 갖는다.</p>
<p>서킷 브레이커는 닫힘 상태로 시작한다. 닫힘 상태는 모든 것이 정상힌 상황으로 모든 요청을 연동 서비스에 전달한다. 외부 연동 과정에서 오류가 발생하기 시작하면, 지정한 임계치를 초과했는지 확인한다. 실패 건수가 임계치를 초과하면 서킷 브레이커는 열림 상태가 된다.</p>
<p>보통 임계치는 다음 조건 중 하나를 사용한다.</p>
<ul>
<li><b>시간 기준 오류 발생 비율</b> (10초 동안 오류 비율이 50% 초과)</li>
<li><b>개수 기준 오류 발생 비율</b> (100개 요청 중 오류 비율이 50% 초과)</li>
</ul>
<p>열림 상태가 되면 연동 요청은 수행하지 않고, 바로 에러 응답을 리턴한다. 열림 상태는 지정된 시간 동안 유지되고 이 시간이 지나면 반 열림 상태로 전환된다. 반 열림 상태가 되면 일부 요청에 한해 연동을 시도한다. 일정 개수 또는 일정 시간 동안 반 열림 상태를 유지하며, 이 기간동안 연동에 성공하면 닫힘 상태로 복귀한다. 반대로 연동에 실패하면 다시 열림 상태로 전환되어 연동을 차단한다. 이러한 상태 변경은 자동으로 수행되며, 상태 전이를 위한 시간들은 시스템 내부에서 관리되므로 대부분 타임아웃과 모니터링 시스템을 제공해준다.</p>
<h3>장점 및 필요성</h3>
<h4>장애 감지 및 격리</h4>
<p>만약, 장애가 발생한 서비스를 호출한다면 요청이 타임아웃만큼 대기하게 되고, 쓰레드와 메모리 및 CPU 등의 자원을 점유하게 된다. 이는 결국 시스템 리소스를 부족하게 만들어 장애를 유발할 수 있다. 장애가 발생한 것은 다른 서비스인데 우리 서비스로 장애가 전파될 수 있는 것이다! 서킷 브레이커 패턴은 장애가 발생한 서비스를 감지하고, 더 이상 요청을 보내지 않도록 차단함으로써 장애를 격리시켜 준다. 따라서, 장애가 발생한 기능 외의 다른 기능들은 동작하게 하여 시스템의 안정성을 높일 수 있다.</p>
<h4>빠른 실패 및 장애 서비스로의 부하 감소</h4>
<p><span style="color: #333333; text-align: start;">또한, 서킷 브레이커가 열려 있는 동안은 연동 서비스에 요청이 전달되지 않기 때문에 연동 서비스가 과부하 상황에서 벗어날 수 있는 기회도 생긴다. 만약, 일시적인 트래픽 과부하로 서킷이 열리면, 대상 벤더사(외부 서비스)에게 회복할 수 있는 쿨다운 시간을 주어 장애가 지속되는 것을 방지할 수 있는 것이다. 이처럼 서킷 브레이커는 문제 상황이 감지되면 해당 기능을 더 이상 실행되지 않고 바로 실패로 처리한다. 이처럼 실패를 빠르게 감지하고, 문제가 있는 기능을 실행하지 않고 중단시키는 방식을<span>&nbsp;</span></span><b>빠른 실패(fail fast)</b>라고 한다. 빠른 실패는 장애가 발생한 기능에 부하가 더해지는 것을 방지할 뿐 아니라, 불필요한 자원 낭비를 줄여 전체 서비스의 안정성을 유지하는 데도 도움이 된다.</p>
<h4>자동 시스템 복구</h4>
<p>서킷 브레이커는 요청이 차단되면 해당 서비스가 정상인지 주기적으로 검사한다. 그리고 해당 서비스가 복구되었다면 차단이 해제되고, 정상적으로 요청을 보내게 된다. 이러한 부분들은 시스템이 자동으로 해주므로 개발자들이 신경쓰지 않아도 된다. 대부분 타임아웃 등을 위한 모니터링 기능까지 제공해주기 때문에 대시보드를 통해 전체 시스템들의 연동 현황까지 모니터링이 가능하다.</p>
<h2>Java 진영의 서킷 브레이커 라이브러리 - Resilience4j</h2>
<p>국내 대부분의 서비스들은 스프링 MVC 기반으로 되어 있다. 스프링 MVC는 멀티 쓰레드 기반으로 동작하므로 장애가 있는 서비스를 호출하면 쓰레드 점유에 의한 응답 지연이 발생하기 쉽다. 때문에 장애가 전파되기도 쉬운데, 서킷 브레이커를 사용하면 빠르게 장애가 발생한 서버로의 요청을 차단하고 이를 해결할 수 있다. 물론, 서킷 브레이커를 도입한다면 서킷 브레이커의 상태 및 히스토리 관리 등을 위한 추가 비용이 발생한다. 하지만, 서킷 브레이커는 안정적인 서비스 운영을 위한 필수 패턴이므로 반드시 적용해야 한다.</p>
<hr contenteditable="false" />
<p style="color: #000000; text-align: start;">Resilience4j는 함수형 프로그래밍으로 설계된 경량 장애 허용 라이브러리로, 서킷 브레이커 패턴을 위해 사용된다. <span style="color: #666666;">Resilience4j는 함수형 기반의 라이브러리인만큼 내부적으로 Java의 Future로 요청을 실행한다. </span><span style="color: #000000; text-align: start;">Resilience4j</span>를 적용하면 외부 서비스에 장애가 발생해도 우리의 시스템은 장애가 전파되지 않고 계속 작동할 수 있다. 넷플릭스에서 만든 Hystrix라는 다른 서킷 브레이커 라이브러리도 존재하지만 deprecated 되었으로, Resilience4를 사용하면 된다.</p>
<p style="color: #000000; text-align: start;">&nbsp;</p>
<p style="color: #000000; text-align: start;"><span style="color: #000000; text-align: start;">Resilience4j에는 여러 가지 코어 모듈이 존재한다. 그 중 CircuitBreaker 모듈에 대해서 살펴보고자 한다.</span></p>
<blockquote><b>공식문서</b></blockquote>
<p style="color: #000000; text-align: start;"><span style="color: #666666; text-align: left;"><a href="https://resilience4j.readme.io/docs/circuitbreaker">https://resilience4j.readme.io/docs/circuitbreaker</a></span></p>
<figure contenteditable="false" id="og_1753439700369"><a href="https://resilience4j.readme.io/docs/circuitbreaker" rel="noopener" target="_blank">
<div class="og-image">&nbsp;</div>
<div class="og-text">
<p class="og-title">CircuitBreaker</p>
<p class="og-desc">Getting started with resilience4j-circuitbreaker</p>
<p class="og-host">resilience4j.readme.io</p>
</div>
</a></figure>
<h3 style="color: #000000; text-align: start;">CircuitBreaker 모듈</h3>
<p><figure class="imageblock widthContent"><span><img height="426" src="https://blog.kakaocdn.net/dn/1xZGU/btsPAyML0TO/Zfdh9J9HaFVamTmjf78rt1/img.png" width="498" /></span></figure>
</p>
<p>일반적인 서킷 브레이커의 기본 상태(CLOSED, OPEN, HALF_OPEN)에 더해 DISABLED와 FORCED_OPEN 이라는 특수한 상태 2개가 추가되어 있다.</p>
<p>CircuitBreaker 모듈은 호출 결과를 저장하고 집계하기 위해 <span style="color: #ef5369;"><b>슬라이딩 윈도우를 사용</b></span>한다. 마지막 N번의 호출 결과를 기반으로 하는 <b>count-based sliding window(횟수 기반 슬라이딩 윈도우)</b>와 마지막 N초의 결과를 기반으로 하는 <b>time-based window(시간 기반 슬라이딩 윈도우)</b>가 있다.</p>
<p>느린 호출율과 호출 실패율이 서킷 브레이커에 설정된 임계값보다 크거나 같다면 CLOSED 상태에서 OPEN으로 상태가 변경된다. 이때, 특정 예외에 대해서만 실패로 간주하고 싶다면 예외 목록을 정의해주면 된다. (기본적으로는 발생한 모든 예외에 대해서 실패로 간주한다.) OPEN 상태가 되면 CallNotPermittedException을 발생시킨다. 특정 시간이 지나 HALF_OPEN 상태로 바뀌고 설정된 수의 요청만을 허용하고 나머지는 동일하게 예외를 발생시킨다. 그리고 동일하게 느린 호출율과 호출 실패율에 따라 서킷의 상태를 OPEN 또는 CLOSED 상태로 변경한다.</p>
<p>앞서 언급했듯, Resilience4j는 <b>DISALBED</b>와 <b>FORCED_OPEN</b>이라는 2가지 특별한 상태를 추가로 지원한다. DISABLED는 서킷 브레이커를 비활성화하여 항상 요청을 허용하는 상태이며, FORED_OPEN은 강제로 서킷을 열어두어 항상 요청을 거부하는 상태이다.</p>
<h4>Thread-Safety</h4>
<p>서킷 브레이커 모듈은 Thread-safe하게 설계되어 있다. 서킷 브레이커의 상태는 AtomicReference에 저장되며 atomic 연산을 사용하여 상태를 업데이트한다. 슬라이딩 윈도우에서 요청을 기록하고 스냅샷을 읽는 작업 역시 동기적으로 처리된다. 이를 통해 서킷 브레이커는 <span style="color: #000000;"><b>원자성이 보장</b></span>되며, 특정 시점에 하나의 쓰레드만이 서킷 브레이커의 상태나 슬라이딩 윈도우를 업데이트할 수 있게 된다.</p>
<p>&nbsp;</p>
<p>하지만 여기서 중요한 점은 서킷브레이커가 <span style="color: #000000;"><b>함수 호출 자체를 동기화하지는 않는다</b> </span>것이다. 만약 함수 호출까지 동기화한다면 이는 심각한 성능 저하와 병목 현상을 일으키게 될 것이다. 예를 들어, 슬라이딩 윈도우 크기가 15로 설정되어 있고 20개의 쓰레드가 CLOSED 상태에서 동시에 호출 허가를 요청하는 상황을 가정해보자. 이 경우 모든 20개의 스레드는 동시에 실제 함수를 호출하게 된다. <b>슬라이딩 윈도우의 크기는 동시에 실행 가능한 요청의 수를 의미하는 것이 아니다.</b> 단순히 최근 N번의 호출에 대한 성공과 실패 통계를 관리하는 용도일 뿐이다. 동시에 실행 수를 제한하고 싶다면 이는 Bulkhead 모듈에서 지원하느 기능을 사용해야 한다.&nbsp;</p>
<p>&nbsp;</p>
<p>즉, 서킷 브레이커는 "언제 호출을 허용하거나 차단할지"에 대한 판단만 담당하며, "몇 개의 요청을 동시에 처리할지"는 별개의 관심사이다.</p>