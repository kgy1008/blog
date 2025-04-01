<h1>문제 링크</h1>
<p><a href="https://school.programmers.co.kr/learn/courses/30/lessons/92344">https://school.programmers.co.kr/learn/courses/30/lessons/92344</a></p>
<figure contenteditable="false" id="og_1743062864857"><a href="https://school.programmers.co.kr/learn/courses/30/lessons/92344" rel="noopener" target="_blank">
<div class="og-image">&nbsp;</div>
<div class="og-text">
<p class="og-title">프로그래머스</p>
<p class="og-desc">SW개발자를 위한 평가, 교육, 채용까지 Total Solution을 제공하는 개발자 성장을 위한 베이스캠프</p>
<p class="og-host">programmers.co.kr</p>
</div>
</a></figure>
<p>&nbsp;</p>
<h2>효율성 테스트 실패 코드</h2>
<p>효율성을 고려하지 않고 Brute Force 방식으로 코드를 작성한다면, 굉장히 쉬운 문제다.</p>
<pre class="angelscript"><code>class Solution {
    public int solution(int[][] board, int[][] skill) {
        for (int[] s : skill) {
            if (s[0] == 1) { // 공격
                for (int i=s[1]; i&lt;=s[3]; i++) {
                    for (int j=s[2]; j&lt;=s[4]; j++) {
                        board[i][j] -= s[5];
                    }
                }

            } else { // 회복
                for (int i=s[1]; i&lt;=s[3]; i++) {
                    for (int j=s[2]; j&lt;=s[4]; j++) {
                        board[i][j] += s[5];
                    }
                }
            }
        }

        int answer = 0;
        for (int[] b : board) {
            for (int x : b) {
                if (x &gt; 0) {
                    answer++;
                }
            }
        }

        return answer;
    }
}</code></pre>
<p>위와 같이 반복문을 돌면서, board 배열의 값들을 수정한 후, 다시 board 전체 배열의 값들을 확인하면서 파괴되지 않는 건물의 개수를 세어주면 된다. 하지만, 이 코드의 시간 복잡도는 O(N * M * K)으로 이렇게 작성할 경우, 효율성 테스트에서 모두 시간 초과가 발생하는 결과가 나오게 된다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/yeoni_/post/a7ec403d-b0cb-467b-b811-531b4595126a/image.png" /></p>
<p><br />어떻게 시간 복잡도를 줄일 수 있을까?</p>
<h2>누적 합 이용</h2>
<p>이 문제는 2차원 배열에서 구간의 변화를 어떻게 효율적으로 처리할 지가 관건인 문제로, 누적합을 이용하면 효율적으로 처리할 수 있다.</p>
<h3>1차원 배열에서의 누적 합</h3>
<p>[1,2,4,8,9]의 num 배열이 있고, 0번째부터 2번째 원소까지 2만큼 빼야 하는 상황이라고 가정해보자. 반복문을 사용해, O(M)의 시간 복잡도로 모든 원소를 순회하며 2를 빼주어도 좋지만, 이를 O(1)로 만들기 위해서는 우선 [-2,0,0,2,0,0]를 저장한 새로운 diff 배열을 생성하는 것이다. 여기서, 2번째 인덱스의 값에 2를 저장하는 것이 아니라 3번째 인덱스의 값에 2를 저장하는 것을 유의하자. 이유는 아래 사진과 같다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/yeoni_/post/322acc40-4f9e-436b-85c9-2a898cb8df63/image.png" /></p>
<p>이 diff 배열을 누적합하여([-2,-2,-2,0,0,0]) 본래의 num 배열과 더해주면 [-1,0,2,8,9]을 얻을 수 있게 된다.</p>
<h3>2차원 배열에서의 누적합</h3>
<p>이 아이디어를 2차원 배열로 확장해보자. 5x5배열에서 (0,0)~(2,2)까지 N만큼 감소시키는 경우를 생각해보자.</p>
<p><img alt="" src="https://velog.velcdn.com/images/yeoni_/post/e6edbb5b-ccb3-47d8-84d9-82df2b56014a/image.png" /></p>
<p><br />즉, 2차원 배열에서 (x1,y1)부터 (x2,y2)까지 n만큼의 변화는 (x1,y1)에 +n, (x1,y2+1)에 -n, (x2+1,y1)에 -n, (x2+1,y2+1)에 +n을 한 것과 같다.<br />해당 배열을 누적합 배열로 바꾸는데 걸리는 시간복잡도는 O(N * M), 그리고 K개의 skill을 모두 처리할 수 있는 배열을 만드는데 O(K)가 걸리게 되므로 최종적인 시간 복잡도는 O(K + N * M)이 된다.</p>
<h3>최종 코드</h3>
<pre class="angelscript"><code>class Solution {
    public int solution(int[][] board, int[][] skill) {
        int n = board.length;
        int m = board[0].length;

        int[][] diff = new int[n+1][m+1];

        for (int[] s : skill) {
            int type = s[0] == 1 ? -1 : 1; 
            int r1 = s[1], c1 = s[2], r2 = s[3], c2 = s[4];
            int degree = s[5] * type;

            diff[r1][c1] += degree;
            diff[r1][c2+1] -= degree;
            diff[r2+1][c1] -= degree;
            diff[r2+1][c2+1] += degree;
        }

        // 행 기준 누적 합
        for (int i = 0; i &lt; n; i++) {
            for (int j = 1; j &lt; m; j++) {
                diff[i][j] += diff[i][j - 1];
            }
        }

        // 열 기준 누적 합
        for (int i = 1; i &lt; n; i++) {
            for (int j = 0; j &lt; m; j++) {
                diff[i][j] += diff[i - 1][j];
            }
        }

        int answer = 0;
        for (int i = 0; i &lt; n; i++) {
            for (int j = 0; j &lt; m; j++) {
                board[i][j] += diff[i][j];
                if (board[i][j] &gt; 0) {
                    answer++;
                }
            }
        }
        return answer;
    }
}</code></pre>
<p>&nbsp;</p>
<p>이렇게 누적합을 이용하여 코드를 구현하면 효율성 테스트를 통과할 수 있다.</p>
<p>&nbsp;</p>
<p><img alt="" src="https://velog.velcdn.com/images/yeoni_/post/75da41b7-667c-4b77-bcb2-0921f742f8f0/image.png" /></p>