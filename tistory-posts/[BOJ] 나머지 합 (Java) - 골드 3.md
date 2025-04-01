<h1 style="color: #000000; text-align: start;">문제 링크</h1><p><a href="https://www.acmicpc.net/problem/10986" target="_blank"><span>https://www.acmicpc.net/problem/10986</span></a><br />&nbsp;<br /><span style="color: #000000;">사실 문제를 보자마자 너무 놀랐다. 삼성 SDS Pro 시험 2차 시험 문제가 거의 똑같았기 때문이다. 그리고 티어를 보고 다시 한번 놀랐다. 시험장에서 4시간 동안 고민했던 문제가 고작 골드3이라니.. 뭔가 허무하기도 했고 실력이 많이 늘었다고 생각했는데 아직 많이 부족하구나 라는 겸손이 생기게 된 문제였다... </span><span style="color: #666666;">(+ 시험장에서 쫄지 말자!)</span><span style="color: #000000;"> 아직 문제 푸는 속도도 그렇고 아이디어를 떠올리는 것도 그렇고 골드 위주의 문제들을 많이 풀어봐야 할 것 같다.</span></p><h3>문제 접근</h3><p><span style="color: #000000;">이 문제는 모든 경우를 구하고자하면 시간초과가 발생하게 된다. N의 범위를 살펴보면&nbsp;</span><span style="background-color: #ffffff;"><span style="color: #000000;">1 ≤ N ≤ 10^6로 전체 구간 길이의 경우의 수는 NC2이기 때문이다. 때문에 효율적으로 구할 수 있는 방법을 생각해야 한다. 일단 문제에서도 </span></span><span style="background-color: #ffffff;"><span style="color: #000000;">'</span></span><span style="background-color: #ffffff;"><span style="color: #000000;">연속된 부분 구간의 합'에 대한 문제이기 때문에 자연스럽게 Prefix Sum알고리즘을 활용하는 방법을 떠올렸다.&nbsp;</span></span></p><h4><span style="background-color: #ffffff;"><span style="color: #000000;">누적 합 + 나머지&nbsp;</span></span></h4><p><span style="background-color: #ffffff;"><span style="color: #000000;">구간의 합이 M으로 나누어 떨어지를 판단하기 때문에 처음부터 나머지</span></span><span style="background-color: #ffffff;"><span style="color: #666666;">(누적 합 % M)</span></span><span style="background-color: #ffffff;"><span style="color: #000000;">을 저장해보자. 입력을 테스트 케이스와 동일하게 입력한다면 아래와 같이 저장될 것이다.</span></span></p><figure class="imageblock alignCenter"><span><img height="116" src="https://blog.kakaocdn.net/dn/UV1CV/btsMWXg4CmZ/oT8dN9cFBLF1uMACRmlwGk/img.png" width="682" /></span></figure>
<p>&nbsp;<br /><span style="color: #000000;">구간합의 특성을 생각해보면 Ai+Ai+1+⋯+Aj=Sj−Si−1 가 만족함을 알 수 있다. 즉, Prefix Sum 배열에 저장된 두 값이 같다면(= 나머지가 같다면) 그 구간 합은 M으로 나누어 떨어지게 되는 것이다. 왜 이것이 성립할까? 수식으로 살펴보자.</span><br /><span style="color: #000000;">우리가 구하고자 하는 조건을 수식으로 표현한다면 아래와 같다.</span></p><pre class="shell"><code>(prefixSum[j] - prefixSum[i-1]) % m = 0 

// 나머지 연산 성질 이용
(prefixSum[j] % m) - (prefixSum[i-1] % m) = 0

(prefixSum[j] % m) = prefixSum[i-1] % m</code></pre><p>&nbsp;<br /><span style="color: #000000;">즉, 위 문제는 같은 나머지를 갖는 두 구간을 찾으면 되는 문제로 바뀐다.&nbsp;</span></p><h4><span style="color: #000000;">구현 과정</span></h4><p><span style="color: #000000;">어떻게 효율적으로 구현할 수 있을까 많이 고민했다. 누적합 배열을 만들지 않고, HashMap을 이용하여 개수를 세아린다면 시간 복잡도 측면이나 공간 복잡도 측면에서도 효율적일 것이라 판단했고 때문에 구간합을 다 저장하는 대신, 그 이전 값만을 저장하는 변수와 나머지 값의 개수를 key-value로 저장하는 HashMap 자료구조를 활용하여 구현하였다. </span><span style="color: #000000;">이때, 주의해야할 점은 문제 조건에서 i&lt;=j라고 하였으므로 i=j일 경우도 포함해주어야 한다. 때문에 반복문을 돌기 전, remainderCount.put(0, 1); 을 해주어 초기값이 0인 값들도 셀 수 있도록 처리하였다. 또한,</span><span style="color: #000000;">&nbsp;만약 모든 구간이 조건을 만족한다면 최종 정답이 int의 범위를 초과하기 때문에 long으로 선언해주어야 한다.</span></p><h2>최종 코드</h2><pre class="java"><code>import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.HashMap;
import java.util.Map;
import java.util.StringTokenizer;

public class Main {
&nbsp;&nbsp;&nbsp;&nbsp;public static void main(String[] args) throws IOException {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;StringTokenizer st = new StringTokenizer(br.readLine());

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int n = Integer.parseInt(st.nextToken());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int m = Integer.parseInt(st.nextToken());

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;st = new StringTokenizer(br.readLine());

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;long result = 0L;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Map&lt;Integer, Integer&gt; remainderCount = new HashMap&lt;&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;remainderCount.put(0, 1);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int prefixMod = 0;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;for (int i = 0; i &lt; n; i++) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int num = Integer.parseInt(st.nextToken());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;prefixMod = (prefixMod + (num % m)) % m;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result += remainderCount.getOrDefault(prefixMod, 0);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;remainderCount.put(prefixMod, remainderCount.getOrDefault(prefixMod, 0) + 1);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(result);
&nbsp;&nbsp;&nbsp;&nbsp;}
}</code></pre><p>&nbsp;</p><figure class="imageblock alignCenter"><span><img height="63" src="https://blog.kakaocdn.net/dn/dzFoOZ/btsMWEPE1pR/756mRaZq0ioWKR2W1Xv4A0/img.png" width="694" /></span></figure>
<p>&nbsp;<br /><span style="color: #000000;">삼성 Pro 2차 시험 전에 이 문제를 풀고 들어갔으면 어땠을까 후회가 남는다.. 거의 똑같은 문제라고 봐도 무방할 정도로 문제를 푸는데 필요한 아이디어가 동일했다. 기대했던 1차를 불합격하고 낙담하여 그냥 보내버린 시간이 너무 아쉽다...ㅠ 역시 실패를 경험했을 때 빨리 극복하는 '회복 탄력성'도 개발자의 중요한 덕목인 것 같다. 좌절에 빠져 앞으로의 기회도 같이 날리지 말자!</span></p>