<h1>문제 링크</h1>
<p><a href="https://www.acmicpc.net/problem/2887">https://www.acmicpc.net/problem/2887</a></p>
<h2>고민한 점</h2>
<p>문제에서 N개의 행성과 <code>N-1개의 간선</code>, <code>모든 행성이 연결되도록</code>, <code>최소 비용</code> 이 3가지 키워드를 보면 최소 스패닝 트리 (MST) 문제구나! 라는 것이 쉽게 파악 된다.<br />다만, 주어진 N의 최대 범위는 100000 으로, 만약 모든 간선을 구하여 풀고자 한다면, 최대 간선의 개수는 $100000C2$로 시간 초과에 더해 메모리 초과까지 발생할 것이다. 때문에 두 행성 간의 터널 비용을 모두 구하는 단순한 방식으로는 이 문제를 해결할 수 없다.</p>
<h3>간선의 개수를 줄여보자.</h3>
<p>크루스칼 알고리즘 플로우를 살펴보면, 간선의 비용이 작은 간선부터 연결하여 연결된 간선의 총 개수가 n-1개가 될때까지 두 노드를 union하는 로직이다. 즉, 간선 리스트에 몇개가 저장되어 있던 연결된 간선의 개수가 n-1개가 되면 종료된다. 그렇다면 간선 리스트에 불필요하게 모든 간선을 저장하는 것이 아니라 가장 유망한 간선만 저장하면 되지 않을까?<br />문제에서 각 행성들간의 연결하는 터널의 비용은 min(|xA-xB|, |yA-yB|, |zA-zB|)이라고 정의되어 있다. 즉, <b>각 좌표 차이의 최소값</b>이 곧 행성 사이의 터널 건설 비용이므로 x,y,z 좌표를 각각 정렬한 뒤 연속된 점들끼리만 간선을 만들어도 무방하다. x, y, z를 따로 정렬한 후 가장 가까운 행성끼리만 연결하면, 어차피 MST를 만들 때 필요 없는 간선은 제외되므로 최적의 결과를 얻을 수 있기 때문이다.<br />예를 들어 주어진 행성들의 x좌표가 [1, 8, 12, 3, 7]이라고 생각해보자. 위 좌표를 오름차순 정렬하면 [1, 3, 7, 8, 12]이 된다. 이처럼 정렬을 하게 되면 값이 비슷한 점들이 연속해서 위치하게 되고 때문에 연속된 점들 간의 x좌표 차이가 곧 작을 가능성이 높기 때문에 유망한 간선이 되는 것이다. 이렇게 만들어진 서로 다른 간선의 개수는 n-1개로 항상 최소 신장 트리를 만들 수 있게 된다.</p>
<p>터널의 비용은 <b>각 좌표 차이의 최소값</b>이기 때문에 x,y,z좌표에 대해 각각 위 과정을 수행하여 간선 리스트에 저장한 후 비용을 기준으로 오름차순 정렬하고 크루스칼 알고리즘을 적용하면 된다.</p>
<hr contenteditable="false" />
<p>정수 범위에 주의하자. 각각의 좌표는 좌표는 -10억보다 크거나 같고, 10억보다 작거나 같은 정수이기 때문에 하나의 터널을 건설하는데 발생할 수 있는 최대 비용은 20억이다. 행성의 개수는 최대 10만개가 주어지므로 필요한 모든 터널을 건설하는데 들어가는 최대 비용은 약 100억이 된다. 이는 int 범위를 벗어나기 때문에 long으로 선언하여 해결해야 한다.</p>
<h2>최종 코드</h2>
<pre class="arduino"><code>import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;
import java.util.StringTokenizer;

public class Main {

    static int[] parent;
    static List&lt;Edge&gt; edges = new ArrayList&lt;&gt;();
    static long distance = 0L;

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st;

        int n = Integer.parseInt(br.readLine()); // 행성 개수
        parent = new int[n + 1];
        for (int i = 1; i &lt;= n; i++) {
            parent[i] = i;
        }

        List&lt;int[]&gt; xList = new ArrayList&lt;&gt;();
        List&lt;int[]&gt; yList = new ArrayList&lt;&gt;();
        List&lt;int[]&gt; zList = new ArrayList&lt;&gt;();

        for (int i = 1; i &lt;= n; i++) {
            st = new StringTokenizer(br.readLine());
            int x = Integer.parseInt(st.nextToken());
            int y = Integer.parseInt(st.nextToken());
            int z = Integer.parseInt(st.nextToken());

            xList.add(new int[]{x, i});
            yList.add(new int[]{y, i});
            zList.add(new int[]{z, i});
        }

        xList.sort((Comparator.comparingInt(o -&gt; o[0])));
        yList.sort((Comparator.comparingInt(o -&gt; o[0])));
        zList.sort((Comparator.comparingInt(o -&gt; o[0])));

        for (int i = 0; i &lt; n - 1; i++) {
            edges.add(new Edge(xList.get(i)[1], xList.get(i + 1)[1], xList.get(i + 1)[0] - xList.get(i)[0]));
            edges.add(new Edge(yList.get(i)[1], yList.get(i + 1)[1], yList.get(i + 1)[0] - yList.get(i)[0]));
            edges.add(new Edge(zList.get(i)[1], zList.get(i + 1)[1], zList.get(i + 1)[0] - zList.get(i)[0]));
        }

        edges.sort(Comparator.comparingLong(o -&gt; o.cost));
        kruskal();

        System.out.println(distance);
    }

    static void kruskal() {
        for (Edge edge : edges) {
            if (find(edge.to) != find(edge.from)) {
                union(edge.to, edge.from);
                distance += edge.cost;
            }
        }
    }

    static void union(int a, int b) {
        int p1 = find(a);
        int p2 = find(b);

        parent[p2] = p1;
    }

    static int find(int x) {
        if (parent[x] == x) {
            return x;
        }
        return parent[x] = find(parent[x]);
    }

    static class Edge {
        int from;
        int to;
        long cost;

        public Edge(final int from, final int to, final long cost) {
            this.from = from;
            this.to = to;
            this.cost = cost;
        }
    }
}</code></pre>
<p>모든 간선의 조합을 구하면 시간 초과가 발생한다는 것은 쉽게 눈치 챘지만, 간선을 줄이기 위해 어떻게 해야할 지 아이디어를 떠올리기가 힘들었던 문제였다. 꼭 습득하고 넘어가자.</p>
<p><img alt="" src="https://velog.velcdn.com/images/yeoni_/post/b1cfccf0-853c-4329-84e3-f9053ad53a84/image.png" /></p>