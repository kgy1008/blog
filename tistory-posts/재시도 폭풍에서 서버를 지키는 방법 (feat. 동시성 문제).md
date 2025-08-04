<h2>들어가며..</h2>
<p>2025 네이버 신입 공채 1차 면접 때 있었던 일이다.<span>&nbsp;</span><span>이제 막 1달 된 따끈따끈한 일이다. </span>당시의 면접관의 질문과 내 답변을 복기해보면 아래와 같다.</p>
<blockquote> 이력서 써놓은 신 것을 봤는데, 지금 동시성 제어를 낙관적 락으로 해결하셨다고 되어 있네요. 그러면 지금 100명이 동시에 좋아요를 눌렀을 때 발생할 수 있는 문제점이 뭐가 있을까요?<br />️<br /> ️ 일단 지금 저의 로직으로는, 낙관적 락을 걸었기 때문에 만약 동시에 수정을 하면 버전 충돌로 인해서 자동으로 다시 시도하는 로직이 구현되어 있습니다. 근데 이제 100명이 동시에 시도를 하면 성공한 1명 빼고 나머지 99명은 똑같이 재시도 로직을 돌게 될 텐데 그 재시도를 도는 과정에서 부하가 일어날 수 있을 것 같습니다.<br /><br />  네 그 부하라는 거는 어떤 부하가 있을까요?<br />️<br /> ️ 재시도 로직이 반복적으로 실행되면서 애플리케이션 레벨에서는 CPU 사용량이 증가할 것이고 또 99개의 스레드가 모두 동시에 활성되기 때문에 컨텍스트 스위칭 오버헤드라던가 커넥션 풀 고갈 문제가 생길 수도 있을 것 같습니다.<br /><br />  그럼 지금 이걸 어떻게 해결할 수 있을까요?<br />️<br /> ️ 제가 생각했을 때 좋아요 수 같은 경우는 사실 이 서비스의 약간 핵심적인 비즈니스 기능은 아니다 보니까 우선 재시도 횟수를 제한을 둬서 사용자에게 에러 메세지를 전달하거나 오류 로그를 보면서 저희가 수작업으로 처리해도 괜찮을 것 같습니다. 현재도 재시도 로직은 3회로 제한한 상태입니다. 또한 실시간 성은 떨어지지만, 메세지 큐 같은 것을 도입하여 좋아요를 비동기적으로 처리할 수도 있을 것 같습니다.<br /><br />  네네.. 혹시 다른 방법은 없을까요? 재시도를 동시에 시키는 것이 아니라.. 혹시 백오프 라는 단어는 알고 계신가요?<br />️<br /> ️ 네, 일정 시간 후에 시도를 하게끔 해가지고 한 번 실패를 하면 일정 시간 후에 다시 재시도하고..<br /><br />  네네 그럼 지수 백오프 방식을 도입해서 해결한다고 했을 때, 이제 구현을 한다고 하면 어떻게 할 수 있을까요?<br />️<br />...</blockquote>
<p><span style="background-color: #ffffff; color: #1f2328; text-align: start;">한번도 현재의 재시도 로직에 대해 더 깊이 생각해본 적이 없었기 때문에, 즉석에서 현재의 문제점과 개선 방안들에 묻는 질문들에 대해 당황할 수 밖에 없었다. 이후의 답변들도 스스로 만족스럽지 못했고, 결국 1차 면접에서 탈락했다ㅠ. 그래도 인생 첫 면접이 네이버였다는 점만으로도 뜻깊었고, 많은 것을 배워갈 수 있었던 시간이었다.</span></p>
<p><span style="background-color: #ffffff; color: #1f2328; text-align: start;">소마 기획 심사의 준비도 어느정도 마치기도 했고 소마 프로젝트를 본격적으로 들어가면 바쁠 것 같아 한 달이 지난 지금, 이전 프로젝트인 '한끼족보&rsquo;의 재시도 로직에 지수 백오프 전략을 직접 도입하여 로직을 보완해보았다.</span></p>
<h2>백오프(Backoff)</h2>
<p>백오프(backoff)는, 시스템에서 실패한 작업을 재시도할 때, 무작정 빠르게 반복하지 않고 재시도 간격을 점차 늘려가면서 서버나 리소스에 과부하가 걸리지 않도록 하는 전략을 말한다. 쉽게 설명하면 실패하면 바로 다시 시도하는 것이 아니라 처음에는 짧은 시간을 기다리고, 계속 실패할 때마다 점점 기다리는 시간을 더 길게 늘려서 재시도하는 것이다.</p>
<p>대표적인 백오프 방식으로는 크게 2가지로 나눌 수 있다.</p>
<p><b>1. 고정 백오프(Fixed backoff) -</b> 매번 동일한 간격으로 재시도 시도</p>
<p><b>2. 지수 백오프(Exponential backoff) 방식 -</b> 재시도할 때마다 대기 시간을 점진적으로 증가시킨다.</p>
<h3>재시도</h3>
<p>재시도 로직을 설계할 때는 보통 '재시도 횟수'와 '재시도 간격'을 고려해야 한다. 이 2가지 조건을 잘 조절하면 실패 상황에서도 안정적인 처리를 기대할 수 있다.</p>
<h4>재시도 폭풍(Retry Strom) 안티 패턴</h4>
<p>재시도는 실패한 작업의 성공 가능성을 높이는 좋은 방법이지만, <b>무분별하거나 일정 간격으로 반복되는 재시도는 오히려 시스템에 더 큰 부하를 줄 수 있다.</b><br />특히 동일한 간격으로 재시도를 수행하면, 실패했던 여러 요청들이 <b>동시에 다시 요청되면서 트래픽이 몰리는 현상</b>이 발생할 수 있다. 이를 <b>재시도 폭풍(Retry Storm)</b>이라고 부른다.</p>
<p>이러한 문제를 해결하기 위해 <b>지수 백오프</b> 전략을 활용할 수 있다. 재시도 요청 간의 간격을 점차 늘림으로써 <b>요청 시점을 자연스럽게 분산시키고</b>, 결과적으로 시스템의 부하를 줄일 수 있다.</p>
<h2>개선해보자</h2>
<h3>기존 로직의 문제점</h3>
<p>기존 로직을 살펴보면 다음과 같다.</p>
<pre class="sql"><code>// @Retry 어노테이션 정의
public @interface Retry {

    int maxAttempts() default 3;

    int backoff() default 100;
}

// AOP 구현 로직
public class RetryAspect {

    @Around("@annotation(retry)")
    public Object retryOptimisticLock(final ProceedingJoinPoint joinPoint, final Retry retry) throws Throwable {
        for (int attempt = 0; attempt &amp;lt; retry.maxAttempts(); attempt++) {
            try {
                return joinPoint.proceed();
            } catch (ObjectOptimisticLockingFailureException e) {
                Thread.sleep(retry.backoff());
            }
        }
        throw new ConflictException(HeartErrorCode.HEART_COUNT_CONCURRENCY_ERROR);
    }
}</code></pre>
<p>Thread.sleep(retry.backoff())는 항상 같은 간격이므로, 여러 요청들이 동시에 실패하면 같은 타이밍에 다시 시도를 하게 될 것이고 이는 서버에 재시도 폭풍으로 인한 부하가 존재하게 된다.</p>
<h3>개선 로직</h3>
<pre class="sql"><code>@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Retry {

    int maxAttempts() default 3;

    // 초기 백오프 시간
    int backoff() default 100;

    // 최대 백오프 시간 - 지수 백오프가 너무 커지는 것을 방지
    int maxBackoff() default 1000;

    // 백오프 배수
    double multiplier() default 1.5;

    // 지터(랜덤 요소) 활성화
    boolean enableJitter() default true;
}

// AOP 구현 로직
@Slf4j
@Order(Ordered.LOWEST_PRECEDENCE - 1)
@Aspect
@Component
public class RetryAspect {

    private final Random random = new Random();

    @Around("@annotation(retry)")
    public Object retryOptimisticLock(final ProceedingJoinPoint joinPoint, final Retry retry) throws Throwable {
        int currentBackoff = retry.initialBackoff();

        for (int attempt = 1; attempt &amp;lt;= retry.maxAttempts(); attempt++) {
            try {
                return joinPoint.proceed();
            } catch (ObjectOptimisticLockingFailureException e) {
                if (attempt == retry.maxAttempts()) {
                    log.warn("최대 재시도 횟수 {} 도달. 메서드: {}", retry.maxAttempts(), joinPoint.getSignature().getName());
                    throw new ConflictException(HeartErrorCode.HEART_COUNT_CONCURRENCY_ERROR);
                }

                int sleepTime = calculateBackoffTime(currentBackoff, retry);
                log.info("낙관적 락 충돌 발생. 재시도 {}/{}, {}ms 후 재시도. 메서드: {}",
                        attempt, retry.maxAttempts(), sleepTime, joinPoint.getSignature().getName());

                Thread.sleep(sleepTime);

                // 다음 시도를 위한 백오프 시간 계산 (지수 백오프)
                currentBackoff = calculateNextBackoff(currentBackoff, retry);
            }
        }

        throw new ConflictException(HeartErrorCode.HEART_COUNT_CONCURRENCY_ERROR);
    }

    private int calculateBackoffTime(int currentBackoff, Retry retry) {
        if (!retry.enableJitter()) {
            return currentBackoff;
        }

        // &amp;plusmn;25% 범위의 지터 추가
        double jitterFactor = 0.75 + (random.nextDouble() * 0.5); // 0.75 ~ 1.25
        return (int) (currentBackoff * jitterFactor);
    }

    private int calculateNextBackoff(int currentBackoff, Retry retry) {
        int nextBackoff = (int) (currentBackoff * retry.multiplier());
        return Math.min(nextBackoff, retry.maxBackoff());
    }
}</code></pre>
<h4>지수 백오프 적용</h4>
<p>기존에는 고정된 시간 간격(backoff())으로 재시도하도록 구성했지만, 실패가 반복될수록 간격을 점점 늘려가도록 개선하였다.<br />currentBackoff 값을 유지하면서, multiplier()에 따라 다음 재시도 시간 간격을 지수적으로 증가시키는 방식이다.</p>
<p><code>currentBackoff = calculateNextBackoff(currentBackoff, retry);</code><br />해당 메서드에서 재시도 간격이 multiplier에 따라 점차 늘어나며, 또한 최대 백오프 시간을 설정하여 maxBackoff 값 이상으로는 증가하지 않도록 제한하였다.</p>
<h4>Jitter(지터) 사용</h4>
<p>지터(Jitter) 전략은 재시도 요청이 모두 동시에 몰려서 서버에 부하가 걸리는 현상(=요청 폭주)을 방지하기 위한 <b>무작위(random)</b> 요소를 말한다. 주로 재시도(backoff) 로직과 함께 사용된다. 즉, Jitter 전략은 백오프 시간에 무작위성(Randomness) 을 부여해서 재시도 타이밍을 분산시킨다.</p>
<pre class="angelscript"><code>private int calculateBackoffTime(int currentBackoff, Retry retry) {
    if (!retry.enableJitter()) {
        return currentBackoff;
    }

    double jitterFactor = 0.75 + (random.nextDouble() * 0.5); // 0.75 ~ 1.25
    return (int) (currentBackoff * jitterFactor);
}
</code></pre>
<p>재시도 타이밍이 일정한 간격으로 고정되면, 여러 요청이 동시에 실패했을 경우 재시도 또한 동시에 몰릴 수 있다. 이를 방지하기 위해 Jitter 전략을 도입하였다. 하지만 Jitter는 의도적인 무작위성을 부여하는 방식이기 때문에, 범위가 지나치게 클 경우 응답 지연이 발생하거나 tail latency가 증가할 수 있다. 이에 따라 재시도 간격에 &plusmn;25% 범위의 무작위 편차만 적용하여, 과도한 랜덤성이 발생하지 않도록 제한하였다.</p>
<p><br />이로써, 동일한 currentBackoff 값을 기준으로 하더라도 요청마다 재시도 간격이 미묘하게 달라지게 되며, 이로 인해 요청 타이밍이 자연스럽게 분산되고 서버에 갑작스럽게 몰리는 부하를 줄일 수 있다.</p>
<h2>다른 방법은 없었을까?</h2>
<p>좋아요 수를 집계하는데 낙관적 락 방식을 사용한 이유는 근본적으로 DB 수준에서 동시성 문제(Race Condition)이 발생했기 때문이다. 해당 문제를 인지했을 당시, 나는 DB 차원에서 동시성을 제어할 수 있는 방법으로 크게 두 가지, 비관적 락(Pessimistic Locking) 과 낙관적 락(Optimistic Locking) 을 알고 있었다. 분산락에 대한 존재도 알고 있었지만, 분산 환경이 아니었고 구현 복잡도 대비 효과가 크지 않다고 판단하여 제외하였다.&nbsp;</p>
<p>&nbsp;</p>
<p>낙관적 락을 선택했던 가장 큰 이유는, 좋아요 수와 관련된 충돌이 빈번하게 발생하지 않을 것이라 판단했기 때문이다. 또한 외부 시스템과의 연동이나 비동기 처리 로직이 없었기 때문에, DB 단에서 레코드 잠금을 발생시키는 비관적 락 방식보다는 낙관적 락이 성능 면에서 우수하다고 판단하였다.</p>
<p>&nbsp;</p>
<p>하지만 이후 동시성 문제를 보다 간단하고 성능적으로 해결할 수 있는 방법으로 <b>증분 쿼리(Incremental Query)</b> 가 있다는 사실을 알게 되었다. 증분 쿼리는 기존처럼 애플리케이션에서 데이터를 먼저 조회한 후 값을 수정해서 저장하는 방식이 아니라, <b>쿼리 자체에서 값을 직접 원자적으로 수정하는 방식</b>이다. 예를 들어 좋아요 수를 증가시키는 경우, 아래와 같이 작성할 수 있다.</p>
<pre class="sql" id="code_1754285904550"><code>update Store set hearCount = hearCount + 1 where id = ?</code></pre>
<p><span style="color: #333333; text-align: start;">한가지 조심해야 할 점은 증분 쿼리는<span>&nbsp;</span></span><b>DB에 따라 원자적 연산이 아닐 수도 있다</b>는 점이다. 따라서 증분 쿼리를 사용할 때는 사용하는 DB에서 원자적으로 처리되는지 반드시 검증해야 한다. 우리 프로젝트에서 사용하고 있는 MySQL(InnoDB 스토리지 엔진 기준)에서는 이러한 연산이 원자적으로 처리되기 때문에, 동일한 레코드에 대해 여러 트랜잭션이 동시에 실행되더라도 데이터 유실 없이 순차적으로 반영된다.</p>
<hr contenteditable="false" />
<p><span>돌아가서, 해당 로직을 구현했던 당시에도 증분 쿼리라는 방식에 대해 충분히 알고 있었다면, 처음부터 낙관적 락이 아닌 증분 쿼리를 선택했을 것 같다.</span></p>
<p>낙관적 락은 DB 단의 락을 사용하지 않는 방식으로 동시성 문제를 해결할 수 있었고, 당시에는 충돌 가능성이 낮다고 판단했기 때문에 나름의 근거 있는 선택이었다. 하지만 실제로는 버전 충돌이 발생하는 경우가 있었고, 이를 처리하기 위한 재시도 로직이나 예외 핸들링 코드가 추가되면서 코드 복잡도도 함께 증가하였다. 무엇보다도, 낙관적 락은 충돌 시 OptimisticLockingFailureException과 같은 예외가 발생하고, 이에 따른 재시도 비용이 무시할 수 없는 수준으로 존재한다. 특히 좋아요처럼 짧은 시간 내에 집중적인 쓰기 경쟁이 발생할 수 있는 로직에서는 이러한 충돌 가능성을 완전히 배제하기 어렵다.</p>
<p>&nbsp;</p>
<p>반면, 증분 쿼리는 애플리케이션 코드에서 엔티티를 조회하거나 수정을 명시적으로 처리할 필요 없이, 단일 쿼리로 값을 원자적으로 증가시킬 수 있다. MySQL(InnoDB)에서는 해당 연산이 원자적으로 처리되기 때문에 데이터 유실의 위험 없이도 안정적인 동시성 처리가 가능하다. 또한 트랜잭션 내부에서 자동으로 레코드 단위 락이 걸리기 때문에 별도의 버전 관리나 재시도 로직이 필요하지 않다는 점에서 단순하면서도 안전한 방법이다.</p>
<p>&nbsp;</p>
<p>결과적으로, 지금 다시 같은 문제를 마주한다면, 낙관적 락보다 성능적 부담이 적고, 예외 처리가 단순하며, 구현이 간결한 증분 쿼리 방식이 더 나은 선택이라고 판단했다. 때문에, 현재의 낙관적 락을 이용하여 동시성 문제를 해결했던 현재의 로직을 증분 쿼리를 사용하여 해결하기로 변경을 결심했다.</p>