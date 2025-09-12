<h2>메시징이란</h2>
<p>메시징은 발신자와 수신자를 분리하는 방법이다. 메시징 시스템은 가장 단순하게 말하면 시스템의 한 부분에서 다른 부분으로 정보를 전달하는 것을 의미한다. 생산자가 메시지를 보내면, 브로커가 이를 저장하고 라우팅하며, 소비자가 가져가서 필요한 작업을 수행한다. 이 과정에서 서비스는 다른 서비스가 작업을 완료할 때까지 기다리지 않는다. 단지 메시지를 전달하고 곧바로 다음 일을 진행한다. 메시지는 브로커에 안전하게 저장되며, 수신자는 준비가 되었을 때 메시지를 처리한다. 만약 수신자가 실패하더라도 메시지는 대기 상태로 남아 있다. 하지만 모든 메시징 시스템이 동일한 동작 방식을 제공하는 것은 아니다. 이번 글에서는 자주 나타나는 3가지 주요 패턴에 대해 살펴 보고자한다.</p>
<h3>핵심 개념</h3>
<p><figure class="imageblock widthContent"><span><img height="1000" src="https://blog.kakaocdn.net/dn/etDB7O/btsQrsfpwX9/97pWO4MswCgeUvjycW8BhK/img.png" width="1895" /></span></figure>
</p>
<h4>생산자, 소비자, 브로커</h4>
<p><b>생산자(Producer)</b> 는 메시지를 생성하는 모든 구성 요소를 말한다. 보통 큐나 토픽에 메시지를 푸시하며, 사용자 활동을 발행하는 프론트엔드 서비스일 수도 있고, 이후 처리할 작업을 큐에 넣는 백엔드 서비스일 수도 있다.</p>
<p><b>소비자(Consumer)</b> 는 시스템에서 메시지를 가져와 처리한다. 데이터베이스에 기록을 추가하거나, 알림을 전송하거나, 백그라운드 계산을 시작하는 등의 역할을 수행한다.</p>
<p>이들 사이에는 메시지를 수신하고, 저장하며, 전달하는 <b>브로커(Broker)</b> 또는 중개자가 존재한다. 브로커가 없다면 생산자와 소비자는 직접 연결되어 강하게 결합된다. 그러나 브로커를 두면 양쪽이 분리되어 서로 독립적으로 확장할 수 있고, 장애에도 더 강한 구조를 만들 수 있다.</p>
<h4>메세지</h4>
<p>메시지는 메시징 모델에 따라 <b>큐(queue)</b> 또는 <b>토픽(topic)</b> 으로 구성된다. <b>큐</b>는 <b>단일 소비 라인</b>을 의미하며, <b>하나의 메시지는 단일 소비자만</b> 가져갈 수 있다. 반면 <b>토픽</b>은 <b>여러 구독자에게 동시에 메시지를 전달</b>하며, <b>각 구독자는 메시지의 복사본을 개별적으로 수신</b>한다.</p>
<p>또한 처리를 확장하기 위해 토픽을 <b>파티션(partition)</b> 으로 나누기도 한다. 각 파티션은 독립적인 로그로 동작하며, 여러 소비자가 병렬로 메시지를 읽을 수 있도록 한다. 그러나 파티셔닝은 메시지 순서 보장과 로드 밸런싱 측면에서 트레이드오프를 발생시킨다. 즉, 확장성과 병렬 처리를 얻는 대신 전체 메시지 순서를 보장하기 어려워지거나, 특정 파티션에 부하가 집중될 수 있다.</p>
<p>&nbsp;</p>
<p>&lt;전달 의미론&gt;</p>
<p>모든 메시지가 반드시 보낸 그대로 도착하는 것은 아니다. 네트워크 단절, 서비스 크래시, 재시도와 같은 상황은 메시지 전달에 영향을 준다. 이로 인해 일반적으로 다음과 같은 세 가지 전달 보장이 정의된다.</p>
<ul>
<li><b>At-most-once</b><br />메시지가 최대 한 번만 전달되는 의미이다. 메시지가 한 번 전달되거나 아예 전달되지 않을 수 있다. 재시도가 없기 때문에 빠르지만, 메시지 손실의 위험이 크다.</li>
<li><b>At-least-once</b><br />메시지가 최소 한 번 이상 전달되는 의미이다. 재시도가 발생하기 때문에 메시지가 중복 전달될 수 있다. 안정적인 접근 방식이지만, 소비자가 중복 처리를 고려해야 한다.</li>
<li><b>Exactly-once</b><br />메시지가 정확히 한 번만 전달되는 의미이다. 각 메시지가 손실이나 중복 없이 한 번만 처리된다. 가장 강력한 보장이지만 구현이 까다롭다.</li>
</ul>
<p><figure class="imageblock widthContent"><span><img height="1000" src="https://blog.kakaocdn.net/dn/AkqCB/btsQtRLrMz5/wM7D2AY0REmPNleti0euO1/img.png" width="1517" /></span></figure>
</p>
<p>많은 시스템은 현실적인 이유로 최소 한 번(At-least-once) 보장에 만족하고, 중복 문제는 다운스트림 단계에서 멱등성 처리를 통해 해결한다.</p>
<h2>메시징 패턴</h2>
<h3>메시지 큐</h3>
<p>메시지 큐는 하나의 생산자가 작업을 큐(Queue)에 넣고 하나 이상의 소비자가 이를 가져와 처리하는 방식이다. 이 모델은 <b>Point-to-Point 메시징</b>이라고 하며, 백엔드 시스템에서 핵심 인프라 중 하나로 널리 사용된다. 서비스가 작업을 백그라운드로 오프로드하거나, 작업자를 여러 인스턴스에 분배하거나, 다운스트림의 느린 처리로부터 동기 실행을 분리해야 할 때 메시지 큐가 적절한 해결책이 된다.</p>
<h4>작동 방식</h4>
<p><figure class="imageblock widthContent"><span><img height="1000" src="https://blog.kakaocdn.net/dn/bxqzVG/btsQuqUhl4E/ZuADRyL41c8kBLljRmosvK/img.png" width="2053" /></span></figure>
</p>
<p>생산자는 큐에 메시지를 푸시한다. 각 메시지는 일반적으로 하나의 개별 작업 단위를 의미한다. 소비자는 큐에서 메시지를 가져와 처리하며, 작업이 끝나면 처리 완료(Ack)를 시스템에 알릴 수 있다. 즉, 메시지는 큐에 저장되고 준비된 소비자가 천천히 가져가 처리하는 구조이다.</p>
<h4>주요 동작 특성</h4>
<ul>
<li><b>단일 소비(Single Consumer)</b><br />각 메시지는 한 번만 처리된다. 한 소비자가 처리하면 큐에서 사라지고, 다른 소비자는 가져갈 수 없다.</li>
<li><b>FIFO(First In, First Out)</b><br />메시지가 들어온 순서대로 처리되는 것을 목표로 한다. 다만, 시스템 설정에 따라 순서가 완전히 보장되지 않을 수 있다.</li>
<li><b>재시도 및 재전달(Retry &amp; Redelivery)</b><br />소비자가 메시지를 처리하는 도중 오류가 발생하거나 서비스가 잠시 중단되며, 큐는 메시지를 다시 시도할 수 있다.</li>
<li><b>DLQ(Dead Letter Queue)</b><br />메시지가 여러 번 시도에도 계속 실패하면, 별도의 큐(DLQ)로 보내져 검토 및 수동 처리가 가능하다.</li>
</ul>
<h4>사용 사례</h4>
<p>메시지 큐는 각 메시지가 한 번씩만 처리되어야 하는 작업에 적합하다. 예를 들어, 웹 요청에서 꼭 필요한 작업이 아닌 일부 처리를 큐에 넣으면 메인 요청 처리 속도를 느리지 않게 유지할 수 있다. 여러 소비자가 같은 큐에서 메시지를 가져가도록 구성하면 작업을 여러 인스턴스가 나눠 처리하므로 서버를 쉽게 늘려 처리량을 높일 수 있다. 또한, 사용자가 갑자기 몰리거나 작업이 한꺼번에 생겨도, 큐가 잠시 메시지를 저장해 천천히 처리하도록 도와 뒤쪽 시스템(메시지를 받아 처리하는 서비스)이 과부하에 걸리는 것을 방지할 수 있다.</p>
<hr contenteditable="false" />
<p>하지만 메시지 큐를 사용할 때는 몇 가지 주의할 점이 있다. 먼저 <b>스케일링(Scaling)</b> 문제를 들 수 있다. 단순히 소비자(Worker)를 늘리면 작업 처리 속도가 빨라질 것 같지만, 메시지 순서가 깨질 수 있다. 따라서 작업 순서가 중요한 경우, 여러 소비자가 동시에 메시지를 처리할 때는 순서를 유지하기 위한 추가적인 조정이나 관리가 필요하다.</p>
<p>또한 <b>재시도(Retries)</b>에도 주의해야 한다. 실패한 메시지를 재시도할 때 지연 처리나 중복 제거가 제대로 이루어지지 않으면 시스템이 중복 작업으로 과부하될 수 있다.</p>
<p>마지막으로 <b>Stuck Consumers&nbsp;문제</b>도 있다. 소비자가 메시지 처리 중 멈춰 있으면 큐가 새로운 메시지를 처리하지 못해 전체 처리 흐름이 지연될 수 있다. 따라서 반드시 <b>헬스 체크(Health Check)와 타임아웃을 설정</b>해, 문제가 발생한 소비자를 감지하고 조치할 수 있도록 해야 한다.</p>
<h3>Pub/Sub</h3>
<p>Pub/Sub은 Publish/Subscribe(발행/구독)의 약자이며, <b>하나의 메시지를 여러 수신자가 동시에 받는 메시징 패턴</b>을 의미한다. 즉, Pub/Sub의 핵심은 하나의 메시지에 여러 구독자가 존재한다는 점이다. 메시지 큐가 단일 작업자에게 메시지를 전달하는 것과 달리, Pub/Sub 시스템은 메시지를 여러 독립적인 구독자에게 Fan-out(분배)한다. 각 구독자는 메시지의 복사본을 받아, 다른 구독자의 처리 속도와 상관없이 자신만의 속도로 처리할 수 있다.</p>
<h4>작동 방식</h4>
<p><figure class="imageblock widthContent"><span><img height="1000" src="https://blog.kakaocdn.net/dn/dNG0co/btsQtHvx3a2/Is4e9e24oGJVCGo93S0LQ0/img.png" width="1480" /></span></figure>
</p>
<p>생산자는 특정 토픽(Topic)에 메시지를 발행한다. 토픽은 브로드캐스트 채널처럼 동작하며, 해당 토픽을 구독하고 있는 모든 구독자는 메시지의 복사본을 받는다.</p>
<p>일부 Pub/Sub 시스템(Kafka)에서는 소비자 그룹을 구성할 수 있다. 한 그룹 내의 여러 소비자는 메시지를 나눠 가져가면서 그룹 내에서 로드를 공유한다. 동시에 여러 그룹이 존재하면 각 그룹은 독립적으로 모든 메시지를 받아 처리한다. 즉, 모든 그룹은 메시지를 브로드캐스트로 받고, 그룹 내에서는 메시지를 큐처럼 나눠 처리하는 하이브로드 구조가 만들어진다.</p>
<h4 style="color: #000000; text-align: start;">사용 사례와 트레이드오프</h4>
<p>Pub/Sub 패턴은 여러 시스템이 동일한 이벤트에 반응해야 할 때 특히 유용하다. 예를 들어 사용자가 사진을 업로드하면, 한 발행으로 다음 작업이 동시에 트리거 된다. 이처럼 이벤트 브로드캐스팅(Event Broadcasting)이 필요한 경우 Pub/Sub 패턴이 적합하다. 또한 실시간 알림, 뉴스 피드 업데이트, 주식 시세 실시간 반영 등 즉시 반응이 필요한 시스템에도 유용하게 사용된다.</p>
<hr contenteditable="false" />
<p>다만, Pub/Sub 패턴을 사용할 때 아래와 같은 몇 가지 고려사항이 존재한다.&nbsp;</p>
<p>먼저 <b>백프레셔(Backpressure)</b> 문제를 들 수 있다. 구독자 중 한 명이 메시지를 처리하는 속도가 느려지면, 시스템은 메시지를 잠시 버퍼에 저장할지, 전체 처리를 느리게 조정할지, 아니면 일부 메시지를 버릴지를 결정해야 한다. 이러한 선택은 메시지의 내구성과 지연에 직접적인 영향을 준다.</p>
<p>또한 <b>재생 가능성(Replayability)</b> 문제도 존재한다. 모든 Pub/Sub 시스템이 이전 메시지를 저장하는 것은 아니며, 일부 시스템은 실시간으로만 메시지를 전달하고 이미 전달된 메시지는 다시 가져올 수 없다. 이 때문에 구독자가 잠시 오프라인 상태였다면 놓친 메시지를 재생할 수 없는 경우가 생길 수 있다.</p>
<p>마지막으로 <b>전달 보장(Delivery guarantees)</b>도 시스템마다 크게 다르다. 메시지가 한 번도 전달되지 않을 수 있는 &ldquo;최대 한 번(At-most-once)&rdquo;부터, 중복 전달될 수 있는 &ldquo;최소 한 번(At-least-once)&rdquo;, 그리고 정확히 한 번만 전달되는 &ldquo;정확히 한 번(Exactly-once)&rdquo;까지 다양하다. 따라서 적절한 브로커를 선택하고 설정을 신중하게 해야 시스템의 신뢰성과 안정성을 확보할 수 있다.</p>
<p>&nbsp;</p>
<h3>이벤트 스트림</h3>
<h4>이벤트 스트림이란?</h4>
<p>이벤트 스트림은 시스템에서 일어난 일을 시간 순서대로 기록한 로그라고 생각하면 쉽다. 한 번 기록된 이벤트는 절대 바뀌거나 삭제되지 않는다. 즉, 큐나 Pub/Sub처럼 단순히 메시지를 전달하는 것이 아니라, 전체 사건의 히스토리를 그대로 보존하는 구조다. 소비자는 지금 발생하는 이벤트만 볼 수도 있고 필요하다면 이전 이벤트를 다시 읽어 재생할 수도 있다. 새로운 이벤트는 항상 스트림의 끝에 추가되며, 스트림 안에서는 순서가 유지된다. 또한 이벤트는 처리 후에도 사라지지 않고 정해진 기간 동안 계속 저장되어 필요할 때 언제든 참조할 수 있다.</p>
<h4>동작 방식</h4>
<p><figure class="imageblock widthContent"><span><img height="822" src="https://blog.kakaocdn.net/dn/b8OCsh/btsQtcJj5uF/HFbPuY5yzTr0XEXGDrk9i1/img.png" width="1280" /></span></figure>
</p>
<p>이벤트 스트림은 시스템에서 발생한 사건을 <b>시간 순서대로 기록한 불변 로그</b>로 구성된다. 생산자(Producer)가 새로운 이벤트를 생성하면, 이벤트는 항상 스트림의 끝에 추가된다. 중요한 점은 이벤트가 소비된 후에도 <b>삭제되지 않고</b>, 정해진 보존 기간 동안 또는 무기한으로 저장된다는 것이다.</p>
<p>&nbsp;</p>
<p>이벤트 스트림에서는 소비자는 자신이 지금까지 어디까지 데이터를 읽었는지를 offset이라는 위치 정보로 관리한다. Offset은 스트림 내 이벤트의 순서를 나타내는 번호로 각 소비자가 마지막으로 처리한 이벤트가 몇 번째인지 기록하는 역할을 한다. 이를 통해 여러 소비자가 동시에 같은 스트림을 읽더라고 서로의 속도나 처리 상태에 영향을 주지 않고 독립적으로 데이터를 처리할 수 있다. 또한, 소비자가 버그로 인해 일부 데이터를 잘못 처리했거나 새로운 로직을 적용해야 하는 경우, offset을 원하는 지점으로 되돌려 과거 이벤트를 다시 읽고 처리할 수 있다. 마찬가지로 소비자가 중간에 다운되거나 네트워크 문제로 메시지를 놓쳤을 때도, 마지막으로 기록된 offset부터 다시 시작할 수 있어 데이터 손실 없이 복구가 가능하다. 이처럼 Offset 덕분에 각 소비자는 자신만의 속도와 필요에 맞게 데이터를 읽고 처리할 수 있으며 이벤트 스트림의 핵심 장점은 재생 가능성과 유연성을 구현할 수 있다.</p>
<hr contenteditable="false" />
<p>이러한 구조 덕분에 이벤트 스트림은 여러 가지 장점을 제공한다. 먼저 과거 데이터를 다시 처리하여 버그를 수정하거나 시스템 상태를 복원할 수 있는 재처리 가능성이 있다. 또한 언제 어떤 일이 발생했는지 정확히 추적하여 문제의 원인을 파악할 수 있는 Time-travel Debugging도 가능하다. 비즈니스 활동을 완전하게 기록할 수 있어 규제 준수, 분석, 사기 탐지 등 다양한 목적으로 활용할 수 있는 데이터 감사(Audit) 기능도 갖추고 있다. 더 나아가 여러 소비자가 독립적으로 데이터를 읽을 수 있어, 서로 조율하지 않아도 자신만의 속도로 스트림을 처리할 수 있다. 결국 이벤트 스트림은 단순히 메시지를 전달하는 것을 넘어, 데이터의 이력과 맥락을 기반으로 시스템을 보다 유연하고 신뢰성 있게 운영할 수 있도록 해준다.</p>
<h2>정리</h2>
<p>모든 메시징 패턴은 모든 상황에서 완벽하게 작동하지 않는다. 각각은 서로 다른 목적을 가지고 있으며, 각기 다른 장점과 한계, 그리고 트레이드오프를 가진다. 이러한 차이를 이해하는 것이 취약한 시스템과 안정적이고 적응 가능한 시스템을 구분하는 핵심이다.&nbsp;</p>
<table border="1" style="border-collapse: collapse; width: 100%; height: 110px;">
<tbody>
<tr style="height: 20px;">
<td style="width: 25%; height: 20px;"><b>항목</b></td>
<td style="width: 25%; height: 20px;"><b>메시지 큐 (Queue)&nbsp;</b></td>
<td style="width: 25%; height: 20px;"><b>Pub/Sub</b></td>
<td style="width: 25%; height: 20px;">이벤트 스트림 (Event Stream)</td>
</tr>
<tr style="height: 18px;">
<td style="width: 25%; height: 18px;"><b>전달 모델</b></td>
<td style="width: 25%; height: 18px;">Point-to-Point, <br />하나의 생산자 -&gt; 하나의 소비자</td>
<td style="width: 25%; height: 18px;">일대다,<br />하나의 메시지가 여러 구독자에게 브로드캐스트</td>
<td style="width: 25%; height: 18px;">일대다 가능,<br />소비자가 지속 로그에서 자신만의 속도로 읽음</td>
</tr>
<tr style="height: 18px;">
<td style="width: 25%; height: 18px;"><b>순서 보장</b></td>
<td style="width: 25%; height: 18px;">FIFO -&gt; 순서 보장 추구<br />재시도 시 순서가 깨질 수도 있음</td>
<td style="width: 25%; height: 18px;">구독자 간 전역 순서 보장 없음</td>
<td style="width: 25%; height: 18px;">각 파티션 내에서 강한 순서 보존</td>
</tr>
<tr style="height: 18px;">
<td style="width: 25%; height: 18px;"><b>재생 가능성</b></td>
<td style="width: 25%; height: 18px;">확인되면 메시지 폐기, 외부 저장소 없으면 재생 불가</td>
<td style="width: 25%; height: 18px;">브로커 구현에 따라 다름 -&gt; 일부 제한적 재생 가능</td>
<td style="width: 25%; height: 18px;">일급 재생 가능,<br />offset/timestamp로 되감기</td>
</tr>
<tr style="height: 18px;">
<td style="width: 25%; height: 18px;"><b>지연 시간 &amp; 처리량</b></td>
<td style="width: 25%; height: 18px;">낮은 지연, 단일 소비자와 최적, 병렬 소비자로 처리량 확장 가능, 순서 제한 있음</td>
<td style="width: 25%; height: 18px;">낮은 지연, 보통 처리량<br />느린 구독자/대규모 팬아웃 시 HOL 블로킹, 백프레셔 발생 가능</td>
<td style="width: 25%; height: 18px;">높은 처리량, 초당 수백만 메세지 가능<br />버퍼링/배칭/파티셔닝에 따라 지연 증가 가능</td>
</tr>
<tr style="height: 18px;">
<td style="width: 25%; height: 18px;"><b>장점</b></td>
<td style="width: 25%; height: 18px;">단순, 신속, 단일 작업 처리 적합</td>
<td style="width: 25%; height: 18px;">이벤트 브로드캐스트, 여러 구독자 동시 처리 가능</td>
<td style="width: 25%; height: 18px;">재생 가능성, 감사 가능, 다수 소비자 독립적 처리 가능</td>
</tr>
</tbody>
</table>
<h2>참고자료</h2>
<p><a href="https://blog.bytebytego.com/p/messaging-patterns-explained-pub" rel="noopener&nbsp;noreferrer" target="_blank">https://blog.bytebytego.com/p/messaging-patterns-explained-pub</a></p>
<figure contenteditable="false" id="og_1757505396567"><a href="https://blog.bytebytego.com/p/messaging-patterns-explained-pub" rel="noopener" target="_blank">
<div class="og-image">&nbsp;</div>
<div class="og-text">
<p class="og-title">Messaging Patterns Explained: Pub-Sub, Queues, and Event Streams</p>
<p class="og-desc">What happens when the downstream service is overloaded? Or slow? Or down entirely?</p>
<p class="og-host">blog.bytebytego.com</p>
</div>
</a></figure>
<p>&nbsp;</p>