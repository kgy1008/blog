<h2>들어가며..</h2>
<p>Refresh Token 재발급 로직을 구현해 PR을 올렸는데, CodeRabbit으로부터 다음과 같은 피드백을 받았다.</p>
<p><figure class="imageblock widthContent"><span><img height="978" src="https://blog.kakaocdn.net/dn/HXSdv/btsOFt7aN75/9xArYcxU52fm7YFKU38aGK/img.png" width="1500" /></span></figure>
</p>
<p>처음에는 단순히 문자열 비교를 통해 저장된 토큰과 일치하는지를 판별했지만, 해당 피드백을 보고 나서 타이밍 공격이 무엇인지, 그리고 <code>MessageDigest.isEqual</code>이 어떻게 이를 방지하는지에 대해 더 깊이 알아보고자 한다.</p>
<h2>타이밍 공격(Timing Attack)이란?</h2>
<p><b>타이밍 공격(Timing Attack)</b>은 암호 알고리즘이 실행되는 데 걸리는 시간을 분석하여 암호 시스템을 공격하는 부채널 공격(side-channel attack)의 일종이다. 컴퓨터의 모든 논리 연산은 수행하는 데 시간이 소요되며, 이 시간은 입력값에 따라 달라질 수 있다. 따라서 각 연산의 소요 시간을 정밀하게 측정하면, 공격자가 입력값을 역추적할 수 있다.<br />이처럼 시스템이 특정 요청에 응답하는 데 걸리는 시간을 분석하면, 민감한 정보가 유출될 수 있다.</p>
<blockquote><i><b>부채널 공격?</b></i><br />암호학에서 부채널 공격은 알고리즘의 약점을 찾거나 무차별 공격을 하는 대신에 암호 체계의 물리적인 구현 과정의 정보를 기반으로 하는 공격 방법이다.</blockquote>
<h3>String.equals()</h3>
<p><figure class="imageblock widthContent"><span><img height="230" src="https://blog.kakaocdn.net/dn/1c25j/btsOFWVfSDx/XagQe97JfJ5V9UHmwrUad0/img.png" width="1276" /></span><figcaption>초기 재발급 코드</figcaption>
</figure>
</p>
<p>초기 코드를 살펴보면, 로직은 매우 단순하다. DB에 저장된 refreshToken과 사용자가 전달한 refreshToken 값이 일치하는지를 비교하는 방식이다. 이때 문자열 비교를 위해 String.equals() 메서드를 사용하는데, 이 방식은 타이밍 공격에 취약할 수 있다.</p>
<h4>Why?</h4>
<p>Java에서 String 객체 간의 동등성 비교를 수행하는 equals 메서드는 대략적으로 아래와 같이 구현되어 있다.</p>
<pre class="processing"><code>public boolean equals(Object anObject) {
    if (this == anObject) {
        return true; // 참조가 같은 경우
    }
    if (anObject instanceof String) {
        String anotherString = (String) anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            for (int i = 0; i &lt; n; i++) {
                if (v1[i] != v2[i]) {
                    return false;
                }
            }
            return true;
        }
    }
    return false;
}</code></pre>
<p>String.equals()는 내부적으로 문자열의 길이만큼 반복문을 돌며 각 문자를 순차적으로 비교하고 있다. 이때 문자열 길이가 다르면 비교 자체를 바로 장료하거나 반복문을 돌면서 값이 일치하지 않을 경우에는 바로 boolean 값을 early return(조기 종료)하고 있다 . 즉, String.equals()와 같은 일반 비교는 어느 위치까지 문자열이 일치하는지에 따라 수행시간이 달라진다. 이로 인해 equals()를 이용해 토큰이나 비밀번호를 비교할 경우, 실행 시간의 차이를 통해 값의 일부를 유추할 수 있는 <b>타이밍 공격에 취약</b>할 수 있다.</p>
<p>때문에 이런 타이밍 공격을 방어하기 위해 실제로, Spring Security의 BCryptPasswordEncoder와 같은 PasswordEncoder 인터페이스 구현체들의 matches() 메서드는 String.equals()와 같은 단순 비교를 사용하지 않고, 내부적으로 상수 시간 비교(constant-time comparison) 방식을 사용해 비밀번호를 비교한다.</p>
<h2>상수 시간 비교(constant-time comparison) 방식</h2>
<p>문자열을 비교할 때 <b>타이밍 공격을 방지</b>하기 위한 <b>상수 시간 비교(constant-time comparison)</b> 방식은, 비교 시간에 <b>입력값의 내용이나 길이에 따라 차이가 발생하지 않도록</b> 만드는 방식이다. 즉, 상수 시간 비교는 문자열의 길이가 같을 때, 비교하는 문자의 길이와 무관하게 항상 같은 비교 횟수를 수행해서 비교 시간이 문자 내용에 따라 달라지지 않도록 설계하는 방법이다.</p>
<h3>Java로 대표적인 상수 시간 비교를 구현하는 방법</h3>
<h4>1. 직접 구현</h4>
<p>비트 연산자 XOR을 사용하여 직접 구현하는 방식이 있다. 가장 널리 쓰이며 기본적인 방식이다. 앞서 언급한 Spring Security는 BCrypt.checkpw() 메서드 역시 내부적으로는 XOR 연산으로 비교하는 방식으로 구현되어 있다.</p>
<pre class="angelscript"><code>public static boolean constantTimeEquals(String a, String b) {
    if (a == null || b == null) return false;
    if (a.length() != b.length()) return false;

    int result = 0;
    for (int i = 0; i &lt; a.length(); i++) {
        result |= a.charAt(i) ^ b.charAt(i);
    }
    return result == 0;
}</code></pre>
<p>길이가 다르면 곧바로 false를 반환하고, 길이가 같다면 모든 문자를 끝가지 비교하면서 XOR 값을 누적한다. 즉 위치에서 다르든 모든 문자들에 대해서 비교를 수행하기 때문에 시간이 항상 일정하다.</p>
<h4>2. MessageDigest.isEqual()</h4>
<p><b>MessageDigest</b> 클래스는 Java에서 <b>임의의 데이터를 입력받아 고정된 길이의 해시값을 생성</b>하는 역할을 하는 클래스이다. 데이터 무결성 검증, 비밀번호 저장, 디지털 서명 등 보안 관련 작업이나 단방향 해시 함수 값을 구할 때 사용된다.</p>
<p>MessageDigest의 isEqual() 메서드는 생성된 해시값들을 비교할 때 타이밍 공격 방지를 위해 상수 시간 비교를 수행하는 유틸 메서드이다. 구체적인 구현을 살펴보면 아래와 같다.</p>
<p><figure class="imageblock widthContent"><span><img height="1038" src="https://blog.kakaocdn.net/dn/EfXjr/btsOFmmBpUs/gkOwHyEtP5pwlXd4j9KKA0/img.png" width="1022" /></span></figure>
</p>
<p>Java 표준 라이브러리의 MessageDigest의 isEqual() 메서드의 내부 구현도 위에서 언급한 직접 구현 방식과 매우 유사하다. XOR 결과를 하나의 result 변수에 누적하고 모든 연산을 끝까지 수행한 후에 마지막에 최종적으로 결과를 반환하고 있다. 즉, 배열 길이가 달라도 무조건 lenA의 길이만큼 조기 종료없이 반복하게 된다. digestb 길이보다 i가 커지더라도, <code>indexB = ((i - lenB) &gt;&gt;&gt; 31) \* i</code> 로직을 통해서 0번 인덱스를 반복 비교하게 함으로써 길이 차이로 인한 정보 노출을 방지한다.</p>
<p>따라서 비교 수행 시간은 비교 대상 내용이 어디까지 일치하는지에 관계없이 항상 lenA에 비례해 일정하게 유지된다.</p>
<h2>개선된 코드</h2>
<p>RefreshToken은 재발급 시 인증의 핵심 키 역할을 하는 토큰으로 보안상 매우 중요한 정보이다. 따라서 타이밍 공격에 취약한 기존의 String.equals()를 이용한 단순 문자열 비교 대신, Java 표준 API인 MessageDigest.isEqual()을 활용하여 RefreshToken의 일치 여부를 검증하도록 개선할 필요성이 존재한다고 판단했다.</p>
<p><figure class="imageblock widthContent"><span><img height="570" src="https://blog.kakaocdn.net/dn/disG8P/btsOFXfz4ly/CTJCcR2NekE0qSxwPusNR0/img.png" width="1378" /></span><figcaption>수정된 코드</figcaption>
</figure>
</p>
<p>위와 같이 Java 표준 API인 MessageDigest.isEqual()를 활용하여 RefreshToken의 일치 여부를 판단하도록 수정하였다. 상수 시간 비교 방식을 채택함으로써 보안성이 더욱 강화된 안전한 코드가 완성되었다!  </p>