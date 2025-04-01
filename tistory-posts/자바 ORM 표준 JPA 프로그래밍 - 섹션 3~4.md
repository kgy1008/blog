<h1>섹션 3. 영속성 관리 - 내부 동작 방식</h1>
<h2>엔티티의 생명주기</h2>
<p><figure class="imageblock alignCenter"><span><img height="1000" src="https://blog.kakaocdn.net/dn/bzK6dY/btsMK76YzqO/wRnkpLAYDhyEbbFE80eID0/img.png" width="1579" /></span></figure>
</p>
<ul>
<li>비영속 (new/transient) : 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태</li>
<li>영속 (managed) : 영속성 컨텍스트에 관리되는 상태</li>
<li>준영속 (detached) : 영속성 컨텍스트에 저장되었다가 분리된 상태</li>
<li>삭제 (removed) : 삭제된 상태</li>
</ul>
<h2>영속성 컨텍스트</h2>
<blockquote>영속성 컨텍스트란, 서버와 DB 중간에서 객체를 보관하는 세미 DB</blockquote>
<p>영속성 컨텍스트는 엔티티 매니저를 통해서 접근할 수 있으며 1차 캐시와 쓰기 지연 SQL 저장소가 존재한다.</p>
<h3>1차 캐시</h3>
<pre class="fsharp"><code>//엔티티를 생성한 상태(비영속)
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");

//엔티티를 영속
em.persist(member);
</code></pre>
<p>persist 메소드를 실행시키면 바로 쿼리문이 나갈까? 아니다!</p>
<p>persist 메소드는 말 그대로 영속화를 하는 것으로 DB에 반영하는 것은 아니다. 대신 영속성 컨텍스트 안의 1차 캐시에 객체가 저장되게 된다. 그럼 쿼리문은 언제 실행이 될까? 먼저 조회의 경우를 살펴보자.</p>
<p>만약에 1차 캐시에는 member2의 객체가 존재하지 않지만 DB에는 존재할 경우, 아래 코드를 실행시켜보자.</p>
<pre class="reasonml"><code>Member findMemeber2 = em.find(Member.class, "member2")
</code></pre>
<p>find 메소드가 실행되며 1차 캐시를 먼저 탐색한다. 1차 캐시에는 member2의 객체가 존재하지 않기 때문에 DB를 조회하게 된다. 즉, 이때 select 쿼리가 1번 나가게 된다. 이렇게 DB에서 찾은 객체는 1차 캐시에 저장이 된 후, 우리에게 반환이 된다.</p>
<pre class="reasonml"><code>Member findMemeber2Again = em.find(Member.class, "member2")
</code></pre>
<p>그 이후, 다시 member2 객체를 찾을 때에는 1차 캐시에 member2 객체가 존재하기 때문에 DB에 직접 접근하지 않아도 되어 select문 쿼리가 나가지 않게 된다.</p>
<h3>쓰기 지연 SQL 저장소</h3>
<p>영속성 컨텍스트에는 1차 캐시와 함께 쓰기 지연 SQL 저장소가 존재한다.</p>
<p><figure class="imageblock alignCenter"><span><img height="1000" src="https://blog.kakaocdn.net/dn/b5kLpB/btsMLZHdaeT/BVu9YKK5a96NNb3d9hpFkk/img.png" width="1768" /></span></figure>
</p>
<p>persist 메소드를 통해 객체를 영속화하면 1차 캐시에 저장되는 동시에 쓰기 지연 SQL 저장소에도 SQL 쿼리문이 생성되어 저장된다. persist를 할 때마다 쓰기 지연 SQL 저장소에는 생성된 SQL 쿼리문이 차곡차곡 쌓이게 되고 그 후 트랜잭션이 커밋(transaction.commit())되면 비로소 DB에 반영이 된다. (쿼리가 나가게 됨)</p>
<p>엔티티를 삭제할 경우에도 위와 같은 로직으로 트랜잭션 커밋 시점에 delete 쿼리가 나가게 된다.</p>
<h3>변경 감지 (Dirty Checking)</h3>
<p>만약, 생성된 데이터에 변경이 생겼다면 이를 다시 persist 메소드를 작성 해주어야 할까? 아니다.</p>
<p>그럼 어떻게 JPA는 객체의 변경을 감지할 수 있을까?</p>
<p><figure class="imageblock alignCenter"><span><img height="999" src="https://blog.kakaocdn.net/dn/dd1Mq0/btsMLZNZTg9/4zcsNBkDA3qWJsh45cPvM1/img.png" width="1781" /></span></figure>
</p>
<p>1차 캐시에는 스냅샷을 따로 저장해주는 공간이 있다. 스냅샷에는 변경되기 전의 상태가 저장이 되며 flush함수(영속성 컨텍스트의 변경내용을 데이터 베이스에 반영하는 것)가 실행될 때, 엔티티와 스냅샷을 비교하고 자동으로 update 쿼리문을 생성해 쓰기 지연 SQL 저장소에 생성을 한다. 이렇게 생성된 쿼리들을 데이터베이스에 전송하여 변경사항들은 DB에 반영이 된다. 이때, flush는 영속성 컨텍스트를 비우지 않는다. 단지 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화하는 작업이 flush이다.<b></b></p>
<blockquote><b>영속성 컨텍스트를 플러시(flush)하는 방법</b><br />- em.flush() : 직접 호출<br />- 트랜잭션 커밋 : transaction.commit()시에 자동으로 flush가 호출<br />- JPQL 쿼리 실행 : JPQL 쿼리가 실행되면 flush가 자동으로 호출</blockquote>
<h3>트랜잭션 격리 수준</h3>
<blockquote>트랜잭션이란, 데이터베이스의 상태를 변화시키는 작업 단위이다</blockquote>
<p>&nbsp;</p>
<p>트랜잭션 격리 수준이 필요한 이유는 무엇일까? 여러 트랜잭션이 같은 데이터를 동시에 읽고 쓸 때에는 수 많은 동시성 문제가 발생할 수 있다. 이런 동시성의 문제를 해결하기 위해서 트랜잭션의 격리가 필요해지게 되었고 이때, 트랜잭션을 완전히 격리할 경우 처리되는 속도가 느려진다는 문제점이 존재하기 때문에 준수한 처리 속도를 위해 격리성 수준을 나누게 되었다. 트랜잭션의 격리 수준은 총 4가지가 존재한다.</p>
<ol>
<li><b>Uncommitted&nbsp;Read (커밋되지 않은 읽기)</b> 가장 저 수준의 격리 수준인 Uncommitted Read는 다른 트랜잭션에서 커밋되지 않은 데이터에 접근할 수 있게 하는 격리 수준이다. 일반적으로 잘 사용하지 않는데 그 이유는 커밋되지 않는 데이터에 타 트랜잭션이 접근하게 되면서 데이터의 부정합을 유발할 수 있기 때문이다.</li>
<li><b>Committed&nbsp;Read&nbsp;(커밋된 읽기)</b> Committed Read는 위와 반대로 commit된 데이터에만 접근할 수 있게하는 격리 수준이다. 때문에 dirty read 현상은 발생하지 않는다. 하지만 해당 수준에서도 비반복 읽기(Non-Repeatable-Read)라는 문제가 발생하는데 이는 한 트랜잭션 내에서 같은 쿼리를 실행시켰을 때, 다른 결과가 나오는 것을 의미한다.</li>
<li><b>Repeatable Read (반복 가능한 읽기)</b> Repeatable Read는 이러한 현상을 방지하기 위한 격리수준으로 각각의 트랜잭션에 순차 증가하는 고유한 번호를 할당하며, 자신의 번호보다 낮은 트랜잭션에서 커밋된 데이터만 읽을 수 있지만 특정 데이터를 반복 조회 시 같은 데이터를 반환한다는 특징이 있다. 즉, 데이터 읽기의 일관성을 보장한다. 하지만 해당 격리 수준에서도 비반복 읽기의 문제 중 하나인 Phantom Read 문제(새로운 데이터가 생기거나 없어지는 문제)가 발생할 수 있다.</li>
<li><b>Serializable<br /></b>마지막으로 Serializable은 가장 높은 트랜잭션 격리 수준으로, 모든 트랜잭션을 순차적으로 처리하는 것을 말한다. 즉, 모든 트랜잭션들을 완전히 격리해서 처리하는 것을 말한다.</li>
</ol>
<p><figure class="imageblock alignCenter"><span><img height="329" src="https://blog.kakaocdn.net/dn/eoOoZW/btsMNpdCVQR/eZjFoXqMl5P05R11IMBbUk/img.png" width="1983" /></span></figure>
</p>
<p>위로 올라갈수록 속도는 빨라지지만 데이터의 일관성을 보장하지 못하고 밑으로 갈수록 트랜잭션을 순차적으로 처리하므로 속도는 느리지만 데이터의 일관성을 보장할 수 있다는 장단점이 있다.</p>
<hr contenteditable="false" />
<p>영속성 엔티티는 1차 캐시로 반복 가능한 읽기(Repeatable Read) 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공하여 동일성을 보장한다.</p>
<h1>섹션 4. 엔티티 매핑</h1>
<h2>기본 키 매핑</h2>
<h3>직접 할당</h3>
<p>@Id 어노테이션만 사용하고 @GeneratedValue 어노테이션을 사용하지 않으면 사용자가 직접 기본 키 값을 설정할 수 있다.</p>
<h3>IDENTITY</h3>
<p>기본 키 생성을 데이터베이스에 위임하는 방식이다. 즉, 해당 전략은 사용자가 직접 ID 값을 넣을 수 없고 Insert 쿼리 문에 Id 값이 null 값으로 날라가면 그때 DB에서 자동으로 값을 설정하는 방식이다. 하지만 이 방식의 문제점은 persist를 했을 때 Id 값이 설정되지 않았다는 점이다. 우리는 지금까지 persist했을 때 바로 DB에 반영되지 않는다고 배웠다. 하지만 영속성 컨텍스트로 데이터를 관리하기 위해서는 기본키(pk)값이 반드시 필요하다.</p>
<p><figure class="imageblock floatLeft"><span><img height="253" src="https://blog.kakaocdn.net/dn/cotHWt/btsML3v5Kcf/ODjH1ADRQrkzjDuZ8mRYj0/img.png" width="365" /></span></figure>
</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>IDENTITY 전략은 영속화를 했을 때, 기본키 값이 null이여서 영속성 컨텍스트로 데이터를 관리하지 못하는 문제를 해결하기 위해 em.persist 시점에 즉시 Insert 쿼리문을 실행하고 DB에서 식별자를 조회한다.</p>
<p>정리하면 JPA는 기본적으로 트랜잭션 커밋 시점에 Insert 쿼리문을 실행하지만 IDENTITY 전략에서는 예외적으로 persist 시점에 즉시 Insert 쿼리문을 실행한다.</p>
<h3>&nbsp;</h3>
<h3>SEQUENCE</h3>
<p>SEQUENCE전략은 DB의 Object인 시퀀스(유일한 값을 순서대로 생성)를 사용해서 ID를 넣어주는 방법이다.</p>
<pre class="less"><code>@Entity
@SequenceGenerator(
	name = "MEMBER_SEQ_GENERATOR",
	sequenceName = "MEMBER_SEQ",
	initialValue = 1,
	allocationSize = 50
)
public class Member{
	@Id
	@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "MEMBER_SEQ_GENERATOR")
	private Long id;
}
</code></pre>
<p>persist를 했을 때, DB의 Sequence Object에게 가서 Id를 받아서 사용하므로 INSERT SQL을 모아서 날릴 수 있다. 하지만 persist를 할 때마다 항상 DB의 Sequence Object에게 가서 next값을 받아 와야하기 때문에 성능적인 문제가 있다. 따라서, allocationSize를 정하여 DB에서 요청해올 때 한번에 그 사이즈 만큼을 가져와 사용하는 방법을 활용한다. 즉 1번 key값을 저장하려고할 때 1번 key 값 뿐만 아니라 50번 key까지 받아서 또 persist가 발생했을 때 다시 DB로 이동하지 않고 과거에 받아온 key의 다음 값을 사용해준다.</p>
<h3>TABLE</h3>
<p>Table 전략은 키를 생성해주는 테이블을 하나 생성하여 사용하는 전략이다. DB의 시퀀스를 흉내내듯이 테이블이 키값을 생성하게 된다.</p>
<pre class="less"><code>@Entity
@TableGenerator(
	name = "MEMBER_SEQ_GENERATOR",
	table = "MY_SEQUENCES",
	pkColumnValue = "MEMBER_SEQ", allocationSize = 1)
)
public class Member{
	@Id
	@GeneratedValue(strategy = GenerationType.TABLE, generator = "MEMBER_SEQ_GENERATOR")
	private Long id;
}
</code></pre>
<p>모든 데이터베이스에 적용이 가능하다는 장점이 있지만 성능적으로 떨어지기 때문에 잘 사용하지 않는 방법이다.</p>
<h3>AUTO</h3>
<p>Auto 전략은 위 3가지 전략중 DB의 SQL 방언에 맞추어 적절하게 자동으로 적용해주는 방법으로 기본키 매핑 전략의 기본(default) 값이다.</p>