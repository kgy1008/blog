<h2>로깅을 하는 이유?</h2>
<p style="background-color: #ffffff; color: #303a3e; text-align: start;">로깅이란 시스템이 동작하는 동안, 그 상태와 동작 정보를 시간의 흐름에 따라 기록하는 행위를 의미한다. 이 과정에서 생성된 로그는 개발자가 애플리케이션의 동작을 파악하고 문제를 진단하는 데 있어 매우 중요한 역할을 한다. 특히 예기치 못한 오류나 예외 상황이 발생했을 때, 로깅은 문제의 원인을 추적하는 실마리를 제공한다.</p>
<p>개발 과정에서 뿐만 아니라 운영 환경에서도 로깅은 유용하게 활용된다. 예를 들어, 사용자 행동에 대한 로그는 단순한 기록을 넘어, 서비스 개선을 위한 분석 데이터로 활용될 수 있다. 사용자의 패턴을 파악하고 이를 기반으로 기능을 개선하거나, 성능 병목 지점을 찾아내는 데에도 큰 도움이 된다.</p>
<p>하지만 로깅이 무조건 많다고 해서 좋은 것은 아니다. 로깅의 수준(level)이나 범위가 적절하지 않으면, 오히려 시스템의 성능에 부담을 주거나 방대한 양의 로그 파일이 생성되어 운영과 유지보수에 어려움을 겪을 수 있다. 예를 들어 디버그 수준의 로그를 운영 환경에 그대로 남겨둘 경우, 의미 없는 정보가 과도하게 쌓이거나 디스크 공간을 빠르게 소모할 수 있다. 반대로, 필요한 정보를 제대로 남기지 않으면 문제 발생 시 원인을 파악하기 어려워진다.</p>
<p>결국, 효율적인 로깅을 위해서는 <b>무엇을, 언제, 어느 수준으로 기록할지에 대한 기준을 명확히 세우는 것</b>이 중요하다. 로그 레벨을 적절히 조절하고, 로그 메시지에 의미 있는 정보를 포함시켜야만 로깅이 진정한 가치를 발휘할 수 있다.</p>
<h2>스프링 부트와 로깅</h2>
<h3>SKF4J</h3>
<p>스프링 부트는 SLF4J(Simple Logging Facade For Java)를 표준 로그 추상화로 사용하며, 기본 로그 구현체로 Logback을 포함하고 있다. 즉, logger 추상체로써 다른 로깅 프레임워크가 접근할 수 있도록 도와주는 추상화 계층이다. 따라서 SLF4J를 이용하면 코드를 일정하게 유지하면서 구현체의 전환을 통해 다른 로깅 프레임워크로의 전환을 쉽고 간단하게 할 수 있다.</p>
<pre class="java"><code>import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@RestController
public class MyController {

    private static final Logger log = LoggerFactory.getLogger(MyController.class);

    public void someMethod() {
        log.info("SLF4J가 알아서 처리해 줄 거예요!");
    }
}</code></pre>
<p>해당 코드에서 Logger는 SLF4J가 제공하는 인터페이스 타입이며, SLF4J의 정적 팩토리 메서드를 통해 현재 클래스에 맞는 Logger 인스턴스를 생성한다.</p>
<h4>왜 @Slf4j만 붙였는데 로그가 찍힐까?</h4>
<p>Spring Boot 프로젝트를 하다 보면, 로그를 남기기 위해 가장 많이 사용되는 방법 중 하나가 Lombok의 @Slf4j 어노테이션을 사용하는 것이다. 즉, 별도의 코드 없이 클래스에 @Slf4j 하나만 붙이면, 어디서든 log.info(), log.error() 같은 로그 메서드를 손쉽게 사용할 수 있는 것이다! 그런데, 분명 SLF4J는 단순한 인터페이스일 뿐인데, 구현체도 명시하지 않았고, 로그 객체를 선언한 적도 없는데 도대체 어떻게 로그가 출력되는 것일까?</p>
<p>이는 Lombock이 SLF4J 인터페이스 코드를 생성하고, Spring Boot가 Logback 의존성을 자동으로 주입해주기 때문이다.</p>
<p>즉, Lombok에서 제공해주는 해당 어노테이션은 <code>private static final Logger log = LoggerFactory.getLogger(...)</code>을 자동으로 생성해준다고 보면 된다.</p>
<p>Spring Boot 프로젝트에서는 기본적으로 spring-boot-starter-logging 의존성이 포함되어 있는데, 이 안에는 SLF4J의 기본 구현체로 <b>Logback</b>이 내장되어 있다. 다시 말해, 우리가 별도로 로그 구현체를 지정하지 않아도, Spring Boot가 SLF4J를 Logback에 자동으로 연결해주는 것이다. 결국 우리가 @Slf4j 어노테이션만 붙여도 로그가 잘 출력되는 이유는, Lombok이 SLF4J Logger 코드를 자동으로 만들어주고, Spring Boot가 그 SLF4J를 Logback에 연결해주는 흐름인 것이다. 덕분에 우리는 로깅에 필요한 반복적인 코드를 줄이고, 더 간결하게 로그를 남길 수 있게 됩니다.</p>
<p>정리하자면, @Slf4j는 단지 로그 객체를 생성해주는 Lombok의 도우미 역할을 하고, SLF4J는 로그를 추상화한 인터페이스, 그리고 실제 로그를 출력하는 것은 Spring Boot가 제공하는 Logback이 담당하는 구조이다. 이처럼 각자의 역할이 잘 나뉘어 있어, 우리는 로그 구현체에 대한 고민 없이 간단하게 로깅을 시작할 수 있다.</p>
<h3>Logback</h3>
<p>Logback은 Java 기반 Logging Framework 중 하나로 SLF4J의 구현체이다. Spring Boot 환경이라면 별도의 의존성 추가 없이 기본적으로 내장되어 있다.</p>
<h4>로그 레벨</h4>
<p>Logback은 총 5단계의 로그 레벨을 가진다.</p>
<ol>
<li><span style="color: #000000;"><b>Error</b></span><br /><span style="color: #000000;"><span style="text-align: start;">ERROR 수준은 시스템 실행 중&nbsp;</span>명백한 문제가 발생했음을 알리는 로그다. 예외가 발생했거나, 사용자 요청을 처리할 수 없는 상황 등이 이에 해당한다. 운영 환경에서 반드시 수집하고 모니터링해야 할 중요한 로그 수준이다.</span></li>
<li><span style="color: #000000;"><b>Warn</b></span><br /><span style="color: #000000;">WARN 수준은 시스템이 동작은 했지만, 잠재적으로 문제가 될 수 있는 상황을 기록한다. 예를 들어 설정 값이 예상과 다르거나, 특정 기능이 실패했지만 전체 프로세스에는 큰 영향을 주지 않는 경우 등이 이에 해당한다. 문제를 미리 감지하고 대응할 수 있는 힌트를 제공한다.</span></li>
<li><span style="color: #000000;"><b>Info</b></span><br /><span style="color: #000000;">INFO 수준은 시스템이 정상적으로 동작하고 있음을 나타내는 일반적인 정보다. 예를 들어 사용자의 로그인 성공, 서비스의 시작 및 종료, 주요 설정값 출력 등이 이에 해당한다. 운영 환경에서도 비교적 안전하게 사용할 수 있으며, 로그 분석 시 중요한 기준점으로 활용된다.</span></li>
<li><span style="color: #000000;"><b>Debug</b></span><br /><span style="color: #000000;">DEBUG 수준은 시스템의 내부 동작을 이해하기 위해 필요한 정보를 기록한다. 어떤 값이 계산되었는지, 어떤 조건문을 통과했는지 등의 내용을 담는다. 개발 및 테스트 환경에서는 유용하지만, 운영 환경에서는 과도한 로그 발생을 막기 위해 제한적으로 사용해야 한다.</span></li>
<li><span style="color: #000000;"><b>Trace</b></span><br />TRACE 수준은 디버깅보다도 더 세밀한 정보를 담는 가장 낮은 수준의 로그다. 메서드 진입/종료, 루프 내부 상태, 변수 값 등 애플리케이션의 흐름을 매우 상세히 기록할 때 사용한다. 주로 개발 환경에서만 활성화하며, 운영 환경에서는 성능 저하를 유발할 수 있어 비활성화하는 것이 일반적이다.</li>
</ol>
<h4>Logback 설정 방법</h4>
<p>콘솔 로그의 출력 수준을 변경하는 대표적인 방법은 크게 두 가지가 있다. 하나는 application.yml 파일에서 설정하는 방법이고, 다른 하나는 logback-spring.xml 파일을 사용하는 방법이다.</p>
<p>application.yml 방식은 설정이 간단하고 빠르게 적용할 수 있다는 장점이 있지만, 세부적인 설정이나 복잡한 로깅 전략을 적용하기에는 한계가 있다. 특히 로그 파일 분리, 압축, 롤링 정책 같은 고급 기능은 XML 기반 설정 파일에서만 제대로 지원되기 때문에 운영 환경이나 실제 서비스 수준에서는 logback-spring.xml을 활용하는 편이 훨씬 더 유연하고 강력하다.</p>
<p>또한 logback-spring.xml은 Spring의 환경 프로파일(springProfile) 기능을 지원해, 개발 환경과 운영 환경 등 상황에 맞춰 별도의 로깅 설정을 손쉽게 분리할 수 있다는 큰 장점도 갖고 있다.</p>
<hr />
<p>모든 Logback 설정은 &lt;configuration&gt; 태그 안에서 이루어진다. 이 태그 내부에 로그 출력을 담당하는 Appender, 로그의 레벨과 범위를 지정하는 Logger, 로그 메시지의 형식을 정의하는 패턴 등을 설정한다.</p>
<p>아울러 &lt;property&gt; 태그를 활용하면, 로그 경로나 패턴처럼 반복적으로 사용되는 문자열을 변수처럼 선언하여 관리할 수 있어 유지보수가 편리해진다.</p>
<h3 style="color: #000000; text-align: start;">Appender를 활용한 로그 파일 관리</h3>
<p>로그를 남기려면 로그를 어디에 출력할지 지정하는 것이 가장 기본이다. Logback에서는 로그 출력 대상을 <b>Appender</b>라고 부른다. 대표적인 Appender로는 ConsoleAppender, FileAppender, 그리고 RollingFileAppender가 있다. 각각의 특징과 사용법을 살펴보자.</p>
<h4>ConsoleAppender</h4>
<p>ConsoleAppender는 로그를 콘솔(터미널, 명령창)에 출력하는 역할을 한다. 개발 환경에서 가장 많이 사용되며, 빠르게 로그를 확인할 수 있어 편리하다.</p>
<pre class="java" id="code_1752257938764"><code>&lt;appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender"&gt;
    &lt;encoder&gt;
        &lt;pattern&gt;[%d{HH:mm:ss.SSS}] [%level] [%thread] %logger - %msg%n&lt;/pattern&gt;
    &lt;/encoder&gt;
&lt;/appender&gt;</code></pre>
<h4>FileAppender</h4>
<p>FileAppender는 로그를 <b>하나의 파일</b>에 저장한다. 간단하게 파일로 로그를 남기고 싶을 때 유용하다.</p>
<pre class="java" id="code_1752257977007"><code>&lt;appender name="FILE" class="ch.qos.logback.core.FileAppender"&gt;
    &lt;file&gt;logs/app.log&lt;/file&gt;
    &lt;append&gt;true&lt;/append&gt; &lt;!-- 기존 파일에 로그를 이어서 쓴다 --&gt;
    &lt;encoder&gt;
        &lt;pattern&gt;[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%level] [%thread] %logger - %msg%n&lt;/pattern&gt;
    &lt;/encoder&gt;
&lt;/appender&gt;</code></pre>
<h4>RollingFileAppender</h4>
<p>RollingFileAppender는 FileAppender를 상속받아 로그 파일을 <b>자동으로 분할(rollover)</b> 하는 기능을 추가한 Appender이다. 보통 로그를 log.txt 같은 특정 파일에 계속 기록하는데, 파일 크기가 너무 커지거나 일정 시간이 지나면 새로운 파일로 교체하는 작업이 필요하다. 이 역할을 RollingFileAppender가 담당한다.</p>
<pre class="java" id="code_1752257999504"><code>&lt;appender name="ROLLING" class="ch.qos.logback.core.rolling.RollingFileAppender"&gt;
    &lt;file&gt;logs/app.log&lt;/file&gt;
    
    &lt;rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy"&gt;
        &lt;!-- 날짜별로 파일 분리 --&gt;
        &lt;fileNamePattern&gt;logs/app.%d{yyyy-MM-dd}.log&lt;/fileNamePattern&gt;
        &lt;!-- 30일 지난 로그는 삭제 --&gt;
        &lt;maxHistory&gt;30&lt;/maxHistory&gt;
        &lt;!-- 압축 여부 --&gt;
        &lt;cleanHistoryOnStart&gt;true&lt;/cleanHistoryOnStart&gt;
    &lt;/rollingPolicy&gt;
    
    &lt;encoder&gt;
        &lt;pattern&gt;[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%level] [%thread] %logger - %msg%n&lt;/pattern&gt;
    &lt;/encoder&gt;
&lt;/appender&gt;</code></pre>
<p>RollingFileAppender는 로그를 계속해서 지정된 타깃 파일(예: log.txt)에 기록하다가, 설정된 조건에 도달하면(예: 날짜 변경, 파일 크기 초과 등) 현재 파일을 다른 이름으로 바꾸고 새로운 파일을 만들어 다시 로그를 기록한다.</p>
<p>이때, RollingFileAppender와 함께 동작하는 핵심 컴포넌트가 두 가지 있다:</p>
<ul>
<li><b>RollingPolicy</b><br />롤오버(파일 교체)에 필요한 구체적인 동작을 정의한다. 예를 들어 언제 롤오버를 할지, 파일명 패턴은 어떻게 될지, 오래된 로그 파일은 몇 개까지 보관할지 등을 설정한다.</li>
<li><b>TriggeringPolicy</b><br />롤오버가 발생하는 <b>조건과 시점</b>을 정의한다. 예를 들어 파일 크기가 10MB를 초과하면 롤오버한다든지, 매일 자정에 롤오버한다든지 하는 규칙을 지정한다.</li>
</ul>
<p>이 두 정책은 서로 협력하여 RollingFileAppender가 적절한 시점에 로그 파일을 분할하도록 돕는다. 예를 들어, TimeBasedRollingPolicy는 시간 기준으로 롤오버를 트리거하는 정책을 포함하며, 이에 맞춰 파일이 자동으로 분리된다.</p>
<h2>실제 프로젝트에 Logback 설정 적용하기 (feat. 효율적인 로그 파일 관리)</h2>
<h3>로그 파일 압축</h3>
<p>서비스가 운영되면서 가장 신경 써야 할 부분 중 하나가 바로 로그 관리이다. 특히 로그가 계속 쌓이다 보면, 파일 크기가 매우 커져 서버 디스크 용량 부족 문제로 이어질 수 있다. 따라서 적절한 시점에 로그 파일을 분할하고, 오래된 로그를 정리하는 자동화된 관리가 필수다.</p>
<p>이번에는 Logback 설정을 통해 INFO 레벨 로그를 별도의 파일에 기록하고, 파일 크기와 날짜 기준으로 자동 분할 및 압축 관리하는 방법을 살펴보자.</p>
<pre class="java" id="code_1752258490836"><code>&lt;appender name="INFO" class="ch.qos.logback.core.rolling.RollingFileAppender"&gt;
    &lt;filter class="ch.qos.logback.classic.filter.LevelFilter"&gt;
        &lt;level&gt;INFO&lt;/level&gt;
        &lt;onMatch&gt;ACCEPT&lt;/onMatch&gt;
        &lt;onMismatch&gt;DENY&lt;/onMismatch&gt;
    &lt;/filter&gt;
    &lt;encoder&gt;
        &lt;pattern&gt;${LOG_PATTERN}&lt;/pattern&gt;
        &lt;charset&gt;UTF-8&lt;/charset&gt;
    &lt;/encoder&gt;
    &lt;rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy"&gt;
        &lt;fileNamePattern&gt;./log/info/%d{yyyy-MM-dd}.%i.log.gz&lt;/fileNamePattern&gt;
        &lt;maxFileSize&gt;100MB&lt;/maxFileSize&gt;
        &lt;maxHistory&gt;30&lt;/maxHistory&gt;
        &lt;totalSizeCap&gt;10GB&lt;/totalSizeCap&gt;
    &lt;/rollingPolicy&gt;
&lt;/appender&gt;</code></pre>
<p>&nbsp;</p>
<h4>주요 설정 포인트</h4>
<ul>
<li><b>로그 레벨 필터링</b><br />LevelFilter를 사용해 INFO 레벨 로그만 이 파일에 기록하도록 제한했다. 이렇게 하면 DEBUG나 ERROR 로그는 다른 Appender에서 별도로 관리할 수 있다.</li>
<li><b>로그 파일 분할 조건</b><br />SizeAndTimeBasedRollingPolicy는 로그 파일이 일정 크기(maxFileSize: 100MB)를 넘거나, 날짜가 바뀌면 자동으로 새로운 파일을 생성한다. 즉, 매일 로그 파일을 새로 만들고, 하나의 파일 크기가 100MB가 넘으면 다음 인덱스 파일로 넘어간다.</li>
<li><b>파일명과 압축</b><br />fileNamePattern에 .gz 확장자를 지정해 로그 파일을 자동으로 gzip 압축한다.&nbsp;</li>
<li><b>보관 기간과 총 용량 제한</b><br />maxHistory는 30일간의 로그만 보관하고, 그 이전 로그는 자동 삭제한다. 또한 totalSizeCap을 10GB로 제한해, 전체 로그 용량이 이 한도를 초과하면 오래된 로그부터 삭제해 디스크 공간을 확보한다.</li>
</ul>
<h4>왜 이 설정이 중요할까?</h4>
<p>서비스가 장시간 운영될수록 로그는 빠르게 쌓이게 된다. 로그 파일이 무한정 커지면 서버 디스크 공간이 부족해져 장애가 발생할 위험이 커진다. 따라서 로그를 적절한 크기와 시점에 나누어 분할하고, 오래된 로그는 주기적으로 정리하는 관리 전략이 서비스 안정성을 유지하는 데 필수적이다. 또한, 로그 파일을 gzip으로 압축해 저장하면 디스크 공간을 크게 절약할 수 있는데, 이러한 방법은 특히 대규모 서비스에서 더욱 효과적이다.&nbsp;</p>
<h3>로그 레벨별 Appender 분리하기</h3>
<p>실무에서는 로그를 한 파일에 모두 쌓는 대신, 로그 레벨별로 로그 파일을 분리해 관리하는 경우가 많다. 이렇게 하면 각 로그의 중요도에 따라 빠르게 원하는 정보를 찾아내기 쉽고, 장애 대응 및 원인 분석이 훨씬 수월해진다.</p>
<p>예를 들어, ERROR 레벨 로그는 별도의 파일에 모아두어 에러 발생 시점과 내용을 빠르게 확인할 수 있다. WARN 로그도 따로 분리하면 시스템 이상 징후를 사전에 감지하는 데 도움을 준다. 반면, INFO 로그는 서비스의 정상적인 동작 정보를 담고 있으므로 별도의 파일에 보관하며, DEBUG 로그는 개발 환경에서만 활성화해 상세한 내부 동작을 추적한다.</p>
<p>아래는 로그 레벨별로 RollingFileAppender를 분리해 설정하는 예시다.</p>
<pre class="java" id="code_1752259361564"><code>&lt;!-- WARN 로그 전용 Appender --&gt;
&lt;appender name="WARN" class="ch.qos.logback.core.rolling.RollingFileAppender"&gt;
    &lt;filter class="ch.qos.logback.classic.filter.LevelFilter"&gt;
        &lt;level&gt;WARN&lt;/level&gt;
        &lt;onMatch&gt;ACCEPT&lt;/onMatch&gt;
        &lt;onMismatch&gt;DENY&lt;/onMismatch&gt;
    &lt;/filter&gt;
    &lt;encoder&gt;
        &lt;pattern&gt;${LOG_PATTERN}&lt;/pattern&gt;
        &lt;charset&gt;UTF-8&lt;/charset&gt;
    &lt;/encoder&gt;
    &lt;rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy"&gt;
        &lt;fileNamePattern&gt;./log/warn/%d{yyyy-MM-dd}.%i.log.gz&lt;/fileNamePattern&gt;
        &lt;maxFileSize&gt;50MB&lt;/maxFileSize&gt;
        &lt;maxHistory&gt;30&lt;/maxHistory&gt;
        &lt;totalSizeCap&gt;5GB&lt;/totalSizeCap&gt;
    &lt;/rollingPolicy&gt;
&lt;/appender&gt;

&lt;!-- ERROR 로그 전용 Appender --&gt;
&lt;appender name="ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender"&gt;
    &lt;filter class="ch.qos.logback.classic.filter.LevelFilter"&gt;
        &lt;level&gt;ERROR&lt;/level&gt;
        &lt;onMatch&gt;ACCEPT&lt;/onMatch&gt;
        &lt;onMismatch&gt;DENY&lt;/onMismatch&gt;
    &lt;/filter&gt;
    &lt;encoder&gt;
        &lt;pattern&gt;${LOG_PATTERN}&lt;/pattern&gt;
        &lt;charset&gt;UTF-8&lt;/charset&gt;
    &lt;/encoder&gt;
    &lt;rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy"&gt;
        &lt;fileNamePattern&gt;./log/error/%d{yyyy-MM-dd}.%i.log.gz&lt;/fileNamePattern&gt;
        &lt;maxFileSize&gt;50MB&lt;/maxFileSize&gt;
        &lt;maxHistory&gt;60&lt;/maxHistory&gt;
        &lt;totalSizeCap&gt;5GB&lt;/totalSizeCap&gt;
    &lt;/rollingPolicy&gt;
&lt;/appender&gt;</code></pre>
<p>그리고 이렇게 분리된 Appender들을 &lt;root&gt; 또는 특정 Logger에 연결해 사용한다.</p>
<pre class="java" id="code_1752259377608"><code>&lt;root level="INFO"&gt;
    &lt;appender-ref ref="CONSOLE"/&gt;
    &lt;appender-ref ref="INFO"/&gt;
    &lt;appender-ref ref="WARN"/&gt;
    &lt;appender-ref ref="ERROR"/&gt;
&lt;/root&gt;</code></pre>
<p>이처럼 로그 레벨별로 파일을 분리하면, 문제 발생 시 중요한 로그부터 빠르게 선별해 볼 수 있어 로그 분석이 훨씬 효과적이다. 또한, 각 로그 파일에 대해 별도의 보관 정책이나 압축 설정을 적용할 수 있으므로, 운영 환경에서의 로그 관리 효율성도 크게 향상된다.</p>
<h4>프로파일별 로그 설정 분리 방법</h4>
<pre class="java" id="code_1752259435789"><code>&lt;springProfile name="local"&gt;
        &lt;root level="INFO"&gt;
            &lt;appender-ref ref="CONSOLE"/&gt;
            &lt;appender-ref ref="INFO"/&gt;
            &lt;appender-ref ref="WARN"/&gt;
            &lt;appender-ref ref="ERROR"/&gt;
        &lt;/root&gt;
&lt;/springProfile&gt;</code></pre>
<p>Spring Boot에서는 logback-spring.xml 파일 내에서 springProfile 태그를 활용해 프로파일별로 서로 다른 로그 정책을 적용할 수 있다. 예를 들어 위와 같이 local 프로파일에 대한 로그 설정을 지정하면, local 프로파일에서만 위 로그 설정이 적용되고, 다른 프로파일(dev, prod 등)에서는 별도의 프로파일 전용 설정을 사용할 수 있다. 덕분에 환경별로 최적화된 로그 정책을 손쉽게 관리할 수 있다.</p>