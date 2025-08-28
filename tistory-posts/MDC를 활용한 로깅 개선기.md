<h2>문제상황</h2>
<p>아래 화면은 현재 우리 서버에서 기록되고 있는 로그 일부이다. 회원이 가입을 하면 비동기로 인증 이메일을 발송하는 로직이 동작하는데, 현행 로그 추적 체계에서는 이 요청이 동일한 흐름(즉, 하나의 요청에서 파생된 작업)인지 식별하기가 어렵다는 문제가 있었다.<br />이 때문에 장애가 발생했을 때도 어떤 요청에서 비롯된 문제인지 추적하기 쉽지 않았다.</p>
<p><figure class="imageblock widthContent"><span><img height="182" src="https://blog.kakaocdn.net/dn/btIFF4/btsPNDtnuCN/6wwlicNuk8APicOCSK8WIk/img.png" width="1062" /></span></figure>
</p>
<p>또한 비동기 작업 흐름과 별개로, 어떤 사용자가 보낸 요청인지 식별할 수 없는 점도 불편함을 키웠다. 실제로 사용자 문의 메일을 통해 장애 보고가 접수되더라도, 해당 상황과 관련된 로그를 바로 찾아내기가 어려워 즉각적인 대응이 지연되곤 했다. 물론 타임스탬프를 통해서 어느정도 구분이 가능하지만, 톰캣의 경우 스레드 풀을 재사용하기 때문에 스레드 이름만으로 로그를 추적하면 서로 다른 요청의 로그가 뒤섞이게 되어 요청별 구분에는 한계가 존재한다. 따라서 이번 글에서는 이러한 로그 추적상의 한계를 어떻게 개선했는지, 그 과정을 정리해보고자 한다.</p>
<h2>MDC(Mapped Diagnostic Context)</h2>
<p>MDC는 애플리케이션에서 클라이언트나 요청별 특화 데이터를 로깅에 포함시키기 위해 제공되는 <b>Thread-local 기반의 key-value 저장소</b> 매커니즘이다. SLF4J, Logback 등 주요 로깅 프레임워크가 MDC를 지원하며, 각 스레드는 독립적인 MDC 컨텍스트를 가지므로 동시성 환경에서도 스레드 간 데이터 충돌 없이 메타 정보를 관리할 수 있다. 이를 통해 특정 요청이나 사용자 세션과 연관된 데이터를 로그에 쉽게 삽입하고 추적할 수 있다. 특히, 각 로그의 출처를 클라이언트 단위로 구분해야 할 때 MDC가 매우 유용하다.</p>
<p>&nbsp;</p>
<p><figure class="imageblock widthContent"><span><img height="470" src="https://blog.kakaocdn.net/dn/OMAQk/btsP77JjmKY/xqj7fxqaoBjif9jTwtTKIK/img.png" width="806" /></span></figure>
</p>
<p>Spring MVC는 <b>Thread-per-request 모델</b>로 동작한다. 즉, 하나의 요청은 하나의 스레드가 처리하며, Servlet을 통해 들어온 요청은 Controller Layer를 거쳐 Repository Layer를 통해 데이터베이스에 접근할 때까지 동일한 스레드를 사용한다. 따라서 MDC는 스레드 단위로 키-값을 저장하지만, Spring MVC 환경에서는 마치 요청 단위로 값을 저장하는 것처럼 동작한다.</p>
<h3>적용</h3>
<p>TraceId, UserId 등 로그에 삽입할 값을 MDC에 등록할 때, 어느 단계에서 처리할지 결정이 필요하다. 후보로는 <b>Filter, Interceptor, AOP</b> 방식이 있지만, 요청의 가장 앞단인 <b>Filter</b>에서 처리하는 것이 모든 HTTP 요청에 일관되게 적용될 수 있다.</p>
<ul>
<li><b>Interceptor</b>
<ul>
<li>Spring MVC의 HandlerInterceptor는 Controller를 호출하기 전후로 동작한다.</li>
<li>하지만 정적 리소스, 특정 서블릿 경로, 또는 Controller가 아닌 요청에서는 Interceptor가 호출되지 않을 수 있어 모든 요청에 MDC를 일관되게 적용할 수 없다.</li>
</ul>
</li>
<li><b>AOP</b>
<ul>
<li>AOP는 주로 특정 메서드나 패키지를 대상으로 적용된다.</li>
<li>따라서 모든 요청을 포괄하지 못하고, Controller 외의 요청이나 외부 라이브러리 호출에는 적용되지 않을 수 있다.</li>
</ul>
</li>
<li><b>Filter</b>
<ul>
<li>Servlet Filter는 모든 HTTP 요청을 가장 먼저 처리하므로, MDC 초기화와 값 등록을 모든 요청에 일관되게 적용할 수 있다.</li>
<li>또한 요청이 끝난 후 Filter에서 clear()를 호출해 스레드 풀 환경에서 발생할 수 있는 사이드 이펙트도 방지할 수 있다.</li>
</ul>
</li>
</ul>
<pre class="java" id="code_1756355892738"><code>@Slf4j
@Component
@Order(Ordered.HIGHEST_PRECEDENCE)
@RequiredArgsConstructor
public class RequestLoggingFilter implements Filter {

    private final JwtTokenService tokenService;

    @Override
    public void doFilter(final ServletRequest request, final ServletResponse response, final FilterChain chain)
            throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) request;

        try {
            MDC.put("requestUrl", httpRequest.getRequestURI());
            MDC.put("method", httpRequest.getMethod());

            final String traceId = UUID.randomUUID().toString();
            MDC.put("traceId", traceId);

            String userId = extractUserIdFromRequest(httpRequest);
            MDC.put("userId", userId);

            // Request
            MDC.put("requestType", "request");
            log.info("Received request: {} {}", httpRequest.getMethod(), httpRequest.getRequestURI());

            long startTime = System.currentTimeMillis();

            chain.doFilter(request, response);

            // Response
            long duration = System.currentTimeMillis() - startTime;
            MDC.put("requestType", "response");
            MDC.put("duration", String.valueOf(duration));

            log.info("Response: {} {} - Duration: {}ms",
                    httpRequest.getMethod(), httpRequest.getRequestURI(), duration);

        } finally {
            MDC.clear();
        }
    }

    private String extractUserIdFromRequest(final HttpServletRequest request) {
        String token = request.getHeader("Authorization");
        if (token != null &amp;&amp; !token.isBlank()) {
            return tokenService.parseMemberUuidFromAccessToken(token);
        }
        return "anonymous";
    }
}</code></pre>
<p>단, 주의할 점이 있다. 모든 작업이 끝난 후에는 반드시 clear()를 호출해야 한다. Tomcat과 같은 스레드 풀 기반 서버에서는 작업이 끝난 스레드가 종료되지 않고 스레드 풀로 반환되기 때문에, MDC를 초기화하지 않으면 이전 요청의 값이 남아 <b>사이드 이펙트</b>를 일으킬 수 있다. 따라서 Filter 단에서 요청 처리 후 MDC를 명시적으로 정리하는 것이 필수적이다.</p>
<h3>Spring 비동기 환경에서 MDC(Mapped Diagnostic Context) 활용</h3>
<h4>비동기 환경에서 MDC의 한계</h4>
<p>Spring MVC와 같은 일반적인 Thread-per-request 환경에서는 MDC가 자연스럽게 작동한다. 하나의 HTTP 요청을 하나의 스레드가 처리하기 때문에, Filter나 Interceptor에서 MDC에 TraceId, UserId 등을 등록하면 해당 요청의 모든 로그에 자동으로 포함된다.</p>
<p>하지만 <b>비동기 작업</b>(@Async 메서드 호출, CompletableFuture, TaskExecutor 기반 비동기 처리 등)에서는 상황이 달라진다.</p>
<p>비동기 작업은 <b>별도의 스레드</b>에서 수행된다. MDC는 <b>Thread-local 기반</b>으로 동작하므로, 새로운 스레드에서는 부모 스레드에서 설정한 MDC 값이 자동으로 전달되지 않는다. 결과적으로 비동기 스레드에서 발생하는 로그에는 TraceId, UserId 같은 요청 컨텍스트 정보가 누락되는 문제가 발생한다.&nbsp;즉, 동기 요청에서는 잘 작동하던 MDC가 비동기 컨텍스트에서는 깨진다는 문제가 발생한다.</p>
<h4>TaskDecorator 활용</h4>
<p>Spring 4.3 이후, TaskExecutor를 통해 실행되는 비동기 작업에서는 <b>부모 스레드의 MDC 컨텍스트를 자식 스레드로 전달</b>할 수 있도록 TaskDecorator가 제공된다. TaskDecorator는 Runnable 또는 Callable을 감싸서, 실행 전 필요한 초기화 작업을 수행하고 실행 후 정리 작업을 처리할 수 있는 <b>hook 역할</b>을 한다.</p>
<pre class="java" id="code_1756356624917"><code>private static class MdcTaskDecorator implements TaskDecorator {
        @Override
        public Runnable decorate(final Runnable runnable) {
            Map&lt;String, String&gt; contextMap = MDC.getCopyOfContextMap();
            return () -&gt; {
                if (contextMap != null) {
                    MDC.setContextMap(contextMap);
                }
                try {
                    runnable.run();
                } finally {
                    MDC.clear();
                }
            };
        }
    }</code></pre>
<p>이를 활용하면, 비동기 스레드 호출 시마다 MDC를 파라미터로 전달할 필요 없이, <b>부모 스레드의 MDC 값을 자동으로 캡처</b>하고 새로운 스레드에서 설정한 뒤, 작업이 끝나면 clear()를 호출하여 MDC를 정리할 수 있다. 결과적으로 비동기 작업에서도 요청 단위의 MDC 정보를 일관되게 유지할 수 있다.</p>
<p><figure class="imageblock widthContent"><span><img height="170" src="https://blog.kakaocdn.net/dn/clNs3E/btsP8xTVbbz/HkcABfo7VSNXpWh1GZGK31/img.png" width="2294" /></span></figure>
</p>
<p>짠! 지금처럼 로그가 개선되었다!</p>
<h3>서버 간 호출 환경에서는 어떻게 TraceId를 전파할까?</h3>
<h4>Multi-Agent 아키텍처와 로그 추적 문제</h4>
<p>이후의 글에서 다루겠지만, 2차 MVP 단계에서는 사용자 피드백에 따라 단일 프롬프팅 기반 생기부 생성에서 벗어나, Multi-Agent 기반 응답 생성 아키텍처로 구조를 변경하였다.</p>
<p>기존에는 Spring 서버에서 생기부 초안을 생성하면, 이를 바로 사용자에게 반환하는 구조였다. 그러나 Multi-Agent 구조에서는 1차 초안은 Spring 서버에서 생성되고, 이후 SQS를 통해 2차 Agent인 AWS Lambda로 전달되어 추가 처리 및 최종 응답 생성이 이루어진다.</p>
<p>이 과정에서 SQS를 통해 메시지가 전달되는 동안 TraceId 기반의 로그 추적 흐름이 끊기는 문제가 발생하였다. 즉, Spring 서버에서 생성된 로그와 Lambda에서 생성된 로그 사이의 요청 단위 연결성이 사라져, 요청 추적이 불가능해지는 상황이었다.</p>
<h4>TraceId 전달 방식 개선</h4>
<p>이를 해결하기 위해서는 SQS 메시지 전송 시 TraceId를 함께 전달하고, Lambda에서 이를 읽어 MDC에 설정하도록 구현을 변경해야 했다. 초기에는 메시지 바디에 TraceId와 UserId를 포함하는 방식을 고려했지만, 메시지 바디에 업무 로직과 직접 관련 없는 메타데이터가 섞이게 되어 책임이 명확하지 않다는 점과 바디 구조가 복잡해지고, 로직 처리 시 불필요한 데이터 파싱 부담 발생한다는 문제점이 존재했다.&nbsp;</p>
<p>HTTP 요청의 헤더와 유사한 기능이 없는지 조사한 결과, SQS 메시지 헤더(MessageAttributes)라는 것이 존재한다는 것을 알게 되었고 이 헤더에 UserId와 TraceId를 담아 요청을 보내도록 수정하였다.</p>
<pre class="java" id="code_1756357618713"><code>Map&lt;String, MessageAttributeValue&gt; messageAttributes = new HashMap&lt;&gt;();
messageAttributes.put("traceId", MessageAttributeValue.builder()
        .dataType("String")
        .stringValue(MDC.get("traceId"))
        .build());
messageAttributes.put("userId", MessageAttributeValue.builder()
        .dataType("String")
        .stringValue(MDC.get("userId"))
        .build());

SendMessageRequest sendMsgRequest = SendMessageRequest.builder()
        .queueUrl(sqsUrl)
        .messageBody(businessPayloadJson)
        .messageAttributes(messageAttributes)
        .build();

sqsClient.sendMessage(sendMsgRequest);</code></pre>
<p>위 코드와 같이, MDC에서 TraceId, UserId를 추출하여 SQS MessageAttributes에 TraceId, UserId를 설정하고 메시지 바디에는 실제 비즈니스 데이터만 담아 전송할 수 있다. Lambda에서는 SQS에서 message를 Polling하여, MessageAttributes에서 TraceId, UserId를 추출하고 추출한 값을 Python 로깅 라이브러리(logging)를 통해 로그에 찍은 후, 다음 Lambda로 메시지를 전달할 때, MessageAttributes에 동일한 TraceId, UserId를 그대로 재사용하여 서버간 로그 연속성을 확보할 수 있었다.</p>
<p>&nbsp;</p>