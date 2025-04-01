<h2>사용자 정의 레포지토리</h2>
<h3>필요한 이유</h3>
<p>개발을 하다보면 기본적인 네이밍 메서드, @Query 어노테이션 안에 간단한 쿼리문으로 구현할 수 없는 복잡한 동적 쿼리(Querydsl 사용) 등이 필요한 상황이 존재한다. 하지만 Spring data JPA가 제공하는 인터페이스에서는 동적쿼리를 구현할 수 없다. 이때, 사용하는 것이 <b>사용자 정의 레포지토리</b>이다.</p>
<h3>구현 방법</h3>
<pre class="java"><code>// 사용자 정의 인터페이스
public interface MemberRepositoryCustom {
     List&lt;Member&gt; findMemberCustom();
}

// 사용자 정의 인터페이스의 구현체
@RequiredArgsConstructor
public class MemberRepositoryImpl implements MemberRepositoryCustom {
    private final EntityManager em;
    @Override
    public List&lt;Member&gt; findMemberCustom() {
        return em.createQuery("select m from Member m")
                .getResultList();
    } 
}
</code></pre>
<p>위의 코드처럼, 인터페이스를 정의해준 후, 구현체를 만들어주면 된다. 이때, 구현체 이름을 &lsquo;<b>레포지토리 인터페이스 명 + Impl</b>&rsquo; 또는 &lsquo;<b>사용자 정의 인터페이스 명 + Impl</b>&rsquo;으로 네이밍을 해야 스프링 데이터 JPA가 인식해서 스프링 빈으로 등록해줄 수 있다. 즉, 위의 코드에서 현재 작성된 구현제 이름인 MemberRepositoryImpl 대신 MemberRepositoryCustomImpl 으로 작성하여도 정상적으로 동작한다.</p>
<h2>Auditing</h2>
<p>&lt;aside&gt; &lt;img src="/icons/clock-alternate_green.svg" alt="/icons/clock-alternate_green.svg" width="40px" /&gt; Audit,&nbsp;&rsquo;감시하다&rsquo;라는 뜻처럼 Spring Data JPA에는 엔티티가 생성되고, <b>변경되는 그 시점을 감지하여 자동으로 시 생성시각, 수정시각, 생성한 사람, 수정한 사람을 기록할 수 있는 기능</b>이 존재한다.</p>
<p>&lt;/aside&gt;</p>
<h3>순수 JPA 사용</h3>
<pre class="java"><code>@MappedSuperclass
@Getter
public class JpaBaseEntity {

   @Column(updatable = false)
   private LocalDateTime createdDate;
   private LocalDateTime updatedDate;
     
   @PrePersist
   public void prePersist() {
       LocalDateTime now = LocalDateTime.now();
       createdDate = now;
       updatedDate = now;
   }
		 
   @PreUpdate
   public void preUpdate() {
       updatedDate = LocalDateTime.now();
   }
}
</code></pre>
<p>@Prepersist 은 Persist()가 호출되기 전에 이벤트를 발생시키는 어노테이션이고 @PreUpdate 는 엔티티의 상태가 변경되어 DB에 업데이트 되기 전에 호출하는 어노테이션이다. 순수 JPA에서는 위의 어노테이션들을 사용하여 Auditing 기능을 구현할 수 있다. 그럼 Spring Data JPA에서는 어떻게 구현할 수 있을까?</p>
<h3>Spring Data JPA</h3>
<p>Spring Data JPA에서 Auditing 기능을 사용하기 위해서는 아래와 같이 먼저 @EnableJpaAuditing 어노테이션을 사용하여 JPA가 엔티티를 감시할 수 있게 해주어야 한다.</p>
<pre class="less"><code>@Configuration
@EnableJpaAuditing
public class JpaAuditingConfig {}
</code></pre>
<p>그 후, 아래와 같이 BaseEntity 코드를 구현하면 된다.</p>
<pre class="less"><code> @EntityListeners(AuditingEntityListener.class)
 @MappedSuperclass
 @Getter
 public class BaseEntity {
 
     @CreatedDate
     @Column(updatable = false)  // 수정 금지
     private LocalDateTime createdDate;  // 등록 시각
     
     @LastModifiedDate
     private LocalDateTime lastModifiedDate;  // 수정 시각
     
     @CreatedBy
     @Column(updatable = false)
     private Long createdId;  // 등록자
     
     @LastModifiedBy
     private Long lastModifiedId;  // 수정자
}
</code></pre>
<p>사용한 어노테이션들을 하나씩 살펴보자. @EntityListeners 은 JPA Entity에서 이벤트가 발생할 때마다 특정 로직을 실행시킬 수 있는 어노테이션이며 @CreatedDate, @LastModifiedDate 어노테이션을 사용하면 생성된 시간 정보, 수정된 시간 정보를 자동으로 저장할 수 있다. 또한 @CreatedBy, @LastModifiedBy를 사용하여, 데이터가 생성되거나 수정한 유저가 누구인지 추적할 수 있다. 이때, 등록자와 수정자를 처리해주기 위해서는 AuditorAware을 구현한 클래스가 필요하다.</p>
<pre class="java"><code>@RequiredArgsConstructor
@Component
public class LoginUserAuditorAware implements AuditorAware&lt;Long&gt; {

    private final HttpSession httpSession;

    @Override
    public Optional&lt;Long&gt; getCurrentAuditor() {
        SessionUser user = (SessionUser) httpSession.getAttribute("user");
        if(user == null) {
            return null;
        }

        return Optional.ofNullable(user.getId());
    }
}
</code></pre>
<p>위의 코드처럼 LoginUserAuditorAware 클래스를 만들어서 Long Type의 UserId를 반환하도록 했다. 현재 코드는 유저 정보를 세션에서 가져오는 방식이지만 스프링 시큐리티를 사용한다면 Security Context에서 유저 정보를 가져와 구현할 수도 있다.</p>
<hr />
<p>DB를 구상하다보면, 특정 테이블에는 등록자나 수정자까지 알 필요는 없는 테이블들이 존재한다. 이럴 경우에는 각 테이블들의 상황에 맞춘 BaseEntity들을 모두 만들어 주어야 할까? 이런 상황에서는 Base 타입을 분리하고 원하는 타입을 선택해서 상속하면 된다.</p>
<pre class="less"><code>// Base 타입 분리
public class BaseTimeEntity {
     @CreatedDate
     @Column(updatable = false)
     private LocalDateTime createdDate;
     @LastModifiedDate
     private LocalDateTime lastModifiedDate;
}

// 원하는 타입 선택 후, 상속
public class BaseEntity extends BaseTimeEntity {
    @CreatedBy
    @Column(updatable = false)
    private String createdBy;
    @LastModifiedBy
    private String lastModifiedBy;
}
</code></pre>
<p>위처럼 구현한다면, 등록자, 수정자가 필요 없는 엔티티는 BaseTimeEntity의 속성을 상속받고, 모든 속성이 필요한 테이블은 BaseEntity의 속성을 상속받으면 된다.</p>
<h2>Web 확장</h2>
<h3>도메인 클래스 컨버터</h3>
<p>도메인 클래스 컨버터는 HTTP 파라미터로 넘어온 엔티티의 아이디로 자체적으로 엔티티 객체를 찾아서 바인딩해주는 것을 말한다.</p>
<pre class="less"><code> @RestController
 @RequiredArgsConstructor
 public class MemberController {
     private final MemberRepository memberRepository;
     @GetMapping("/members/{id}")
     public String findMember(@PathVariable("id") Member member) {
         return member.getUsername();
     }
}
</code></pre>
<p>HTTP 요청으로 회원 id를 받아왔지만, 도메인 클래스 컨버터가 중간에 동작하여 자동으로 회원 엔티티 객체로 바인딩하여 반환하였다. 실무에서는 잘 사용하지 않는다고 한다.</p>
<h3>페이징과 정렬</h3>
<p>Spring Data JPA가 제공하는 페이징과 정렬 기능을 스프링 MVC에서 편리하게 사용할 수 있다.</p>
<pre class="routeros"><code> @GetMapping("/members")
 public Page&lt;Member&gt; list(Pageable pageable) {
     Page&lt;Member&gt; page = memberRepository.findAll(pageable);
     return page;
 }
</code></pre>
<p>위의 코드처럼 Pageable 인테페이스를 파라미터로 받으면 자동으로 PageRequest 구현체를 만들어 주입시켜준다. 이때, 기본 디폴트 페이지 사이즈는 20, 최대 페이지 사이즈는 2000으로 설정되어 있다. 해당 설정을 변경하고 싶다면, @PageableDefault 어노테이션을 사용하여 변경할 수 있다.</p>
<pre class="bash"><code>@GetMapping("/members_page")
public String list(@PageableDefault(size = 12, sort = "username", direction = Sort.Direction.DESC) Pageable pageable){ 
}</code></pre>
<h2>구현체 분석</h2>
<pre class="less"><code> @Repository
 @Transactional(readOnly = true)
 public class SimpleJpaRepository&lt;T, ID&gt; ...{
     
     @Transactional
     public &lt;S extends T&gt; S save(S entity) {
         if (entityInformation.isNew(entity)) {
             em.persist(entity);
             return entity;
         } else {
             return em.merge(entity);
				 } 
		}
  ...
}
</code></pre>
<p>SimpleJpaRepository는 Spring Data JPA가 제공하는 공통 인터페이스의 구현체이다.</p>
<h3>@Repository</h3>
<p>@Repository는 데이터 액세스 계층에서 사용되는 어노테이션이다. 보통은 컴포넌트 스캔의 대상이 되기 위해 적용하지만 스프링 부트와 JPA를 사용할 경우 예외 변환기를 자동으로 등록하여 @Repository를 적용한 빈을 프록시로 변환한다. 이렇게 변환된 프록시는 데이터 액세스 계층에서 예외가 발생하면 JPA의 예외를 스프링이 추상화한 예외(DataAccessException)로 변환해준다. 이는 발생되는 예외를 JPA, JdbcTemplete, MyBatis와 같은 하부 데이터 접근 기술과 상관없이 통합적으로 관리할 수 있다는 이점이 있다. 즉, 데이터 접근 기술을 변경하더라도 예외 처리하는 로직을 수정할 필요가 없게 된다.</p>
<h3>@Transactional</h3>
<p><b>JPA의 모든 변경은 트랜잭션 안에서 동작</b>한다. 때문에 스프링 데이터 JPA도 변경(등록, 수정, 삭제) 메서드를 처리하기 위해 레포지토리 단에서 자체적으로 트랜잭션 처리를 해두고 있다. 서비스 계층에서 트랜잭션을 시작하면 레포지토리는 해당 트랜잭션을 전파 받아서 사용하고 시작하지 않는다면, 레포지토리에서 자체적으로 트랜잭션을 시작한다. 때문에 스프링 데이터 JPA를 사용할 때, 서비스 계층에서 @Transactional 어노테이션을 명시하지 않았더라도 레포지토리 계층에 자체적으로 걸려있기 때문에 데이터 등록 및 변경이 가능하다.</p>
<blockquote>
<p>데이터를 단순히 조회만 하고 변경하지 않는 트랜잭션에서 readOnly = true 옵션을 사용하면, flush를 생략하여 성능적인 향상을 얻을 수 있다.</p>
</blockquote>
<h2><b>Persistable 인터페이스</b></h2>
<h3>새로운 엔티티를 구별하는 방법</h3>
<p>save() 메서드를 살펴보면, 새로운 엔티티일 경우 persist()를 호출하고 아닐 경우, merge()를 호출한다. 이처럼 Spring Data JPA는 어떻게 새로운 엔티티인지 구별할 수 있을까? Spring Data JPA는 식별자가 null 값일 경우, 새로운 엔티티라고 판별한다. 만약, 식별자가 기본 타입일 때는 0일 경우를 새로운 엔티티로 판별한다.</p>
<p>JPA에서 식별자 생성 전략이 @GeneratedValue일 경우, persist()가 호출 되기 전에는 Id값이 생기지 않는다. 때문에 save()를 호출하는 시점에는 식별자가 존재하지 않기 때문에 새로운 엔티티로 인식하는 것이다. 하지만 식별자 생성 전략이 직접 할당이면 어떨까? <b>직접 할당</b>일 경우, 이미 식별자 값이 존재한 상태로 save()를 호출하게 되고 때문에 merge()가 호출된다. merge()는 우선 DB에 select 쿼리를 날려 값이 존재하는지 확인하고 값이 없을 경우 새로운 엔티티라고 인지하게 되므로 불필요한 select쿼리가 발생하는 비효율이 발생한다. 때문에 이런 상황에서는 <b>Persistable</b> 인터페이스를 사용하여 새로운 엔티티 확인 여부 로직을 직접 구현하는 것이 효과적이다.</p>
<h3>Persistable 인터페이스란?</h3>
<pre class="angelscript"><code>// persistable 인터페이스
public interface Persistable&lt;ID&gt; {
     ID getId();
     boolean isNew();
}
</code></pre>
<p>JPA에서는 엔티티 객체의 상태를 관리하기 위해 다양한 방법을 제공한다. 그중 하나가 <b>Persistable</b> 인터페이스이다. 이 인터페이스는 엔티티가 새로 생성된 상태인지, 아니면 이미 존재하는 상태인지를 판단하는 isNew() 메서드를 제공하며 오버라이드하여 사용이 가능하다.</p>
<h3>isNew()</h3>
<p>isNew() 메서드는 JPA가 엔티티의 상태를 결정하는 데 사용되며 return 값으로 true와 false 값을 반환할 수 있다. 만약 true를 반환한다면 JPA는 해당 엔티티를 새로운 것으로 인식하고 insert 쿼리를 수행하고 반대로 false를 반환하면, JPA는 해당 엔티티가 이미 존재한다고 판단하여 select 쿼리를 수행한 후 update 쿼리를 수행한다.</p>
<pre class="bash"><code>@Entity
@EntityListeners(AuditingEntityListener.class)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Item implements Persistable&lt;String&gt; {
    @Id
    private String id;
    
    @CreatedDate
    private LocalDateTime createdDate;
    
    public Item(String id) {
        this.id = id;
    } 
    
    @Override
    public String getId() {
		return id; 
	}

	@Override
    public boolean isNew() {
        return createdDate == null;
    }
}</code></pre>
<p>&nbsp;</p>
<p>위의 코드는 등록시간의 존재 여부를 기준으로 새로운 엔티티 여부를 판단하는 로직으로 isNew()메서드를 재정의 하였다. 이렇게 식별자 생성 전략이 직접 할당일 경우, isNew() 메서드를 재정의하면, 불필요한 쿼리를 발생시키는 merge()를 호출하여 새로운 엔티티 여부를 판별하는 대신 바로 persist()를 호출함으로써 그 여부를 효율적으로 판별할 수 있다.</p>
<h2>Projection</h2>
<blockquote>DB의 필요한 속성만을 조회하는 것을 projection이라고 한다.</blockquote>
<p>&nbsp;</p>
<p>프로젝션을 하는 방법은 크게 <b>인터페이스 기반 프로젝션</b>과 <b>클래스 기반 프로젝션</b>으로 나눌 수 있다. <b>인터페이스 기반 프로젝션</b>은 조회를 원하는 속성들의 집합으로 인터페이스를 만들고 <b>클래스 기반 프로젝션</b>은 구현체인 Dto를 이용하여 프로젝션을 수행할 수 있다.</p>
<h3>Closed Projection</h3>
<pre class="cs"><code>public interface UsernameOnly {
		String getUsername();
}

// Repository
List&lt;UsernameOnly&gt; findProjectionsByUsername(String username);
</code></pre>
<p>먼저 인터페이스 기반 프로젝션 방법 중 하나인 Closed Projection이다. 이때, 조회할 엔티티의 필드를 getter 형식으로 지정하면 해당 필드만 선택해서 조회할 수 있다. 이런식으로 인터페이스만 구현하면 Spring Data JPA가 자체적으로 구현체를 만들어준다.</p>
<h3>Open Projection</h3>
<pre class="bash"><code>public interface UsernameOnly {
	
    @Value("#{target.username + ' ' + target.age + ' ' + target.team.name}")
    String getUsername();
 }</code></pre>
<p>&nbsp;</p>
<p>위와 같이 SpEL문법을 사용하여 프로젝션을 할 수 있다. 하지만 Closed Projection과는 다르게 먼저 DB에서 모든 엔티티 필드를 다 조회한 후, 애플리케이션 단에서 필요한 필드들만을 추출하는 방식으로 동작한다. 때문에 JPQL select 절 최적화가 불가능하다. 해당 방식으로 프로젝션을 하더라도 엔티티의 모든 필드에 대해 select 쿼리가 나가는 것은 동일하기 때문이다.</p>
<h3>클래스 기반 Projection</h3>
<p>앞서 살펴본 방법들처럼 인터페이스를 생성하지 않고 구체적인 Dto를 만들어 프로젝션하는 것도 가능하다. 아래 코드는 Dto 생성자의 파라미터 명을 분석하여 프로젝션하는 방법이다.</p>
<pre class="angelscript"><code>// DTO
public class UsernameOnlyDto {
     private final String username;
     
     public UsernameOnlyDto(String username) {
         this.username = username;
		 }
     
     public String getUsername() {
         return username;
		 } 
}

// Repository
public interface MemberRepository extends JpaRepository&lt;Member, Long&gt; {
   //클래스 기반 projections
   List&lt;UsernameOnlyDto&gt; findProjectsDtoByUsername(String username);  
}
</code></pre>
<p>&nbsp;</p>
<p>그렇다면 동적으로 프로젝션을 처리할 수는 없을까? Member에서 name 으로 조회하는데 service1에서는 id, name 만 가져오고, service 2에서는 id, email 만 가져오고 싶은 상황을 가정해보자. 즉, 발생하는 쿼리는 같지만(name으로 조회) <b>동적으로 프로젝션</b>(프로젝션 속성이 다름)을 처리하고 싶다면 레포지토리에서 메소드를 정의할 때, 제네릭 타입을 사용하면 된다.</p>
<pre class="excel"><code>&lt;T&gt; List&lt;T&gt; findProjectionsByUsername(String username, Class&lt;T&gt; type);
</code></pre>
<h3>중첩구조 처리</h3>
<pre class="routeros"><code>public interface NestedClosedProjection {
     String getUsername();
     TeamInfo getTeam();
     
     interface TeamInfo {
		     String getName();
     }
}
</code></pre>
<p>&nbsp;</p>
<p>Member에 대한 정보와 함께 Team에 대한 정보도 함께 조회하고 싶다면 위의 코드처럼 중첩 인터페이스를 구현하면 된다. 이때, 기본 Join 설정은 Left Outer Join 방식이다. 하지만, Member 엔티티에 대해서는 프로젝션이 적용되어 username만 조회하지만 내부의 Team 경우에는 모든 필드를 조회한다. 이처럼 프로젝션 대상이 root 엔티티를 넘어가면 JPQL select 최적화가 불가능하다. 때문에 복잡한 쿼리를 해결하기에는 한계가 존재하기에 실무에서는 단순할 때만 사용하고 복잡해질 경우에는 QueryDSL을 이용하여 처리한다.</p>
<h2>네이티브 쿼리</h2>
<pre class="routeros"><code>public interface MemberRepository extends JpaRepository&lt;Member, Long&gt; {
     @Query(value = "select * from member where username = ?", nativeQuery =true)
     Member findByNativeQuery(String username);
 }
</code></pre>
<p>&nbsp;</p>
<p>Spring Data JPA에서 네이티브 쿼리를 사용하기 위해서는 @Query 어노테이션을 사용하여 쿼리를 입력하고, nativeQuery 옵션을 true로 설정하면 된다.</p>