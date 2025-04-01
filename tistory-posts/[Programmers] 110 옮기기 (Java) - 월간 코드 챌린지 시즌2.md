<h1>문제 링크</h1>
<p><a href="https://school.programmers.co.kr/learn/courses/30/lessons/77886">https://school.programmers.co.kr/learn/courses/30/lessons/77886</a></p>
<figure contenteditable="false" id="og_1743062907680"><a href="https://school.programmers.co.kr/learn/courses/30/lessons/77886" rel="noopener" target="_blank">
<div class="og-image">&nbsp;</div>
<div class="og-text">
<p class="og-title">프로그래머스</p>
<p class="og-desc">SW개발자를 위한 평가, 교육, 채용까지 Total Solution을 제공하는 개발자 성장을 위한 베이스캠프</p>
<p class="og-host">programmers.co.kr</p>
</div>
</a></figure>
<p>&nbsp;</p>
<h2>사고 과정</h2>
<p>해당 문제는 문자열에서 110을 추출하여 다시 삽입하는데 문자열 중 사전 순으로 가장 앞에 오는 문자열이 되도록 삽입해야 한다.</p>
<p><code>사전 순으로 가장 앞에 와야 한다.</code> 가 이 문제의 핵심 요구사항이다. 어떻게 하면 사전 순으로 가장 앞에 오게 만들 수 있을까?</p>
<p>일단 <b>추출 과정</b>부터 살펴보자. 주어진 예시 중 하나인 "100111100"에서 110을 추출해보자. 추출 후 문자열은 "100110"이 된다. 즉, 처음의 문자열에서 110을 추출하였는데 추출 후 다시 문자열이 '110'을 포함하는 상황이 발생했다. 이러한 상황에서 자료구조 '스택(Stack)'을 이용하면 문자열의 순서를 유지한 채 나중에 생기는 110을 간단히 찾아낼 수 있다.</p>
<p>이제 <b>삽입과정</b>을 살펴보자. 110을 추출한 문자열에 다시 '110'을 삽입할 때, 삽입할 위치 뒤에 0이 존재해서는 안된다. 뒤에 0이 존재하는 순간 사전 순으로 가장 앞에 와야 한다는 문제의 요구사항을 위반하기 때문이다. 즉, 110을 삽입할 위치는 가장 마지막으로 0이 나온 위치 다음에 삽입해야 한다. 마지막으로 위치한 0의 위치 역시 stack의 <code>peek()</code> 메서드를 사용하면 쉽게 찾을 수 있다.</p>
<h2>최종 코드</h2>
<pre class="arduino"><code>import java.util.*;

class Solution {
    public String[] solution(String[] s) {
        String[] answer = new String[s.length];

        for (int j = 0; j &lt; s.length; j++) {
            Deque&lt;Character&gt; stack = new ArrayDeque&lt;&gt;();
            int cnt = 0; // 110의 개수
            StringBuilder sb = new StringBuilder();
            String str = s[j];

            for (int i = 0; i &lt; str.length(); i++) {
                char z = str.charAt(i);
                if (stack.size() &gt;= 2) {
                    char x = stack.pop();
                    char y = stack.pop();

                    if (z == '0' &amp;&amp; x == '1' &amp;&amp; y == '1') {
                        cnt++;
                    } else {
                        stack.push(y);
                        stack.push(x);
                        stack.push(z);
                    }
                } else {
                    stack.push(z);
                }
            }

            while (!stack.isEmpty() &amp;&amp; stack.peek() != '0') {
                sb.append(stack.pop());
            }

            for (int t = 0; t &lt; cnt; t++) {
                sb.append(0).append(1).append(1);
            }

            while (!stack.isEmpty()) {
                sb.append(stack.pop());
            }
            String temp = sb.reverse().toString();
            answer[j] = temp;
        }

        return answer;
    }
}
</code></pre>
<p>&nbsp;</p>
<p>스택을 이용해서 풀어야 한다는 것을 빠르게 눈치채지 못했다. '가장 최근(가까운)' 것을 파악해야 하는 문제라면 스택을 떠올리는 자세를 챙겨야겠다.</p>
<p>&nbsp;</p>
<p><img alt="" src="https://velog.velcdn.com/images/yeoni_/post/67fb784d-a25b-4883-ad96-2c036d112fcb/image.png" /></p>