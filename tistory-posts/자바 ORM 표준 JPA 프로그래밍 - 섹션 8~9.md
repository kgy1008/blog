<p>&nbsp;</p>
<h1>섹션 8. 프록시와 연관관계 관리</h1>
<h2>프록시 (Proxy)</h2>
<h3>프록시 객체 (Proxy Object)</h3>
<blockquote>  프록시 객체는 엔티티의 실제 데이터를 데이터베이스에서 가져오는 시점을 지연시키기 위해 원본(타겟) 객체를 대신해서 호출될 가짜 객체이다.</blockquote>
<p><figure class="imageblock alignCenter"><span><img height="543" src="https://blog.kakaocdn.net/dn/Phygh/btsMNdqZA87/PY5OtdtsPa91FmqQ7v3kS0/img.png" width="2000" /></span></figure>
</p>
<p>프록시 객체는 실제 클래스(엔티티)를 상속 받아서 만들어지며 클라이언트 코드와 실제 데이터베이스에서 로드된 엔티티 객체(타겟 객체) 사이에 위치한다. 때문에 클라이언트는 실제 엔티티 객체에 직접 접근하지 않고, 프록시 객체를 통해 간접적으로 접근하게 된다. 즉, 프록시 객체는 실제 객체의 참조(target)값을 보관하며 타겟 클래스와 겉모양이 같은 껍데기일 뿐이다. 관계를 쉽게 비유하자면 타겟 객체를 집 주인이라고 생각했을 때, 프록시 객체는 집 주인을 대신해서 계약을 요청받는 중개인으로 생각하면 된다.</p>
<h3>getReference()</h3>
<p><figure class="imageblock alignCenter"><span><img height="1000" src="https://blog.kakaocdn.net/dn/G2rfl/btsMM5s85X6/mEyNp8AYiJLknvlvZs5hc1/img.png" width="1644" /></span></figure>
</p>
<p>em.getReference()는 데이터베이스 조회를 미루는 가짜(프록시) 엔티티 객체를 조회하는 메서드이다.</p>
<p>em.getReference()호출 시점에는 실제 엔티티가 로드되는 것이 아니라 프록시 객체가 생성되기 때문에 쿼리가 날라가지 않는다. 이때, 프록시 객체가 초기화가 되기 전까지는 프록시 객체의 참조(target)값은 null값으로 설정된다. 사용자(클라이언트)가 해당 엔티티에 접근하여 메서드를 호출하면 초기화가 이루어진다. 이때 프록시 객체의 초기화는 처음 사용될 때 딱 한 번만 이루어지며 프록시 객체를 통해서 실제 엔티티에 접근이 가능해진다. (이때 실제 엔티티를 생성하기 위해 DB에 접근하면서 쿼리문이 나가게 된다.)</p>
<p>하지만 만약 getName()이 아닌 기본키 값인 getId() 메서드를 실행한다면 select 쿼리문은 발생하지 않는다. 즉, 식별자를 조회할 때는 프록시를 초기화하지 않는다. 더 자세한 내용은 아래 링크를 참고하자.</p>
<p><a href="https://tecoble.techcourse.co.kr/post/2022-10-17-jpa-hibernate-proxy/">https://tecoble.techcourse.co.kr/post/2022-10-17-jpa-hibernate-proxy/</a></p>
<h3>영속성 컨텍스트와 프록시 객체</h3>
<p>아래 예시 코드를 살펴보자.</p>
<pre class="reasonml"><code>Member m1 = em.find(Member.class, member1.getId()) 
Member m2 = em.getReference(Member.class, member1.getId())

System.out.println("m1 == m2" + (m1.getClass() == m2.getClass()));
</code></pre>
<p>위의 코드를 살펴보면 우리는 m1은 실제 Member 엔티티를 m2는 프록시 객체를 담고있을 것이라고 기대한다. 하지만, 정말 실제로 저렇게 동작을 한다면 m1과 m2는 pk값이 동일한 객체들끼리의 == 비교를 했을 때, false가 나오게 된다. 그럼 실행 결과는 어떻게 될까? 놀랍게도 true이다. 이는 m2가 우리의 em.getReference() 를 호출했을 때 프록시 객체가 반환할 것이라는 우리의 기대와 달리 실제 엔티티를 반환했기 때문이다. 그 이유가 무엇일까? 먼저, 이미 영속성 컨텍스트의 1차 캐시에 해당 엔티티가 존재하면 proxy 객체로 반환할 이점이 전혀 없다. 하지만 그 이유보다 중요한 것은 <b>같은 영속성 컨텍스트(트랜잭션 레벨)안에서 pk(기본키 값)가 동일한 객체들끼리의 == 비교는 반드시 같아야 한다.</b> 때문에 위에서 em.find()를 통해 실제 엔티티를 반환했고 == 비교가 true가 나오기 위해서는 반드시 같은 참조값을 가져야 하므로 m2는 em.getReference()를 호출했음에도 프록시 객체가 아닌 실제 엔티티를 반환하는 것이다. 그 반대도 성립한다.</p>
<pre class="reasonml"><code>Member m1 = em.getReference(Member.class, member1.getId())
Member m2 = em.find(Member.class, member1.getId())

System.out.println("m1 == m2" + (m1.getClass() == m2.getClass()));
</code></pre>
<p>이번에는 em.find()로 실제 엔티티를 먼저 반환하는 대신, em.getReference()를 통해 프록시 객체를 먼저 반환하는 코드이다. 이 경우에도 같은 영속성 컨텍스트(트랜잭션 레벨)안에서 pk(기본키 값)가 동일한 객체들끼리의 == 비교는 반드시 같아야 한다라는 원칙은 반드시 적용되어야 하기 때문에 em.find()를 호출했음에도 프록시 객체가 반환이 되게 된다.</p>
<hr contenteditable="false" />
<p>만약 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때, 아래와 같이 프록시를 초기화하면 어떻게 될까?</p>
<pre class="reasonml"><code>Member refMember = em.getReference(Member.class, member1.getId());
em.detach() // 준영속 상태로 바꿈
refMember.getName() // 초기화
</code></pre>
<p>위의 코드는 refMember 객체를 준영속 상태를 바꾼 후에 메소드를 호출함으로써 프록시 객체의 초기화를 시도하고 있다. 하지만 이 경우에는 org.hibernate.LazyInitializationException라는 예외가 발생한다. 해당 예외가 발생하는 이유는 프록시 객체가 초기화를 하기 위해서는 결국 영속성 컨텍스트에 접근하여 실제 엔티티를 불러와야 하는데 detach를 통해 영속성 컨텍스트의 관리에서 벗어났기 때문에 프록시를 초기화 할 수 없다는 오류가 발생하는 것이다.</p>
<h2>즉시로딩과 지연로딩</h2>
<h3>즉시로딩 (Eager Loading)</h3>
<p><b>즉시 로딩</b>이란 말 그대로 데이터를 조회할 때, 연관된 모든 객체의 데이터까지 한 번에 불러오는 방식이다. 만약 Post 엔티티와 Blog 엔티티가 FetchType.EAGER 즉, 즉시 로딩으로 설정되어 있다면 어떻게 될까? postRepository.findById(postId) 메서드를 통해 Post 엔티티를 조회할 때 즉시 Post 데이터와 연관된 Blog 데이터가 함께 로드된다. 즉, 프록시 객체의 생성 없이 별도의 쿼리를 통해 바로 Blog 엔티티에 접근하는 것이다. 이처럼 즉시 로딩 방식은 연관된 데이터를 함께 불러오는 작업일 때, 빠르게 처리를 할 수 있다는 장점이 있다. 하지만 실무에서는 예상하지 못한 SQL문이 발생하거나 Join 쿼리가 한번에 너무 많이 나가게되는 등 여러가지 이유로 인해 가급적 사용하지 않는다고 한다. 연관관계 중 @ManyToOne, @OneToOne의 기본 설정은 즉시 로딩이라고 한다. 때문에 해당 연관관계에서는 @OneToMany(fetch = FetchType.EAGER)과 같이 설정하여 지연로딩으로 바꿔주어야 한다.</p>
<h3>지연로딩 (Lazy Loading)</h3>
<p><b>지연 로딩</b>은 엔티티가 로드될 때, 연관된 엔티티를 즉시 로드하지 않고 필요한 시점에 연관된 객체의 데이터를 로드하는 방식이다. 지연 로딩 방식에서는 연관된 엔티티 데이터는 실제로 접근할 때까지 로드되지 않는다. 즉, 실제 엔티티를 로드하는 대신 프록시 객체를 로드하고 있는 것이다. 클라이언트 코드가 객체의 메서드를 호출해야 비로소 프록시 객체는 초기화하기 위해 그 순간 데이터베이스에 접근하여 실제 데이터를 로드하게 된다.</p>
<h2>영속성 전이(CASCADE)와 고아 객체</h2>
<h3>영속성 전이(CASCADE)</h3>
<p>영속성 전이는 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때 사용한다. 영속성 전이는 연관관계를 매핑하는 것과 아무런 관련이 없다. 단지 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 편리함만을 제공할 뿐이다. CASCADE는 두 엔티티의 생명주기가 일치하고 부모 - 자식 관계처럼 소유자가 단 한개 뿐일 때 사용하면 편리하다.</p>
<p><a href="https://tecoble.techcourse.co.kr/post/2023-08-14-JPA-Cascade/">https://tecoble.techcourse.co.kr/post/2023-08-14-JPA-Cascade/</a></p>
<h3>고아 객체</h3>
<p>고아 객체란&nbsp;부모 엔티티와 연관관계가 끊어진 자식 엔티티를 의미하며 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제하는 것을 고아 객체 제거라고 한다.</p>
<pre class="lasso"><code>Parent parent = entityManager.find(Parent.class, id);
parent.getChildren().remove(0);
</code></pre>
<p>orphanRemoval 설정을 true로 한 후, Parent의 Child 중 0번째 인덱스에 존재하는 Child를 제거하고 트랜잭션 commit을 하면 아래와 같이 자동으로 연관관계가 끊어진 객체를 삭제하는 delete 쿼리문이 나간다.</p>
<pre class="sql"><code>// orphanRemoval 이 동작함으로 고아가 된 자식 하나에 대해 Delete Query가 발생한다.
DELETE FROM CHILD WHERE ID = ?
</code></pre>
<p>고아 객체 제거는 참조하는 곳이 영속성 전이와 마찬가지로 하나일 때 사용하야 하며 연관관계가 @OneToOne, @OneToMany일 경우에만 사용 가능하다.</p>
<h1>섹션 9. 값 타입</h1>
<h2>임베디드 타입</h2>
<p>최상위 레벨로 보면 JPA는 데이터 타입을 두 가지로 분류한다. 바로 <b>엔티티 타입</b>과 <b>값 타입</b>이다. 엔티티 타입은 우리가 구현할 때 @Entity 어노테이션을 붙여 정의하는 객체이다. 엔티티 내부의 모든 값들을 바꿔도 식별자만 유지되면&nbsp;추적이 가능하다. 반면 값 타입은 int, Integer, String처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체를 말한다. 식별자가 없고 값만 있으므로 변경시 추적 불가능하다. 임베디드 타입은 값 타입 중 하나이다. 주로 기본 값 타입을 모아서 만들기 때문에 복합 값 타입이라고도 한다.</p>
<p>&nbsp;</p>
<p><figure class="imageblock alignCenter"><span><img height="944" src="https://blog.kakaocdn.net/dn/b3MhMB/btsMNhGSYnf/tfErQMIJ4ILkjj0WUA0Tj0/img.png" width="1520" /></span></figure>
</p>
<p>&nbsp;</p>
<p>여기서 사용하는 Period와 Address가 바로 임베디드 타입이다. 임베디드 타입을 통해 연관된 속성들을 한번에 관리할 수 있어 응집도를 높일 수 있으며 다른 객체에서도 사용 할 수 있어 재사용성을 높힐 수 있다. 위와 같이 임베디드 타입을 통해 객체를 분리하더라도 테이블은 하나만 매핑된다. 즉, 임베디드 타입을 사용하든 안하든 DB 테이블 입장에서는 변경되는 것이 없다는 말이다.</p>
<h3>임베디드 타입 구현</h3>
<p>임베디드 타입을 구현할 때는 아래와 같이 구현한다.</p>
<pre class="kotlin"><code>// Member 클래스
@Entity
@Getter @Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_id")
    private Long id;

    private String name;

    @Embedded
    Address address;

    @OneToMany(mappedBy = "member")
    List&lt;Order&gt; orders = new ArrayList&lt;&gt;();
}

// Address 클래스
@Embeddable
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Address {

    private String city;

    private String street;

    private String zipcode;

    public Address(String city, String street, String zipcode) {
        this.city = city;
        this.street = street;
        this.zipcode = zipcode;
    }
}
</code></pre>
<p>위의 코드에서 볼 수 있듯이 임베디드 타입임을 정의하는 Address 클래스 위에는 @Embaddable 어노테이션을 붙인다. 그리고 이 임베디드 타입을 사용하는 Member 클래스의 필드에는 @embadded 어노테이션을 붙인다. 둘 중 하나만 사용해도 정상적으로 작동하지만 모든 클래스에서 해당 타입이 내장 타입인 것을 가시적으로 확인하기 위해 두개의 어노테이션 모두 쓰는 것을 권장한다. 추가적으로 임베디드 타입의 값을 null로 설정하면(Address address = null) 매핑한 컬럼 값들 또한 모두 null로 설정된다.</p>
<h3>@AttributeOverrides</h3>
<p>만약 한 엔티티에서 같은 값 타입을 사용하고 싶다면 어떻게 해야할까?</p>
<pre class="less"><code>@Entity
public class Member {
  
  @Id @GeneratedValue
  private Long id;
  private String name;
  
  @Embedded
  Address homeAddress;
  
  @Embedded
  Address companyAddress;
}
</code></pre>
<p>위와 같이 코드를 작성하면 테이블에 매핑하는 컬럼명이 중복된다는 문제점이 발생한다. 때문에 이런 상황에서는 아래와 같이 @AttributeOverrides를 사용해서 컬럼명 속성을 재정의해야한다.</p>
<pre class="less"><code>@Entity
public class Member {
  
  @Id @GeneratedValue
  private Long id;
  private String name;
  
  @Embedded
  Address homeAddress;
  
  @Embedded
  @AttributeOverrides({
    @AttributeOverride(name="city", column=@Column(name="COMPANY_CITY")),
    @AttributeOverride(name="street", column=@Column(name="COMPANY_STREET")),
    @AttributeOverride(name="zipcode", column=@Column(name="COMPANY_ZIPCODE"))
  })
  Address companyAddress;
}
</code></pre>
<h2>값 타입과 불변 객체</h2>
<h3>객체 타입의 한계</h3>
<p>앞서 설명한 임베디드 타입처럼 직접 정의한 값 타입은 객체 타입이다. 따라서 임베디드 타입을 여러 엔티티에서 공유하게 된다면 심각한 부작용(side effect)이 발생할 수 있다. 아래 코드를 살펴보자.</p>
<pre class="reasonml"><code>Address address = new Address("city", "street", 1000)

Member member = new Member();
member.setAddress(address);

Member member2 = new Member();
member2.setAddress(address);

member.getHomeAddress().setCity("newCity");
</code></pre>
<p>코드 작성자의 의도는 첫번째 Member 객체의 city 값만을 바꾸는 것을 기대하고 코드를 작성했을 것이다. 하지만 Address와 같이 임베디드 타입은 객체 타입이기 때문에 공유 참조 문제가 발생한다. 즉, member2의 city값도 모두 newCity로 변경되는 것이다. 이처럼 객체 타입은 참조 값을 직접 대입하는 것을 막을 방법이 없다. 때문에 객체 타입을 수정할 수 없게 만들어 부작용을 원천 차단해야한다. 즉, <b>값 타입을 불변 객체로 설계</b>하여 생성 시점 이후에는 절대 값을 변경할 수 없도록 만들어야 한다. 불변 객체로 만드는 방법은 간단하다. 생성자로만 값을 설정하도록 하고 수정자(setter)를 만들지 않아 그 값을 수정할 수 없도록 설계하면 된다.</p>
<h3>값 타입의 비교</h3>
<p>값 타입을 비교하는 데는 2가지 관점이 있다. 먼저 첫번째는 <b>동일성(Identity) 비교</b>이다. 동일성 비교는 인스턴스의 참조 값을 비교하며 == 을 사용하여 비교한다. 두번째는 <b>동등성(equivalenve) 비교</b>이다. 동등성 비교는 인스턴스의 값을 비교하며 equals()를 사용하여 비교한다. 본래 equals() 메서드는 Object의 번지를 비교하는 메소드로, 기본적으로는 동일성 비교 수행한다. 때문에 동등성 비교를 수행하기 위해서는 값 타입의 equals() 메서드를 적절하게 재정의하는 과정이 반드시 필요하다. 대표적으로 String 클래스는 equals() 메서드를 오버라이딩 해서 문자열을 비교하도록 하였다. 즉, 쉽게 정리하면 <b>동일성</b>은 <b>물리적으로 같은 메모리에 있는 객체 인스턴스인지 참조값을 확인</b>하는 것이고 <b>동등성</b>은 <b>논리적으로 같은지 확인</b>하는 것이다.</p>
<blockquote>
<p>equals()와 hashcode(), 두 메소드 모두 객체의 동등성을 검사하기 위한 것이다. 해시 자료구조를 사용하고 두 메소드 중 하나를 재정의한다면 나머지 하나도 반드시 재정의해줘야 한다.</p>
</blockquote>
<h2>값 타입 컬렉션</h2>
<h3>값 타입 컬렉션</h3>
<p><figure class="imageblock floatLeft"><span><img height="145" src="https://blog.kakaocdn.net/dn/ufijZ/btsMMg27zCT/VAupnwF7nTe4895HXmOAH0/img.png" width="324" /></span></figure>
</p>
<p>&nbsp;</p>
<p>여러 개의 값 타입을 컬렉션 형태로 저장하고 싶으면 어떻게 처리할까? DB가 이를 처리할 수 있을까? 유감스럽게도 그렇지 않다. DB가 해당 컬렉션을 매핑해서 같은 테이블에 저장할 수 있는 방법은 존재하지 않는다. 때문에 컬렉션을 DB에 반영하기 위해서는 해당 값을 저장하는 새로운 테이블을 만들어야 한다.</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p><figure class="imageblock alignCenter"><span><img height="387" src="https://blog.kakaocdn.net/dn/b6hCwB/btsMMB6ZZbt/leiVnxj27hRvCzKLb4Yh3k/img.png" width="763" /></span></figure>
</p>
<p>이처럼 값 타입을 하나 이상 저장할 때에는 아래 코드와 같이 @ElementCollection 과 @CollectionTable 어노테이션을 사용해서 컬렉션을 저장하기 위한 별도의 테이블을 자동으로 만들 수 있다.</p>
<pre class="less"><code>@ElementCollection
@CollectionTable(name = "FAVORITE_FOOD",
        joinColumns = @JoinColumn(name = "MEMBER_ID")
)
@Column(name = "FOOD_NAME") // 컬럼이 하나고 직접 정의한 것이 아니기 때문에 테이블 생성할 때 칼럼 이름을 설정할 수 있다.
private Set&lt;String&gt; favoriteFoods = new HashSet&lt;&gt;();

@ElementCollection
@CollectionTable(name = "ADDRESS",
        joinColumns = @JoinColumn(name = "MEMBER_ID")
)
private List&lt;Address&gt; addressesHistory = new ArrayList&lt;&gt;()
</code></pre>
<p>위의 코드를 실행하면 아래와 같이 테이블이 자동으로 만들어진다.</p>
<p><figure class="imageblock alignCenter"><span><img height="658" src="https://blog.kakaocdn.net/dn/cQDtBV/btsMMG8dyD4/QhcEXtcvF0yBlHhVfuGOb1/img.png" width="633" /></span></figure>
</p>
<p>&nbsp;</p>
<p>이렇게 만들어진 별도의 테이블(FAVORITE_FOOD, ADDRESS)은 본인 스스로의 생명주기가 존재하지 않고 Member 테이블에 의존하게 된다. 이 또한 코드로 살펴보자!</p>
<pre class="reasonml"><code>Member member = new Member();
member.setUsername("member1");
member.setHomeAddress(new Address("homeCity", "street", "10000"));

member.getFavoriteFoods().add("치킨");
member.getFavoriteFoods().add("족발");
member.getFavoriteFoods().add("피자");

member.getAddressHistory().add(new Address("old1", "street1", "10001"));
member.getAddressHistory().add(new Address("old2", "street2", "10002"));

em.persist(member);

tx.commit();
</code></pre>
<p>값 타입 컬렉션을 매핑하는 테이블은 자신의 생명주기를 가지지 않고 Member 테이블의 생명주기에 의존한다. 때문에 em.persist(member)를 호출하고 트랜잭션이 커밋되면 favorite_food와 address값 모두 즉시 insert되는 것을 확인할 수 있다. 즉, 값 타입 컬렉션은 영속성 전이(Cascade)와 고아 객체 제거 기능을 필수로 가진 것과 비슷하다고 할 수 있다. 추가적으로 값 타입 컬렉션을 매핑하는 테이블은 지연 로딩 전략을 사용하기 때문에 em.find(member)를 호출 했을 때 바로 값들이 로드되지 않는다.</p>
<h3>값 타입 컬렉션의 제약사항</h3>
<p>만약 값 타입 컬렉션의 값을 수정하기 위해서는 어떻게 해야할까? 아래 코드와 같이 값을 직접 삭제한 후, 새로운 값을 다시 넣어주어야 한다.</p>
<pre class="less"><code>member.getFavoriteFoods().remove("치킨");
member.getFavoriteFoods().add("피자");

member.getAddressHistory().remove(new Address("oldCity", "street", "10000"));
member.getAddressHistory().add(new Address("newCity", "street", "10000"));
</code></pre>
<p>위의 코드를 보았을 때 우리는 첫번째 요소만 삭제하는 delete 쿼리가 발생하고 새로운 값을 추가하는 1개의 insert 쿼리문이 나갈 것으로 기대한다. 하지만 값 타입 컬렉션 안의 데이터를 수정할 때는 일부만 수정하는 것이 아니라 주인 엔티티와 연관된 <b>모든 데이터를 삭제하고 값 타입 컬렉션에 있는 현재 값을 모두 다시 저장</b>하는 방식으로 동작한다. 때문에 총 2개의 insert 쿼리문이 발생하게 된다. 이처럼 값 타입 컬렉션은 성능상의 문제가 존재하며 엔티티와 다르게 식별자 개념이 존재하기 때문에 값을 변경했을 때 추적이 어렵다. 이러한 제약사항 때문에 값 타입 컬렉션을 사용하는 대신, 일대다 관계를 고려하여 새로운 엔티티를 만들어 해당 엔티티에 값 타입을 사용하는 방식을 권장한다.</p>