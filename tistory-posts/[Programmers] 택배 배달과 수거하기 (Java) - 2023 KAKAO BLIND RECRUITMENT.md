<h1>문제 링크</h1>
<p><a href="https://school.programmers.co.kr/learn/courses/30/lessons/150369">https://school.programmers.co.kr/learn/courses/30/lessons/150369</a></p>
<figure contenteditable="false" id="og_1743062953429"><a href="https://school.programmers.co.kr/learn/courses/30/lessons/150369" rel="noopener" target="_blank">
<div class="og-image">&nbsp;</div>
<div class="og-text">
<p class="og-title">프로그래머스</p>
<p class="og-desc">SW개발자를 위한 평가, 교육, 채용까지 Total Solution을 제공하는 개발자 성장을 위한 베이스캠프</p>
<p class="og-host">programmers.co.kr</p>
</div>
</a></figure>
<p>&nbsp;</p>
<h2>접근 방법</h2>
<p>처음 문제를 읽어보면, 복잡하다고 생각할 수 있지만 예시와 상황을 찬찬히 살펴보면 그리디적으로 접근하면 쉽게 해결할 수 있다는 것을 금방 눈치챌 수 있다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/yeoni_/post/589bd9b6-75ca-4da4-8b90-d2f6f7cca53f/image.png" /></p>
<p>최소 거리로 이동하면서 모든 집에 배달 및 수거하기 위해서는, 가장 멀리 떨어진 집에 배달을 하면서 물류창고로 돌아올 때, 멀리 떨어진 집부터 택배 상자를 함께 수거해야 한다. 즉, 가장 거리가 먼 집부터 방문하고 돌아오면서 최대한 멀리 있는 집들의 택배 상자도 함께 수거하여 멀리 떨어진 집을 재방문 하지 않도록 하는 것이 핵심 이다.</p>
<h3>구현</h3>
<p>2개의 포인터를 두어, 각각의 포인터는 배달을 위한 가장 멀리 떨어진 집과 수거를 위한 가장 멀리 떨어진 집을 가르키도록 했다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/yeoni_/post/6488af19-6368-4ecc-a42c-c2c787209375/image.png" /></p>
<p>이때, <code>deliveries</code> 또는 <code>pickups</code>의 값이 0이라면, 굳이 방문하지 않아도 되는 집이므로 while문을 돌려 포인터를 이동시키는 작업을 사전에 추가해 주었다.</p>
<p>모든 물건을 배달 및 수거하고 다시 물류 창고로 돌아오는 왕복 거리이므로, 두 포인터 중 (더 멀리 떨어진 거리에 * 2)를 해주면 한 번 배달 및 수거하는데 이동한 거리가 된다.</p>
<p>이런식으로 2개의 포인터를 이동시켜가면서 최대 용량(CAP)만큼을 처리한 후, 다음에 배달 및 수거해야 할 집들 중 가장 멀리 떨어진 집의 인덱스를 포인터 값에 저장하는 식으로 구현하였다. 또한 배달을 모두 완료했더라도 상자를 수거해야할 집이 남아있다면(그 반대의 경우도 마찬가지) 다시 출발해야하므로 반복문을 수행할 조건은 2개 포인터 중 하나라도 0보다 클 경우로 설정하였다.</p>
<p>각 집을 한 번만 방문하면서 택배를 처리하기 때문에 시간 복잡도는 O(n)으로 문제에서 주어진 n의 범위가 1 &le; n &le; 100,000 이므로 시간 초과 또한 발생하지 않음을 알 수 있다.</p>
<h3>최종 코드</h3>
<pre class="angelscript"><code>class Solution {
    public long solution(int cap, int n, int[] deliveries, int[] pickups) {
        long distance = 0L;
        int end1 = n - 1, end2 = n - 1;

        // 배달과 수거의 끝 지점 찾기
        while (end1 &gt;= 0 &amp;&amp; deliveries[end1] == 0) {
            end1--;
        }
        while (end2 &gt;= 0 &amp;&amp; pickups[end2] == 0) {
            end2--;
        }

        while (end1 &gt;= 0 || end2 &gt;= 0) {
            // 가장 먼 곳까지 가야 함 (왕복이므로 *2를 해줌)
            distance += (Math.max(end1, end2) + 1) * 2;

            // 배달 물량 처리
            int give = cap;
            while (give &gt;= 0 &amp;&amp; end1 &gt;= 0) {
                give -= deliveries[end1];
                if (give &lt; 0) {
                    deliveries[end1] = give * -1;
                    break;
                } else {
                    end1--;
                }
            }

            // 수거 물량 처리
            int get = cap;
            while (get &gt;= 0 &amp;&amp; end2 &gt;= 0) {
                get -= pickups[end2];
                if (get &lt; 0) {
                    pickups[end2] = get * -1;
                    break;
                } else {
                    end2--;
                }
            }
        }

        return distance;
    }
}</code></pre>
<p>최종 코드는 위와 같다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/yeoni_/post/aa367bb0-5c84-471a-ae77-fe93041ab817/image.png" /></p>
<p>&nbsp;</p>
<hr contenteditable="false" />
<p>별개로 카카오 공식 해설에서는 Stack을 이용하여 풀이하고 있는데, 시간 복잡도 상 차이는 없지만 이 방법으로 연습해보는 것도 좋을 것 같다.<br /><a href="https://tech.kakao.com/posts/567">https://tech.kakao.com/posts/567</a></p>