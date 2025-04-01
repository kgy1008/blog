<h1>섹션 10, 11. 객체 지향 쿼리 언어</h1>
<h2>JPQL 문법</h2>
<h3>TypeQuery와 Query</h3>
<p>TypeQuery는 반환 타입이 명확할 때 사용하는 반면, Query는 반환 타입이 명확하지 않을 때 사용한다.</p>
<pre class="stata"><code>// TypeQuery
TypedQuery&lt;Member&gt; query = em.createQuery("SELECT m FROM Member m", Member.class);

// Query
Query query = em.createQuery("SELECT m.username, m.age FROM Member m");
</code></pre>
<p>위의 코드를 살펴보자. 첫 번째 줄의 경우, 반환 타입이 Member임이 자명하다. 때문에 이 경우에는 TypeQuery를 사용해도 된다. 하지만 두 번째의 경우, username은 String 타입이지만 age는 Int 타입이다. 이처럼 반환 타입이 명확하지 않을 때는 TypeQuery 대신 Query를 사용해야 한다.</p>
<h3>결과 조회 API</h3>
<p>query.getResultList()는 결과가 하나 이상일 때 사용하며 리스트를 반환한다. 만약 결과가 존재하지 않는다면 빈 리스트를 반환한다. 반면 query.getSingleResult() 는 단일 객체를 반환한다. 때문에 결과가 정확히 하나일 때 사용해야 한다. 결과가 존재하지 않는다면 NoResultException 예외를 발생시키고 만약 결과가 둘 이상이면 NonUniqueResultException 예외를 발생시킨다.</p>
<h3>페이징</h3>
<p>페이징은 전체 목록을 한 번에 보여주지 않고 일정 개수씩 페이지를 만들고 페이지 번호에 따라 보여주는 목록을 조정해 주는 것을 말한다. setFirstResult()를 통해 조회 시작 위치를 명시할 수 있으며 setMaxResults() 메서드 사용을 통해 조회할 데이터 수를 지정할 수 있다.</p>
<h2>경로 표현식</h2>
<pre class="cs"><code>select m.username // 상태 필드
from Member m
join m.team t  // 단일 값 연관 필드
join m.orders o // 컬렉션 값 연관 필드
where t.name = 'teamA'
</code></pre>
<h3>상태 필드(state field)</h3>
<p>단순히 값을 저장하기 위한 필드이다. 더 이상의 경로 탐색을 할 수 없다. 즉, 위의 JPQL에서 m.username뒤에 다시 .을 찍어서 탐색을 이어나갈 수 없다.</p>
<h3>연관 필드(association field)</h3>
<p>연관 필드는 크게 <b>단일 값 연관 필드</b>와 <b>컬렉션 값 연관 필드</b>로 나눌 수 있다. 단일 값 연관 필드는 @ManyToOne, @OneToOne 관계처럼 대상이 엔티티(ex.m.team)일 경우를 말한다. 반면, 컬렉션 값 연관 필드는 @OneToMany, @ManyToMany관계와 같이 대상이 컬렉션(ex.m.orders)일 경우를 말한다. 연관 필드는 모두 묵시적 내부 조인(inner join)이 발생한다. 아래 JPQL을 살펴보자.</p>
<pre class="cs"><code>// 명시적 조인
select m from Member m join m.team t

// 묵시적 조인
select m.team from Member m
</code></pre>
<p>첫번째 쿼리는 명시적 조인 쿼리이며 두번째 쿼리는 묵시적 조인(경로 표현식에 의해 묵시적으로 내부 조인 발생)이다. Member 엔티티와 Team 엔티티는 연관관계 매핑이 되어 있기 때문에 따로 명시하지 않아도 묵시적으로(Inner)Join쿼리가 발생하는 것이다. 하지만 Join 쿼리가 발생한다는 것을 가시적으로 확인할 수 있도록 명시적 조인을 사용하는 것을 권장한다.</p>
<hr />
<p><b>단일 값 연관 필드</b>의 경우 탐색을 이어나갈 수 있다. 즉, m.team뒤에 다시 .을 찍어서 탐색을 이어나갈 수 있다. 반면 <b>컬렉션 값 연관 필드</b>는 상태 필드와 마찬가지로 탐색을 이어나갈 수 없다. 즉,t.members 뒤에 .username 을 사용할 수 없다. 만약 탐색을 이어나가고 싶다면 아래와 같이 명시적 조인을 사용하여 별칭을 만들어 이용하도록 쿼리문을 작성해주어야 한다.</p>
<pre class="sqf"><code>select m.username from Team t join t members m
</code></pre>
<h2>FetchJoin</h2>
<p>페치 조인(Fetch join)은 연관된 엔티티나 컬렉션을 SQL 한 번에 함께 조회하는 기능으로 join fetch 명령어를 사용한다. Fetch join을 사용함으로써 JPA N+1문제를 방지할 수 있다.</p>
<h3>엔티티 페치 조인</h3>
<p>먼저 엔티티 페치 조인을 살펴보자.</p>
<pre class="sql"><code>// JPQL
select m from Member m join fetch m.team

// SQL
SELECT M*, T* FROM MEMBER M INNER JOIN TEAM T ON M.TEAM_ID = T.ID
</code></pre>
<p>위의 JPQL문에서는 Member 엔티티만을 select한다고 작성하였지만 join fetch를 함으로써 실제 SQL문은 Member와 Team 테이블을 모두 조회하는 쿼리가 나가게 된다. 이처럼 페치 조인을 사용하면 연관된 엔티티를 함께 조회할 수 있다.</p>
<h3>컬렉션 페치 조인</h3>
<p>컬렉션 페치 조인은 말 그대로 연관된 컬렉션을 함께 조회하는 것을 말한다. @OneToMany 관계와 같이 대상이 컬렉션일 경우를 아래와 같이 컬렉션 페치 조인을 사용할 수 있다.</p>
<pre class="sql"><code>// JPQL
select t from Team t join fetch t.members

// SQL
SELECT T*, M* FROM TEAM T INNER JOIN MEMBER m ON T.ID = M.TEAM_ID
</code></pre>
<p>하지만 컬렉션 페치 조인의 경우, 데이터 복제 현상이 발생한다.</p>
<p><figure class="imageblock alignCenter"><span><img height="1228" src="https://blog.kakaocdn.net/dn/rETHz/btsMMAAnAgN/8w1yEkbQZk6xQZ6xTmHbYk/img.png" width="1000" /></span></figure>
</p>
<p>Team과 Member 엔티티를 조인했을 때, Team 관점에서 보면 데이터가 중복 생성되었다. 즉, 팀A인 데이터가 여러개 존재한다. 특히, 영속성 컨텍스트는 PK값을 기준으로 데이터를 관리하기 때문에 첫 번째 데이터와 두 번째 데이터는 영속성 컨텍스트 내에서는 같은 값으로 인식된다.</p>
<pre class="less"><code>for(Team team : teams) {
	System.out.println("teamname = " + team.getName() + ", team = " + team);
	for (Member member : team.getMembers()) {
		System.out.println(&ldquo;-&gt; username = " + member.getUsername()+ ", member = " + member);
}

/*
teamname = 팀A, team = Team@0x100
-&gt; username = 회원1, member = Member@0x200
-&gt; username = 회원2, member = Member@0x300
teamname = 팀A, team = Team@0x100
-&gt; username = 회원1, member = Member@0x200
-&gt; username = 회원2, member = Member@0x300
*/
</code></pre>
<p>따라서 위의 코드를 실행시키면 팀A에 대한 정보가 2번 중복으로 출력이 된다. 이처럼 발생하는 중복을 제거하기 위해서는 distinct 명령어를 사용해야 한다. SQL의 distinct는 데이터가 완전히 일치하는 경우만 중복으로 인식하여 제거를 한다. 때문에 순수 SQL distinct 쿼리로는 중복 제거가 불가능하다.</p>
<p><figure class="imageblock alignCenter"><span><img height="772" src="https://blog.kakaocdn.net/dn/dbmdAr/btsMOdqpt6E/Zrd3XkORLqzRoXGAM0MR3K/img.png" width="2048" /></span></figure>
</p>
<h3>페치 조인의 한계</h3>
<p>JPA N+1 문제를 방지하는 페치 조인은 장점만 존재할까? 페치 조인에도 분명한 한계가 존재한다. 먼저 <b>페치 조인 대상에는 별칭을 사용할 수 없다.</b> 엄밀히 말하면 사용할 수는 있지만 권장되지 않는다.</p>
<hr contenteditable="false" />
<p>두번째로 <b>둘 이상의 컬렉션은 페치 조인할 수 없다</b>. 컬렉션 페치 조인이 2개 이상이 될 경우 너무 많은 값이 메모리로 들어와 MultipleBagFetchException이 발생하게 된다. 코드로 살펴보자.</p>
<pre class="less"><code>@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private List&lt;Article&gt; articles = new ArrayList&lt;&gt;();
    
    @OneToMany(mappedBy = "question", fetch = FetchType.LAZY)
    private List&lt;Question&gt; questions = new ArrayList&lt;&gt;();
    
}
</code></pre>
<p>위의 코드에서 User 엔티티는 Article, Question 2개의 엔티티와 일대다 관계를 가지고 있다. 이때 2개의 엔티티에 대해 페치 조인을 하면 어떻게 될까? 즉, 아래 코드처럼 2개의 컬렉션 페치 조인을 실행하면 어떻게 될까?</p>
<pre class="n1ql"><code>@Query("select distinct u from User u left join fetch u.articles left join fetch u.questions")
List&lt;User&gt; findAllWithArticlesAndQuestions();
</code></pre>
<p>xToMany관계가 두 개, 즉 컬렉션 페치 조인이 두 개 이상 걸리기 때문에 MultipleBagFetchException이 발생하게 된다.</p>
<blockquote>
<p>Bag 컬렉션은 Hibernate에서 사용하는 용어로 순서가 없고 키가 없으며, 중복을 허용한다. 하지만 Java Collection에는 Bag이 구현되어있지 않아 List를 사용한다고 한다.</p>
</blockquote>
<p>&nbsp;</p>
<p>그럼 어떻게 해결할 수 있을까? @BatchSize 옵션을 사용하면 해결이 가능하다. 이때, @BatchSize이 적용된 컬렉션에 fetch join을 걸면 안된다. fetch join이 우선시되어 적용되기 때문에 batch size가 무시되기 때문이다.</p>
<hr contenteditable="false" />
<p>마지막으로 <b>컬렉션을 페치 조인하면 페이징 API를 사용할 수 없다.</b> 일대일, 다대일 같은 단일 값 연관 필드들은 페치 조인을 사용해도 페이징이 가능하지만 일대다와 같은 컬렉션 페치 조인은 페이징이 불가능하다. 이 점이 페치 조인의 가장 큰 한계점이다. 왜 컬렉션 페치 조인은 페이징이 불가능할까?</p>
<p>Team과 Member는 1:N 관계를 갖고 있는 상태에서 컬렉션 페치 조인을 했다고 가정해보자.</p>
<p><figure class="imageblock alignCenter"><span><img height="430" src="https://blog.kakaocdn.net/dn/dEe9CE/btsMNbUoE2u/DKCKbcSxmvfuvc7Krn88rK/img.png" width="1280" /></span></figure>
</p>
<p>그럼 위와 같은 결과가 나올 것이다. 이때, Team을 기준으로 페이징을 하고 싶은데 결과는 Member의 개수에 맞춰져 있다. 아까와 언급한 데이터 중복 생성 문제가 발생한 것이다. 따라서 DB 입장에서는 페이징을 해야하는 기준과 결과 row가 달라서 페이징을 할 수 없게 된다. 그럼 @OneToMany 관계에서는 페이징이 불가능한 것일까? 다행히 이어서 설명할, 하이버네이트에서 제공하는 @BatchSize 옵션을 사용하면 컬렉션 엔티티 조회와 페이징을 같이 사용할 수 있다.</p>
<h3>BatchSize</h3>
<pre class="less"><code>@Entity
public class Team {
	
    // ...

    @BatchSize(size = 100) // size는 일반적으로 100~1000을 사용한다. 
    @OneToMany(mappedBy = "team")
    private final List&lt;Member&gt; members = new ArrayList&lt;&gt;();
}
</code></pre>
<p>컬렉션 엔티티 조회 시 발생하는 N+1 문제를 해결하기 위해,&nbsp;@OneToMany로 매핑된 컬렉션 엔티티 필드에 @BatchSize 어노테이션을 사용하였다. @BatchSize를 사용하면, 연관된 엔티티 조회 시 지정한 size 만큼&nbsp;IN&nbsp;쿼리를 사용하여 조회한다. 즉, 기존에는 &lsquo;Team을 조회하는 쿼리 1개&rsquo; 와 &lsquo;조회된 N개의 Team에 대해 연관된 엔티티(Member)를 조회하는 쿼리 N개&rsquo; 로 인해 N+1 문제가 발생했다면, @BatchSize를 적용한 후에는 &lsquo;Team을 조회하는 쿼리 1개&rsquo; 와 &lsquo;조회된 N개의 Team에 대해 연관된 컬렉션 엔티티를 조회하는&nbsp;IN&nbsp;쿼리 1개&rsquo;의 쿼리가 실행된다. 즉,&nbsp;1 + N&nbsp;번 실행 되었던 쿼리가&nbsp;1 + 1번 실행됨으로써 N+1 문제를 해결한 것이다. 요약하면, Batch Size 옵션을 이용하면 프록시 초기화 발생 시 SQL의 IN절을 실행함으로써 JPA N+1 문제를 해결하고, 컬렉션 페치 조인과 페이징을 함께 사용 시 발생하는 메모리 페이징 문제를 해결할 수 있다.</p>
<hr />
<p><a href="https://velog.io/@balparang/JPA-컬렉션-엔티티와-페이징을-함께-사용하기-feat.-BatchSize#페치-조인의-한계-컬렉션을-페치-조인하면-페이징이-불가능">https://velog.io/@balparang/JPA-컬렉션-엔티티와-페이징을-함께-사용하기-feat.-BatchSize#페치-조인의-한계-컬렉션을-페치-조인하면-페이징이-불가능</a></p>
<h2>벌크 연산</h2>
<p>JPA에는 변경 감지 기능이 있어 Update 쿼리문을 날리지 않고도 자동으로 데이터를 수정할 수 있다고 배웠다. 하지만 변경 감지 기능으로 실행할 데이터가 너무나 많다면, 즉 한 번에 다건의 데이터를 수정해야 한다면 어떻게 해야할까? 이때 사용하는 것이 벌크 연산이다. 즉, 벌크 연산을 통해 쿼리 한 번으로 여러 테이블의 row를 변경할 수 있다. 하지만 벌크 연산은 영속성 컨텍스트를 무시하고 DB에 직접 쿼리를 날리기 때문에 주의해야 한다.</p>
<h3>주의점</h3>
<p>언급했듯, 벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리를 날린다. 때문에 영속성 컨텍스트를 초기화하지 않을 경우 DB와 영속성 컨텍스트간의 데이터 불일치가 발생할 수 있다. 아래 예시를 살펴보자.</p>
<blockquote>
<ol>
<li>트랜잭션 시작</li>
</ol>
</blockquote>
<ol>
<li>영속성 컨텍스트에 나이가 20인 직원 1, 직원 2 등록</li>
<li>벌크 연산을 사용하여 나이가 20인 직원 연봉 인상</li>
<li>findByAge(int age)로 나이가 20인 직원들을 가져옴</li>
<li>해당 직원들의 데이터를 클라이언트에게 보내줌</li>
<li>트랜잭션 커밋</li>
</ol>
<p>3번째 과정에서 벌크 연산을 통해 DB는 변경이 되었지만 영속성 컨텍스트는 이전과 같은 상태를 유지한다. 즉, DB에는 나이가 20인 직원들의 연봉이 인상되었지만 영속성 컨텍스트의 1차 캐시에는 여전히 인상되지 않은 데이터가 남아있는 것이다. 이때 영속성 컨텍스트를 초기화(em.clear())하지 않고 findByAge(int age) 메서드를 사용하여 데이터를 조회한다면, 영속성 컨텍스트의 1차 캐시 정보를 가져오게 되어 DB와 영속성 컨텍스트간의 데이터 불일치가 발생하게 된다. 이를 해결하기 위해서는 벌크 연산을 먼저 실행(flush)하고 <b>벌크 연산 수행 후에는 반드시 영속성 컨텍스트를 초기화하는 과정을 거쳐야 한다</b>. 영속성 컨텍스트를 초기화하게 되면 1차 캐시에는 아무 정보도 남아있지 않으므로 데이터를 조회할 때, DB에 select 쿼리를 날려 조회하기 때문에 데이터 불일치 문제를 해결할 수 있다.</p>