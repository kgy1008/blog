<h1>문제 링크</h1>
<p><a href="https://school.programmers.co.kr/learn/courses/30/lessons/81303">https://school.programmers.co.kr/learn/courses/30/lessons/81303</a></p>
<figure contenteditable="false" id="og_1743062890354"><a href="https://school.programmers.co.kr/learn/courses/30/lessons/81303" rel="noopener" target="_blank">
<div class="og-image">&nbsp;</div>
<div class="og-text">
<p class="og-title">프로그래머스</p>
<p class="og-desc">SW개발자를 위한 평가, 교육, 채용까지 Total Solution을 제공하는 개발자 성장을 위한 베이스캠프</p>
<p class="og-host">programmers.co.kr</p>
</div>
</a></figure>
<p>&nbsp;</p>
<h1>문제 분석</h1>
<p>시간 복잡도를 줄이는 것이 관건인 문제이다. 문제에서 주어진 변수들의 범위는 5 &le; n &le; 1,000,000과 1 &le; cmd의 원소 개수 &le; 200,000이다. 명령어 'U' 또는 'D'가 입력되었을 때, 주어진 숫자 X만큼 이동하는 것이 핵심이다. 이때, 삭제된 행으로 이동하는 것은 이동했다고 간주하지 않기 때문에, 배열로 선언하여 순차 탐색으로 포인터를 움직인다면 주어진 X보다 더 많이 반복문을 돌며 포인터를 이동해야하기 때문에 시간 초과가 발생할 수 있다. 때문에, X만큼 포인터를 이동할 때, 정확히 딱 X만큼 연산을 수행하여 이동시키는 것이 핵심이다.</p>
<h2>TreeSet 이용</h2>
<p>자바에는 TreeSet이라는 컬렉션이 존재한다. Set의 구현체 중 하나로 일반적인 HashSet에서 정렬 기능이 추가된 Set이라 생각하면 된다.(<i>내부적으로 <b>Red-Black Tree(레드-블랙 트리)</b>를 사용하여 항상 정렬된 상태를 유지</i>)<br />해당 자료 구조를 이용하면 시간 복잡도를 줄일 수 있다.</p>
<p>먼저, 삭제되지 않은 행들의 인덱스 값을 저장하는 treeSet을 만들고, 삭제된 행들의 인덱스 값들을 저장하는 stack을 만들자. stack으로 선언한 이유는 명령어 'Z'을 이용하여 복구 시, 가장 마지막으로 삭제된 행을 복구 시켜야 하기 때문에 LIFO인 stack을 이용해 구현하면 간단하게 해결할 수 있기 때문이다.</p>
<p>기본적으로 Set은 순서를 보장하지 않기 때문에 매번 값을 삭제하거나 추가할 때마다 자동으로 인덱스 값 크기 순서대로 오름차순 정렬하기 위해서 treeSet을 이용했다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/yeoni_/post/988e9656-97e7-401a-ac3f-f7217f965831/image.png" /></p>
<p><br />이처럼 삭제된 행들의 인덱스 값들을 아예 리스트에서 삭제시켜 버리면, 불필요하게 삭제된 행들로 이동하는 과정없이 바로 존재하는 다음 행으로 포인터를 이동시킬 수 있다. 즉, <code>X만큼 이동하라</code> 라고 했을 때, 추가적인 연산 없이 정확히 X번만 연산하여 포인터를 이동시킬 수 있는 것이다.</p>
<h3>구현해보자.</h3>
<p>대충 어떻게 구현할 지 정리해보았으니 구현해보자. TreeSet은 여러가지 유용한 API들을 제공해준다. 아래 링크를 참고해보자.<br /><a href="https://codevang.tistory.com/137">TreeSet 주요 메서드</a></p>
<p><code>lower()</code>과 <code>higher()</code> 이 2개의 메서드를 알고 있다면 정말 쉽게 구현할 수 있다. (</p>
<p><del>나도 처음에 몰랐다. 익혀두자!</del></p>
<p>) 이때 NPE에 조심해야 하는데, 만약 포인터가 마지막 행을 가리키고 있었고 이때 이 행을 삭제했을 경우에는 포인터를 위로 옮겨주어야 하기 때문에 현재의 pointer 값보다 큰 값이 존재하지 않을 경우에는 <code>lower()</code> 메서드를 실행하도록 분기처리 했다.</p>
<h3>최종 코드</h3>
<pre class="processing"><code>import java.util.*;

class Solution {
    static int pointer;

    public String solution(int n, int k, String[] cmd) {
        TreeSet&lt;Integer&gt; set = new TreeSet&lt;&gt;();
        Stack&lt;Integer&gt; deleted = new Stack&lt;&gt;();
        pointer = k;

        for (int i = 0; i &lt; n; i++) {
            set.add(i);
        }

        for (String commands : cmd) {
            if (commands.equals("C")) { // 삭제
                deleted.push(pointer); 
                set.remove(pointer);

                if (set.higher(pointer) != null) { 
                    pointer = set.higher(pointer); 
                } else {
                    pointer = set.lower(pointer); 
                }
            } else if (commands.equals("Z")) { // 복구
                set.add(deleted.pop());
            } else {
                String[] command = commands.split(" ");
                int dist = Integer.parseInt(command[1]);

                if (command[0].equals("D")) { // 아래로 이동
                    while (dist-- &gt; 0) {
                        pointer = set.higher(pointer);
                    }
                } else { // 위로 이동
                    while (dist-- &gt; 0) {
                        pointer = set.lower(pointer);
                    }
                }
            }
        }

        StringBuilder sb = new StringBuilder();
        for (int i = 0; i &lt; n; i++) {
            if (set.contains(i)) {
                sb.append("O");
            } else {
                sb.append("X");
            }
        }

        return sb.toString();
    }
}</code></pre>
<p>확실히 Java Collection이 제공해주는 API들을 많이 알면 알수록 유리해지는 것 같다. 많은 연습이 필요할 것 같다.</p>