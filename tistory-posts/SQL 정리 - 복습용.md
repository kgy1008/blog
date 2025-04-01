<h1>SQL 동작 순서</h1>
<blockquote>
<p>  'FROM -&gt; WHERE -&gt; GROUP BY -&gt; HAVING -&gt; SELECT -&gt; ORDER BY'</p>
</blockquote>
<h1>정규 표현식</h1>
<p><img alt="" src="https://velog.velcdn.com/images/yeoni_/post/d6c6a531-6a25-48ad-b868-fc79461c0d61/image.png" /></p>
<p><a href="https://schatz37.tistory.com/39">https://schatz37.tistory.com/39</a></p>
<h1>CTE</h1>
<p>CTE는 쿼리 내에서 임시 테이블을 정의하여, 후속 SELECT문에서 사용할 수 있도록 하는 구조<br />-&gt; 임시로 쿼리 결과를 저장해 놓고, 여러번 참조해서 사용하는 용도로 사용</p>
<h2>사용 방법</h2>
<p>WITH 키워드와 AS로 cte_name에 맵핑할 쿼리를 작성한 후 cte_name으로 지정한 테이블을 조회한다.</p>
<pre class="sql"><code>WITH CTE_NAME AS (
    -- CTE 정의 부분
    SELECT column1, column2, ...
    FROM some_table
    WHERE condition
)
-- CTE 사용 부분
SELECT * FROM CTE_NAME;
</code></pre>
<h3>여러 CTE 사용</h3>
<p>여러 개의 CTE를 콤마(,)로 구분하여 동시에 정의할 수 있음.<br />각 CTE는 쿼리에서 독립적으로 사용할 수 있다.</p>
<pre class="sql"><code>WITH 
    TotalSales AS (
        SELECT SUM(amount) AS total_amount
        FROM sales
    ),
    AverageSales AS (
        SELECT AVG(amount) AS average_amount
        FROM sales
    )

-- CTE 사용
SELECT total_amount, average_amount
FROM TotalSales, AverageSales;</code></pre>
<h1>재귀 쿼리</h1>
<p><a href="https://inpa.tistory.com/entry/MYSQL-%F0%9F%93%9A-RECURSIVE-%EC%9E%AC%EA%B7%80-%EC%BF%BC%EB%A6%AC">https://inpa.tistory.com/entry/MYSQL-%F0%9F%93%9A-RECURSIVE-%EC%9E%AC%EA%B7%80-%EC%BF%BC%EB%A6%AC</a></p>
<h2>사용 방법</h2>
<p><code>WITH RECURSIVE</code> 쿼리문을 작성하고 내부에 <code>UNION</code>을 통해 재귀를 구성</p>
<pre class="sql"><code>WITH RECURSIVE cte_count 
AS ( 
    -- Non-Recursive 문장( 첫번째 루프에서만 실행됨 )
    SELECT 1 AS n
    UNION ALL
    -- Recursive 문장(읽어 올 때마다 행의 위치가 기억되어 다음번 읽어 올 때 다음 행으로 이동함)
    SELECT n + 1 AS num 
    FROM cte_count
    WHERE n &lt; 3 
)

SELECT * FROM test;</code></pre>
<h2>별칭</h2>
<h3>칼럼(column)에 별칭 사용하기</h3>
<pre class="sql"><code>SELECT mem_id AS "아이디", addr AS "주소" FROM member;</code></pre>
<h3>테이블(Table)에 별칭 사용하기</h3>
<pre class="sql"><code>SELECT * FROM member as "개인정보";</code></pre>
<hr />
<p>별칭을 지정해줄 때 띄어쓰기가 들어간다면, 큰 따옴표(")로 묶어주어야 한다. 또한 테이블 별칭은 WHERE 절에서 사용 가능하지만 <b>열(칼럼) 별칭은 WHERE 절에서 사용 불가능</b>하다. (SELECT절보다 WHERE 절이 먼저 실행되기 때문)</p>
<h1>기타 함수</h1>
<h2>✅ 문자열 자르기</h2>
<h3>SUBSTRING( 문자열, 시작위치, 길이 )</h3>
<p>문자열에서 시작 위치부터 길이만큼 출력</p>
<h3>LEFT( 문자열, 길이 )</h3>
<p>문자열에서 왼쪽부터 길이만큼 출력</p>
<h3>RIGHT( 문자열, 길이 )</h3>
<p>문자열에서 오른쪽부터 길이만큼 출력</p>
<h2>✅ 날짜와 시간의 연산 -&gt; DATE_ADD()</h2>
<pre class="gcode"><code> DATE_ADD(연산을 수행할 날짜, INTERVAL 더하거나 빼고자 하는 값 단위)
</code></pre>
<p>주요 단위로는 <code>YEAR</code>, <code>MONTH</code>, <code>DAY</code>, <code>HOUR</code>, <code>MINUTE</code>, <code>SECOND</code> 등이 있다.</p>
<h3>사용 예시</h3>
<pre class="sql"><code>SELECT DATE_ADD(NOW(), INTERVAL 1 DAY) AS tomorrow;</code></pre>
<h2>✅ <code>TRUNCATE(number, decimal_places)</code></h2>
<ul>
<li><code>number</code>: 변환할 숫자</li>
<li><code>decimal_places</code>: 유지할 소수점 이하 자리 수 (또는 정수 자리수 조정)
<ul>
<li><b>양수(+)</b> &rarr; 소수점 이하 자리 유지</li>
<li><b>0</b> &rarr; 소수점 이하 버림 (정수로 변환)</li>
<li><b>음수(-)</b> &rarr; 정수 부분에서 해당 자리 이하를 0으로 만들고 버림</li>
</ul>
</li>
</ul>
<hr />
<h3>사용 예시</h3>
<pre class="sql"><code>SELECT 
    TRUNCATE(123.456, 2) AS TRUNC_2,   -- 소수점 2자리 유지
    TRUNCATE(123.456, 0) AS TRUNC_0,   -- 정수 변환 (소수점 이하 버림)
    TRUNCATE(123.456, -1) AS TRUNC_-1, -- 10 단위로 자름
    TRUNCATE(123.456, -2) AS TRUNC_-2; -- 100 단위로 자름</code></pre>
<h3>  <b><code>TRUNCATE()</code> vs <code>ROUND()</code> 차이</b></h3>
<table>
<thead>
<tr>
<th>함수</th>
<th>동작 방식</th>
<th>예제 (<code>123.456</code>)</th>
<th>예제 (<code>98.765</code>)</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>ROUND(값, 2)</code></td>
<td>반올림</td>
<td><code>123.46</code></td>
<td><code>98.77</code></td>
</tr>
<tr>
<td><code>TRUNCATE(값, 2)</code></td>
<td>버림</td>
<td><code>123.45</code></td>
<td><code>98.76</code></td>
</tr>
<tr>
<td><code>ROUND(값, -1)</code></td>
<td>10 단위 반올림</td>
<td><code>120</code></td>
<td><code>100</code></td>
</tr>
<tr>
<td><code>TRUNCATE(값, -1)</code></td>
<td>10 단위 버림</td>
<td><code>120</code></td>
<td><code>90</code></td>
</tr>
</tbody>
</table>
<p>  <b>즉, <code>ROUND()</code>는 반올림, <code>TRUNCATE()</code>는 그냥 자름!</b></p>
<h2>✅ <code>COALESCE(A,B)</code></h2>
<p>인자로 주어진 컬럼들 중에서 NULL이 아닌 첫 번째 값을 반환하는 함수<br />A 컬럼 값이 NULL 값이 아닌 경우 A 값을 리턴하고 A가 NULL이고 B가 NULL이 아닌 경우 B 값을 리턴함. 만약 모든 인수가 NULL이면 NULL을 반환한다.</p>
<h2>✅ SET</h2>
<p>특정 변수 값을 설정할 때 사용됨</p>
<pre class="gams"><code>SET @myVar := 10;  -- 변수 설정 (MySQL)</code></pre>
<p><code>:=</code>은 대입 연산자로 비교 연산자인 <code>=</code>과 구분하기 위해 앞에 <code>:</code>를 붙인다.</p>
<h1>윈도우 함수</h1>
<h2>  RANK()</h2>
<p>특정 기준에 따라 순위를 매기는 함수 -&gt; 같은 값이 있으면 같은 순위를 부여하고, 다음 순위는 건너뛴다. 즉, 3,3 다음은 5로 순위가 시작됨<br /><code>PARTITION BY</code>를 사용하면 그룹별로 순위를 매길 수 있음</p>
<pre class="sql"><code>SELECT 
    이름, 부서, 급여,
    RANK() OVER (ORDER BY 급여 DESC) AS 순위
FROM 직원;</code></pre>
<h2>  DENSE_RANK()</h2>
<p>RANK()와 기능과 사용방법은 똑같지만 동일한 값이 있으면 같은 순위를 부여하지만 건너뛰지 않음. 즉 3,3 다음은 4등</p>
<pre class="n1ql"><code>with salary_rank_cte as (
    select id,
        dense_rank() over(partition by departmentId order by salary desc) as salary_rank
    from Employee
)
select d.name as Department, e.name as Employee, e.salary as Salary
from Employee e join Department d on e.departmentId = d.id join salary_rank_cte s on s.id = e.id
where s.salary_rank &lt;= 3</code></pre>
<p>이렇게 <code>partition by</code>와 혼용해서 사용할 수도 있다. (다른 윈도우 함수들도 가능)</p>
<h2>  ROW_NUMBER()</h2>
<p>고유한 순위를 부여함. 값이 동일해도 다른 순위를 부여한다는 뜻</p>
<pre class="pgsql"><code>SELECT name, salary, 
       ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num
FROM employees;</code></pre>
<h2>  LAG()</h2>
<p>이전 행의 값을 가져온다. 사용법은 아래와 같다.</p>
<pre class="pgsql"><code>LAG(컬럼명, 이동할 행 수, 기본값) OVER (ORDER BY 정렬기준)</code></pre>
<p>이때, 기본값을 지정하지 않으면 이전 행이 존재하지 않을 경우 NULL로 처리된다.</p>
<h3>예시</h3>
<pre class="pgsql"><code>SELECT name, salary, 
       LAG(salary, 1, 0) OVER (ORDER BY salary DESC) AS prev_salary
FROM employees;</code></pre>
<h2>  LEAD()</h2>
<p>다음 행의 값을 가져옴</p>
<pre class="pgsql"><code>LEAD(컬럼명, 이동할 행 수, 기본값) OVER (ORDER BY 정렬기준)</code></pre>
<p>이때, 기본값을 지정하지 않으면 NULL로 처리된다.</p>
<h3>예시</h3>
<pre class="sql"><code>SELECT name, salary, 
       LEAD(salary, 1, 0) OVER (ORDER BY salary DESC) AS next_salary
FROM employees;</code></pre>
<hr />
<p>SQL에 부족함을 느껴서 3일간 프로그래머스 SQL 고득점 KIT를 모두 풀었다.. 계속 복습해서 익숙해지자!  </p>
<p><img alt="" src="https://velog.velcdn.com/images/yeoni_/post/383ae4b5-b45c-42e5-b4c2-eccd36bbb4bc/image.png" /></p>