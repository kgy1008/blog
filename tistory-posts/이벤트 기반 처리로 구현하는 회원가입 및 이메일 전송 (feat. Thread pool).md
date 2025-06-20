<h2>요구사항</h2>
<p><figure class="imageblock widthContent"><span><img height="401" src="https://blog.kakaocdn.net/dn/bz9No6/btsOMSYVv6g/TcKtkWkfqLkMM3saHgZ8i0/img.png" width="1280" /></span></figure>
</p>
<p><a href="https://wing1008.tistory.com/52">이전 포스팅</a>에서도 언급했듯, EduMate 서비스에는 이메일 인증 프로세스가 존재한다. 사용자가 회원가입 버튼을 클릭하면 인증을 위한 인증 코드가 생성되고, 해당 코드가 포함된 인증 링크가 담긴 이메일이 발송되는 흐름이다. 해당 로직을 구현하면서 고려했던 사항들은 아래와 같다.</p>
<p>&nbsp;</p>
<p><b>1. 회원가입이 성공적으로 완료된 후, 이메일이 발송되어야 한다.</b></p>
<p>즉, 회원가입을 수행하던 도중 어떠한 이유로 오류가 발생하여 롤백되는 경우, 인증 이메일이 발송되는 일은 없어야 한다. 가입 정보는 DB에 존재하지 않는데, 이메일은 발송되었다면 혼란을 줄 수 있기 때문이다. 따라서, 이메일 전송은 회원가입이 성공한 이후에만 실행되어야 한다.</p>
<p>&nbsp;</p>
<p><b>2. 외부 메일 서비스의 장애가 회원가입 로직의 실패로 이어져서는 안된다.</b></p>
<p>현재 우리 서비스는 메일 전송을 위해 AWS SES를 이용하고 있다. 하지만 외부 메일 서비스는 언제든 장애가 발생할 수 있다. 만약 AWS SES가 일시적으로 응답하지 않거나 타임아웃이 발생하는 경우에도 회원가입 자체가 실패하는 일은 없어야 한다. 즉, 부가기능인 이메일 전송 실패 여부와는 별개로, 핵심 기능인 회원 가입은 정상적으로 완료되어야 한다.</p>
<p>&nbsp;</p>
<p><b>3. 메일 전송과 같이 네트워크를 통해 원격 서버와 통신하는 작업은 DBMS의 트랜잭션 내에서 제거해야 한다.</b></p>
<p>메일 전송이 트랜잭션 범위에 포함되어 있다면, 커넥션을 소유하는 시간이 길어져 DB 커넥션이 장시간 점유되고 커넥션 풀이 고갈될 위험이 존재한다. 또한, 프로그램이 실행하는 동안 메일 서버와 일시적으로 통신할 수 없는 상황이 발생한다면 웹 서버뿐 아니라 DBMS 서버까지 위험해지는 상황이 발생할 수 있기 때문에 반드시 트랜잭션 범위에서 제거하여야 한다. 따라서 이메일 전송은 회원가입 트랜잭션이 완전히 commit 된 이후에 트랜잭션 범위 밖에서 수행되어야 한다.</p>
<h2>메일 전송을 비동기로 처리하자</h2>
<p>이메일 전송 요청 흐름을 동기적으로 처리한다면, 이메일을 보내는 동안 HTTP 요청-응답 흐름이 완료되지 않고 계속 대기하게 되고 이는 결국 클라이언트가 응답을 받기까지 걸리는 시간이 길어지게 된다. 특히, 이메일 서버가 느리거나 일시적 네트워크 지연이 발생하는 상황에서는, 웹 서버가 불필요하게 커넥션을 길게 점유하게 되고 이는 전체 시스템의 리소스 낭비로 이어질 수 있다.</p>
<p>또한 회원가입 API 통신에서 실제로 '회원가입' 자체는 성공적으로 완료되었음에도 불구하고, 이메일 전송에 실패했다는 이유로 클라이언트에게 에러 메세지를 내려주게 되고 혼란을 유발할 수 있다.</p>
<p>&nbsp;</p>
<p>더하여, 우리 서비스의 기획상, 이메일을 받지 못한 사용자를 위해 인증 메일 재발송 버튼을 제공하고 있으며, 이메일 인증에 실패하더라도 로그인은 허용하고 있다. 즉, 가입 직후 인증 메일 전송이 실패하더라도 사용자는 언제든지 마이페이지나 재발송 버튼을 통해 인증을 완료할 수 있기 때문에 비동기로 처리했을 때 발생할 수 있는 일시적인 데이터 불일치 문제는 결과적으로는 최종적인 일관성을 보장할 수 있다고 판단했다.</p>
<p>&nbsp;</p>
<p>이러한 이유로, 핵심 비즈니스 로직인 '회원가입'과 외부 시스템인 '이메일 전송'을 명확히 분리할 필요가 있다고 판단하였고, 최종적으로 <b>이메일 전송을 비동기로 처리</b>하기로 결심했다.</p>
<h2>비동기 처리만으로는 부족하다!</h2>
<p>메일 전송을 비동기로 처리하기로 결정했음에도 앞서 작성했던 모든 고려 사항들을 충족시킬 수는 없었다.</p>
<p><figure class="imageblock widthContent"><span><img height="686" src="https://blog.kakaocdn.net/dn/Oi96l/btsOKDPVs63/ASWBpGgOWCVtJyXWZxcjS1/img.png" width="1268" /></span></figure>
</p>
<p>단순히 이메일 전송 로직을 비동기로 처리하면, 회원가입 로직이 아직 완료되지 않았음에도 불구하고 이메일이 전송되는 문제가 발생할 수 있다. 이는 앞서 언급했던 첫 번째 고려사항인 '회원가입이 성공적으로 완료된 후에만 이메일이 발송되어야 한다.'라는 원칙을 위배하게 된다. 즉, 트랜잭션이 커밋되기도 전에 비동기 메일 전송 작업이 먼저 실행된다면 가입 정보는 데이터베이스에 존재하지 않는데 사용자는 인증 메일을 받게되는 이상한 상황이 발생할 수 있다.</p>
<p>때문에 단순한 비동기 처리만으로 충분하지 않았고, <b>회원가입이 성공적으로 완료되고 인증코드가 정상적으로 생성된 이후에만 메일을 전송하도록 보장</b>하는 방법을 고민해야 했다.</p>
<h3>이벤트 기반(Event-Driven) 처리</h3>
<p>이벤트 기반으로 처리한다는 것은, <b>회원가입 트랜잭션이 커밋된 이후에 이벤트를 발행하고</b>, 해당 이벤트를 <b>리스너가 구독하여 이메일 전송 작업을 수행하는 방식</b>을 의미한다.&nbsp;</p>
<h4>Spring Event</h4>
<p>Spring은 Observer(옵저버) 패턴을 기반으로 한 이벤트 처리 메커니즘을 프레임워크 수준에서 지원한다. <br />ApplicationEventPublisher는 이 이벤트 시스템의 핵심 컴포넌트로, 빈(Bean) 간의 느슨한 결합을 유지하면서 <b>비동기적 혹은 동기적인 흐름 제어</b>를 가능하게 해준다.</p>
<p>이벤트 발행 측에서는 아래 코드와 같이 ApplicationEventPublisher를 주입받아, 이벤트를 발행할 수 있다.</p>
<pre class="java" id="code_1750427884904"><code>@Component
@RequiredArgsConstructor
public class MemberSignUpService {

    private final ApplicationEventPublisher eventPublisher;

    public void signUp(Member member) {
        // 회원가입 로직 수행
        ...

        // 이벤트 발행
        eventPublisher.publishEvent(new MemberSignUpEvent(member.getEmail()));
    }
}</code></pre>
<p>이렇게 발행된 이벤트는 컨테이너에 등록된 리스너가 처리한다. 이 리스너는 @EventListener 또는 @TransactionalEventListener 어노테이션을 통해 선언할 수 있는데, 특히, <code>@TransactionalEventListener</code>를 활용하면 트랜잭션이 커밋된 이후에만 해당 이벤트를 처리하도록 설정할 수 있다.</p>
<pre class="java" id="code_1750427200266"><code>@Async("emailTaskExecutor")
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
public void handleEmailEvent(final MemberSignedUpEvent event) {
    try {
        emailService.sendEmail(event.email(), event.memberUuid(), event.verificationCode());
    } catch (Exception e) {
        log.error("비동기 이메일 발송 실패. event={}", event, e);
    }
}</code></pre>
<p>위의 코드처럼 <code>phase = TransactionPhase.AFTER_COMMIT</code>옵션을 명시하면, 회원가입 트랜잭션이 성공적으로 완료된 이후에만 이메일 전송이 실행되도록 보장할 수 있다.&nbsp;</p>
<p>이를 통해 3번째 고려사항이었던, '이메일 전송 로직을 트랜잭션 외부로 안전하게 분리해야 한다'는 요구상황을 만족할 수 있으며, 불필요한 커넥션 점유나 외부 서비스 장애로 인해 회원가입이 영향을 받는 상황도 방지할 수 있다.</p>
<p>또한, Spring Event는 기본적으로 동기적으로 처리되자만, 필요한 경우 @Async 어노테이션을 추가하여 비동기로 처리할 수도 있다.</p>
<h4>메시지 브로커를 사용</h4>
<p>AWS SQS, Kafka등 메시지 브로커를 활용해서 해결할 수도 있다. 회원가입이 완료되면, 이메일 전송을 직접 수행하는 대신 메시지를 큐(브로커)에 발행(Publish)한다. 이메일 전송을 담당하는 소비자(Consumer)가 큐를 구독(Subscribe)하고 있다가 해당 메시지를 받으면 메일을 전송하는 구조이다.</p>
<p><figure class="imageblock widthContent"><span><img height="496" src="https://blog.kakaocdn.net/dn/bjEuz2/btsOL4Z77oR/83sy2z0jcoM6FZsOtW6fk0/img.png" width="1716" /></span></figure>
</p>
<p>이런 방식은 회원가입과 이메일 전송을 완전히 분리할 수 있다는 장점이 있다. 또한, 이메일 서비스가 일시적으로 중단되더라도 메시지는 큐에 안전하게 저장되기 때문에 장애가 복구된 이후에도 메일 전송 작업을 이어서 처리할 수 있다.</p>
<h2>선택한 해결 방법</h2>
<p>서비스가 크고 이메일 외에도 다양한 후속 작업들이 발생하는 경우에는 메시지 큐 기반의 아키텍쳐로 확장하는 것이 훨씬 유연하고 견고한 방식이 될 수 있지만&nbsp;<span style="color: #333333; text-align: start;">현재 우리의 서비스는 단일 애플리케이션 환경이며 서비스 규모 또한 크지 않은 상황이다.<span> 또한 가입 인증 메일 전송 작업은 큐에 저장해 처리할 만큼 절대 유실되어서는 안되는 중요한 작업도 아니다. 이러한 이유로 외부 메시지 브로커를 도입하는 것은 오버 스펙이라고 생각하였기 때문에 <span style="color: #333333; text-align: start;">고민 끝에 Spring Event를 사용해서 해결하기로 선택했다.<span>&nbsp;</span></span></span></span></p>
<h3><span style="color: #333333; text-align: start;"><span><span style="color: #333333; text-align: start;"><span>구현을 합시댜</span></span></span></span></h3>
<h4><span style="color: #333333; text-align: start;"><span><span style="color: #333333; text-align: start;"><span>Spring Event</span></span></span></span></h4>
<pre class="java" id="code_1750427238722"><code>@Service
@RequiredArgsConstructor
public class AuthFacade {

    private final ApplicationEventPublisher eventPublisher;

    @Transactional
    public MemberSignUpResponse signUp(final String email, final String password, final String subjectName, final String school) {
        checkPreconditions(email, password);
        Subject subject = subjectService.getSubjectByName(subjectName);
        Member member = createMember(email, password, subject, school);
        String accessToken = tokenService.generateTokens(member, TokenType.ACCESS);
        String refreshToken = tokenService.generateTokens(member, TokenType.REFRESH);
        memberService.updateRefreshToken(member, refreshToken);
        String authCode = issueVerificationCode(member);
        // 이벤트 발행
        eventPublisher.publishEvent(MemberSignedUpEvent.of(member.getEmail(), member.getMemberUuid(), authCode));
        return MemberSignUpResponse.of(accessToken, refreshToken);
    }
}</code></pre>
<p>회원가입 API에서는 회원 정보를 저장하고 인증 코드를 생성한 뒤, 회원가입이 완료되었음을 알리는 이벤트를 발행한다. 이후 API는 곧바로 클라이언트에게 회원 가입이 완료되었다는 성공 응답을 반환한다.</p>
<pre class="java" id="code_1750428574949"><code>@Slf4j
@Component
@RequiredArgsConstructor
public class EmailEventListener {

    private final EmailService emailService;

    @Async("emailTaskExecutor")
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void handleEmailEvent(final MemberSignedUpEvent event) {
        try {
            emailService.sendEmail(event.email(), event.memberUuid(), event.verificationCode());
        } catch (Exception e) {
            log.error("비동기 이메일 발송 실패. event={}", event, e);
        }
    }
}</code></pre>
<p>이벤트를 수신한 리스너는, 트랜잭션이 성공적으로 커밋된 이후에만 실행되며, 비동기 방식으로 인증 메일을 전송한다.</p>
<h4>@Async</h4>
<p>비동기 기능을 활성화하기 위해서는 아래와 같이 Spring Boot 메인 설정 클래스에 @EnableAsync 어노테이션을 붙여 비동기 처리를 활성화시켜줘야 한다.</p>
<pre class="java" id="code_1750432335437"><code>@EnableAsync
@Configuration
public class AsyncConfig {

    @Bean(name = "emailTaskExecutor")
    public Executor emailTaskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(Runtime.getRuntime().availableProcessors());
        executor.setMaxPoolSize(Runtime.getRuntime().availableProcessors() * 2);
        executor.setQueueCapacity(500);
        executor.setThreadNamePrefix("email-async-");
        executor.initialize();
        return executor;
    }
}</code></pre>
<p>@Async를 사용했을 때 직접 스레드풀 빈을 등록하지 않으면 스프링은 내부에 기본으로 등록된 SimpleAsyncTaskExecutor를 사용한다. <span style="color: #333333; text-align: start;">SimpleAsyncTaskExecutor은 thread pool이 아니라 매 작업마다 단순히 새로운 thread를 생성해서 실행하는 방식으로 동작한다. </span></p>
<p><span style="color: #333333; text-align: start;">즉, Spring처럼 Thread per Request로 동작하는 환경에서는 매 요청마다 새로운 스레드가 만들어지므로, 스레드가 재사용되지 않아 스레드 생성 비용이 증가하며 스레드 수가 무제한으로 증가할 수 있다. 이로 인해 메모리 부족 현상이 발생하거나 어느 순간 서버 전체가 응답 불가능 상태에 빠질 수도 있다. </span><span style="color: #333333; text-align: start;">때문에 <b>@Async가 사용할 스레드풀을 반드시 직접 빈으로 생성해 등록</b>하는 것을 권장한다.</span></p>
<p><figure class="imageblock widthContent"><span><img height="323" src="https://blog.kakaocdn.net/dn/bibUIR/btsOKUjQuCn/Q1QRf21gURLMFeGHg2aksK/img.png" width="723" /></span></figure>
</p>
<p><span style="color: #333333; text-align: start;">스레드 풀을 직접 생성할 때, 적절한 스레드 수를 결정하는 일은 쉽지 않다. 특히, 이메일 전송과 같은 I/O Bound 작업의 경우, 최적의 스레드 개수는 경험과 테스트를 통해 찾아야 한다. 스레드 수가 너무 적으면 처리 병목이 발생할 수 있고 반대로 스레드 수가 과도하게 많으면 컨텍스트 스위칭 비용이 커지고 메모리 사용량도 늘어나 성능 저하와 자원 낭비를 초래할 수 있다. <br />또한 큐의 사이즈도 신중히 설정해야 한다. 큐의 사이즈를 제한하지 않는다면 끊임없이 요청이 쌓여 메모리가 고갈될 위험이 있기 때문이다.</span></p>
<p>&nbsp;</p>
<p><span style="color: #333333; text-align: start;">현재는 CPU 코어 수 만큼 스레드를 기본으로 유지하되, 필요 시 최대 코어 수의 2배만큼 확장 가능하도록 설정해두었다. 후에 성능 및 부하 테스트를 진행하며 최적의 설정값을 찾아갈 계획이다.</span></p>
<p>&nbsp;</p>
<hr contenteditable="false" />
<p>이로써 핵심 로직(회원 가입)과 외부 시스템 호출(이메일 전송)을 명확하게 분리하면서도, 이메일 전송 실패가 회원가입 로직에 영향을 주지 않도록 구현할 수 있다.</p>