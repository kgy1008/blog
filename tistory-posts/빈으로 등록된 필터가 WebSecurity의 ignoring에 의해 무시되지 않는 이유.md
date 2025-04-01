<p><img alt="" src="https://velog.velcdn.com/images/yeoni_/post/25c3c590-7997-4cfe-ad76-6fcc5d990a3c/image.png" /></p>
<p>커스텀 필터를 만들었을 때, <code>@Componenet</code> 어노테이션을 통해 해당 필터를 bean으로 등록했는지의 여부에 따라 filter가 web ignoring에 의해 무시되지 않는 상황이 발생했다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/yeoni_/post/f88234ab-bc53-44bf-987b-ad796dcaddf8/image.png" /><img alt="" src="https://velog.velcdn.com/images/yeoni_/post/01d3b45e-801b-4f17-8a73-a603c415b815/image.png" /></p>
<p>첫번째는 <code>@component</code>로 빈을 등록했을 때이고, 두번째는 빈으로 등록하지 않았을 때이다.</p>
<p>왜 이런 상황이 발생했는지 차근차근 알아보자.</p>
<h2>Spring Security</h2>
<p>  <b>Spring Security</b><br />Spring 기반의 애플리케이션의 보안(인증과 권한, 인가 등)을 담당하는 스프링 하위 프레임워크</p>
<h2>Spring Security 구조</h2>
<p>먼저 Spring Security의 구조를 먼저 이해할 필요가 있다.</p>
<h3><b>Security 의존성이 없는 경우</b></h3>
<p><img alt="" src="https://velog.velcdn.com/images/yeoni_/post/16a69040-9bb9-48cd-bdce-4ad5a030e06d/image.png" /></p>
<p>클라이언트 요청은 서버 컴퓨터의 WAS(톰캣)의 필터들을 통과한 뒤 스프링 컨테이너의 컨트롤러에 도달한다.</p>
<h3><b>Security 의존성 추가 &rarr; 사용자의 요청을 감시</b></h3>
<p><img alt="" src="https://velog.velcdn.com/images/yeoni_/post/7d4d7093-f4e9-49ae-989f-0fb201aa4afb/image.png" /><img alt="" src="https://velog.velcdn.com/images/yeoni_/post/f32ab542-5fc0-4c15-927a-ca3c8ccdd476/image.png" /></p>
<p>WAS의 필터단에서 요청을 가로챈 후 시큐리티의 역할을 수행한다.</p>
<ul>
<li>WAS의 필터에 하나의 필터를 만들어서 넣고 해당 필터(Delegating Filter Proxy)에서 요청을 가로챔</li>
<li>해당 요청은 스프링 컨테이너 내부에 구현되어 있는 스프링 시큐리티 감시 로직을 거침</li>
<li>시큐리티 로직을 마친 후 다시 WAS의 다음 필터로 복귀</li>
</ul>
<h2>동작 과정</h2>
<p><img alt="" src="https://velog.velcdn.com/images/yeoni_/post/62ffd156-5ea5-4834-9151-de8f73f53a79/image.png" /></p>
<p>즉 정리하면 전체적인 동작 과정은 위와 같다. 왼쪽의 FilterChain은 WAS의 필터체인이고 오른쪽의 SecurityFilterChain은 Spring Security에서 사용하는 필터체인이다. 즉, HTTP요청을 가져와야하기 때문에 WAS의 필터단에 위치하는 특별한 필터인 Delegating Filter Proxy에서 요청을 가로채서 SecurityFilterChain으로 연결시켜준다.</p>
<p>여기서 주목해야할 점은 우리가 <code>@Component</code> 어노테이션을 사용해서 빈으로 등록된 filter는 SecurityFilterChain에 위치하는 것이 아니라 WAS의 필터단에 위치한다는 것이다.</p>
<h3>Custom filter</h3>
<pre class="scala"><code>@RequiredArgsConstructor
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter { ... }</code></pre>
<p>즉, 위와 같이 <code>@Component</code> 어노테이션과 Filter 인터페이스를 이용하면 필터체인에 추가되는데, 이 경우 WAS의 ApplicationFilterChain에 추가되는 것이다.</p>
<h3>Security Config</h3>
<pre class="aspectj"><code>@Bean
public WebSecurityCustomizer webSecurityCustomizer() {
    return web -&gt; web.ignoring().requestMatchers(whiteList);
}</code></pre>
<p>WebSecurity.ignoring() 은 필터가 없는 SecurityFilterChain 을 만들게 되고,whiteList에 포함된 경로에 대한 요청을 필터가 존재하지 않는 SecurityFilterChain으로 전달한다</p>
<p><img alt="" src="https://velog.velcdn.com/images/yeoni_/post/46b5fc53-4a06-4efe-829f-2dc1852a22e1/image.png" /></p>
<p><code>@Component</code>로 등록한 MyCustomFilter(JwtAuthenticationFilter)는 WAS의 ApplicationFilterChain에 등록되어 있기 때문에 ignoring을 해주어도 항상 해당 커스텀 필터를 지나게 된다.</p>
<hr />
<h3>참고 자료</h3>
<p><a href="https://bitgadak.tistory.com/10">https://bitgadak.tistory.com/10</a></p>