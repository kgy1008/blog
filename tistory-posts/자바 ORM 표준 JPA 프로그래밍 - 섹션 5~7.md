<h1>섹션 5, 6. 연관관계 매핑</h1>
<h2>연관관계 주인</h2>
<blockquote>⚠️ 외래 키가 있는 곳을 연관관계의 주인으로 정해라</blockquote>
<p><b>연관관계의 주인</b>이란, <b>양방향 매핑에서</b> 두 객체 중 <b>외래 키</b>를 누가 <b>관리하는 객체</b>를 말한다. 주인이 아닌 객체는 읽기만 가능하다. 즉, 연관관계의 주인은 단순히 외래 키를 누가 관리하느냐의 문제이기 때문에 비즈니스 상 우위에 있다고 주인으로 설정해서는 안된다. 이때, 일대다 관계에서 <b>외래키는 항상 다</b>(<b>多</b>)<b>쪽에 위치하도록 설계</b>해야 한다.</p>
<p><figure class="imageblock alignCenter"><span><img height="999" src="https://blog.kakaocdn.net/dn/qsGkc/btsMNQaVF58/deMQ3pc0Np59w0opSMvLEK/img.png" width="1911" /></span></figure>
</p>
<p>만약, 위와 같은 연관관계가 존재한다고 가정해보자. 이때, Member(<b>多</b>)가 아닌 Team이 연관관계의 주인이 된다면 어떨까? 해당 팀에 소속된 member에 변경이 생기게 된다면 본인의 테이블인 Team이 아닌 다른 테이블 즉, Member 테이블에 Update 쿼리가 나가게 된다. 즉, Team 객체에 행위를 하였는데 다른 객체인 Member의 상태가 변하는 것이다. 때문에 <b>연관관계의 주인을 외래키가 존재하는 엔티티</b>로 설정하는 것을 권장한다.</p>
<hr contenteditable="false" />
<blockquote><b>일대일 관계에서의 연관관계 주인은 어떻게 설정할까?</b><br /><b>자주 접근하는 주 테이블을 연관관계의 주인으로 설정</b>하면, <b>JPA 매핑이 편리</b>하다는 장점이 있다. 때문에 객체지향 개발자들이 선호하는 방식이지만 매핑된 대상 테이블의 값이 없으면 외래 키에 NULL을 허용해야 한다는 단점이 존재한다. <br />반면, <b>대상 테이블을 연관관계의 주인으로 설정</b>하는 방식은 전통적인 DBA분들이 선호하는 방식이다. <b>NULL 값을 허용해야 한다는 문제점도 없고</b> 비즈니스 적으로 관계가 일대다 관계로 변경되는 상황이 발생했을 때, <b>테이블 구조를 유지하면서 수정사항을 반영할 수 있다.</b> 다만 코드 상으로 봤을 때, JPA에서 대상 테이블에 외래키 단방향 매핑을 지원하지 않아 양방향 매핑을 해야한다는 점과 JPA가 제공하는 기본 프록시 기능의 한계로 지연 로딩으로 설정해도 항상 즉시 로딩으로 설정된다는 단점이 있다.</blockquote>
<h3>mappedBy</h3>
<p>mappedBy 를 직역하면 &lsquo;~에 의해 매핑된&rsquo;이다. 말 그대로 mappedBy 조건이 붙은 속성은 연관관계에 있어 상대 엔티티의 속성에 의해 매핑이 되어버린 속성을 의미한다. 아래 코드를 살펴보자.</p>
<pre class="less"><code>@Entity
public class Member {
    @Id @GenerateValue
    private Long id;

    @Column(name = "username")
    private String name;

    private int age;

    @ManyToOne
    @joinColumn(name = "team_Id") // 여기서 name은 매핑할 외래 키의 이름을 지정하는 것 -&gt; 디폴트 값은 연관된 대상의 기본키 칼럼명이다.
    private Team team;
}

@Entity
public class Team {
    @Id @GenerateValue
    private Long id;

    @Column(name = "username")
    private String name;

    @OneToMany(mappedBy = "team")
    private List&lt;Member&gt; members = new ArrayList&lt;Member&gt;();
}</code></pre>
<p>위의 코드에서도 알 수 있듯 Member와 Team은 서로 다대일 관계이다. 때문에 연관관계의 주인은 Member엔티티가 되고 <b>Member 객체만이 등록하거나 수정</b>할 수 있다. 반면 <b>Team의 경우 읽기만이 가능</b>하다. 이 &lsquo;읽기만 가능하다&rsquo; 라는 제약 조건을 코드에 적용하기 위해서 사용하는 조건이 바로 mappedBy인 것이다. mappedBy를 사용함으로써 members라는 필드는 Member 객체의 team에 의해 매핑이 되었다! 라고 명시하는 것이다. 추가적으로 mappedBy 속성을 붙인 필드는 해당 엔티티의 컬럼으로 만들어지지 않는다. 즉, 테이블에는 외래 키 컬럼이 생성되지 않는다.</p>
<h3>양방향 매핑 시 주의점</h3>
<p>양방향 매핑 시 주의해야할 점은 연관관계 주인이 아닌 엔티티의 필드에 접근하여 값을 수정하거나 추가할 경우, 반영되지 않는다는 점이다. 아래 코드를 살펴보자.</p>
<pre class="abnf"><code>Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("member1");

team.getMembers().add(member);
em.persist(member);</code></pre>
<p>Team은 아까 전에도 언급했듯이 Member와의 관계에 있어 연관관계의 주인이 아니다. 하지만 연관관계의 주인이 아닌 Team에 접근하여 member를 추가하면 어떻게 될까? 정답은 아무 일도 일어나지 않는다. 즉, Member 테이블에서 team_Id값이 null값으로 수정된다. 이는 add함수를 통해 memberList에 member를 추가해주는 코드를 작성했음에도 반영되지 않았다는 것을 의미한다. 만약, 우리의 의도대로 정상적으로 반영하고 싶으면 어떻게 해야할까? 연관관계의 주인인 Member 엔티티에 접근하여 값을 추가하거나 수정해야한다.</p>
<pre class="reasonml"><code>Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("member1");
// team.getMembers().add(member);
// 연관관계의 주인에 값 설정
member.setTeam(team);
em.persist(member);</code></pre>
<p>위와 같이 연관관계의 주인인 Member 객체에 접근하여 값을 설정해주어야 한다. 이처럼 연관관계의 주인에 접근하여 처리하는 코드인 member.setTeam(team) 만 작성해도 우리의 의도대로 정상 동작하지만 일반적으로 team.getMembers().add(member) 코드도 함께 작성해주는 것을 권장한다. 그 이유가 무엇일까?</p>
<pre class="reasonml"><code>Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("member1");
member.setTeam(team);
em.persist(member);

// em.flush();
// em.clear(); 

Team findTeam = em.find(Team.class, team.getId());
List&lt;Member&gt; members = findTeam.getMembers();  // 빈 리스트 반환</code></pre>
<p>em.flush(), em.clear() 를 호출하여 영속성 엔티티의 값들을 초기화시켜주면 상관이 없지만 만약 호출하지 않았다고 가정해보자. 이 경우에는 persist()만을 호출하였고 아직 트랜잭션이 커밋되지 않았기 때문에 1차 캐시에만 저장되어 있을 뿐 DB에는 아직 반영이 되지 않은 상태일 것이다. 이 경우 team의 Id값을 통해 Team 객체를 찾아 해당 팀에 속해있는 member를 조회한다면, DB에 접근하지 않고 1차 캐시에서 값을 읽어올 것이다. 여기서 주의해야 할 점은 1차 캐시에는 순수 객체 상태만이 저장되어 있다는 것이다. 때문에 members라는 리스트는 빈 리스트 값을 반환하게 된다. 즉, team.getMembers().add(member) 라는 코드를 따로 작성하지 않은 상태에서 영속성 컨텍스트를 flush하지 않았기 때문에 우리의 의도대로 동작하지 않았다.</p>
<p>때문에 이러한 상황을 고려해서 <b>항상 양쪽 모두 값을 설정</b>하는 것을 권장한다.</p>
<h3>연관관계 편의 메서드</h3>
<blockquote>☝  연관관계 편의 메서드는 한 번에 양방향 관계를 설정하는 메서드이다.</blockquote>
<p>아까 언급했듯이, 양방향 연관관계일 때는 연관관계의 주인 여부와 상관없이 모두 값을 설정하는 것을 권장한다고 언급했었다. 이때, 연관관계 메서드를 따로 설정해주지 않으면 전의 코드처럼 하나하나 수작업으로 설정해주어야 한다. 이는 번거러울 뿐더러 실수로 한 줄을 넣지 않게 되면 둘 중 하나만 호출이 되어 양방향이 깨질 수 있다. 이를 방지하고자 연관관계 편의 메서드를 생성하는 것이 좋다. 때문에 아래와 같이 연관 관계 편의 메서드를 작성하여 하나인 것처럼 사용하는 것이 안전하다.</p>
<pre class="cs"><code>// 연관관계 편의 메서드
public void changeTeam(Team team) {
    this.team = team;
    team.getMembers().add(this);
}</code></pre>
<h1>섹션 7. 고급 매핑</h1>
<h2>상속관계 맵핑</h2>
<h3>조인 전략</h3>
<p><figure class="imageblock alignCenter"><span><img height="258" src="https://blog.kakaocdn.net/dn/CF7ju/btsMMJXVUqs/QwvKJ66ffXzuhB1jTf3wl0/img.png" width="809" /></span></figure>
</p>
<pre class="css"><code>@Inheritance(strategy=InheritanceType.JOINED)</code></pre>
<p>@DiscriminatorColumn 을 선언해야만 DTYPE 칼럼이 생성되며 필수는 아니지만 만드는 것을 권장한다. 상위 테이블의 기본키가 하위 테이블의 외래키이자 기본키가 된다. DTYPE에 따라 필요한 하위 테이블과 조인하여 사용한다. 가장 일반적으로 많이 사용되는 방법이자 가장 정규화된 방법이다. 외래 키 참조 무결성 제약 조건도 활용이 가능하다. 다만 조회 시 조인을 많이 사용하여 조회 쿼리가 복잡하고 데이터를 저장할 때, Insert 쿼리문이 2번 호출된다는 단점이 존재한다.</p>
<h3>단일 테이블 전략</h3>
<p><figure class="imageblock alignCenter"><span><img height="271" src="https://blog.kakaocdn.net/dn/cX3OIm/btsMMd6iQKo/TmpeeCV45RqVQzQav7V8BK/img.png" width="716" /></span></figure>
</p>
<pre class="css"><code>@Inheritance(strategy=InheritanceType.SINGLE_TABLE)</code></pre>
<p>하나의 상위 테이블(ITEM)에 모두 저장하고 이를 DTYPE으로 구분하는 전략이다. 하나의 테이블에 모두 저장하기 때문에 DTYPE 없이는 구분을 할 수 없기 때문에 @DiscriminatorColumn 을 선언하지 않아도 DTYPE 칼럼이 자동 생성된다. 해당 방식은 조회 시에 조인 쿼리가 필요하지 않으므로 성능이 빠르다는 장점이 있다. 하지만 자식 엔티티가 매핑한 컬럼은 모두 null 값을 허용(nullable)하기 때문에 데이터 무결성 입장에서는 단점이 존재한다. &rarr; 특정 자식 데이터가 들어왔을 때, 다른 자식 테이블의 속성 값은 비어 있기 때문에 null 값을 허용해주어야 한다.</p>
<pre class="scala"><code>// 부모 클래스
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn  // 부모 클래스에 선언 -&gt; 하위 클래스를 구분하는 용도(default = DTYPE)
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;
    private int price;
    private int stockQuantity;

    @ManyToMany(mappedBy = "items")
    private List&lt;Category&gt; categories = new ArrayList&lt;&gt;();

}

// 자식 클래스
@Entity
public class Album extends Item {
    private String artist;
    private String etc;
}</code></pre>
<h3>구현 클래스마다 테이블 전략 (비추천!)</h3>
<p><figure class="imageblock alignCenter"><span><img height="215" src="https://blog.kakaocdn.net/dn/b6CPEY/btsMLwyMupF/rb2cuDDH96LBhQKUgFP8VK/img.png" width="816" /></span></figure>
</p>
<pre class="css"><code>@Inheritance(strategy=InheritanceType.TABLE_PER_CLASS)</code></pre>
<p>슈퍼타입 칼럼들을 서브타입으로 내리는 전략이다. 상위 클래스를 추상 클래스(abstract)로 선언하면 테이블로 만들어지지 않는다. 하지만 이 방법은 추천하지 않는 방법이다. Item_ID로 Item을 찾는다고 생각을 해보자. 이때, 해당 ID값에 맞는 데이터를 찾기 위해 JPA는 UNION ALL 쿼리를 내보낸다. 즉 여러 자식 테이블을 함께 조회하는 것이다. 때문에 해당 방식은 성능이 느리며 추천하지 않는다.</p>
<h2>@MappedSuperClass</h2>
<p><code>@MappedSuperClass</code> 어노테이션은 공통 매핑 정보가 필요할 때 사용한다.</p>
<p><figure class="imageblock alignCenter"><span><img height="1000" src="https://blog.kakaocdn.net/dn/SpQTe/btsMLNmJ6fg/WUDouqa1uxWd6ctLlToK90/img.png" width="1982" /></span></figure>
</p>
<p>즉, DB는 완전히 별개이지만 공통적인 속상만 뽑아서 다른 테이블에 저장하고 이를 상속받아 사용하고 싶을 때 활용한다. 절대 상속 관계 매핑이 아니라는 점에 유의하자!</p>
<pre class="scala"><code>// MappedSuperClass
@MappedSuperClass
public abstract class BaseEntity {
    private LocalDateTime createdDate;
    private LocalDateTime createdDate;
}

// MappedSuperClass를 상속받는 class
// Memeber 클래스에 BaseEntity의 속성인createdDate와 createdDate이 추가되어 create된다.
@Entity
public class Member extends BaseEntity {
    private String artist;
    private String etc;
}</code></pre>
<p>이렇게 @MappedSuperClass 로 선언된 객체(BaseEntity)는 엔티티가 아니며 테이블로 매핑되지도 않는다. 그저 자신을 상속 받는 자식 클래스에 매핑 정보만을 제공할 뿐이다. 때문에 직접 생성해서 사용할 일이 없으므로 추상 클래스로 선언하는 것을 권장한다.</p>
<p>&nbsp;</p>
<p>&nbsp;</p>