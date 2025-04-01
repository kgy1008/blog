<h2>공통 인터페이스 구성</h2>
<p><figure class="imageblock alignCenter"><span><img height="668" src="https://blog.kakaocdn.net/dn/Vniwl/btsMOtUqzX5/dZeoCPDSNcsVC0HzLYcs6K/img.png" width="486" /></span></figure>
</p>
<h2>엔티티의 기본생성자</h2>
<h3>기본생성자가 필요한 이유</h3>
<p>JPA 구현체마다 스펙이 조금 달라서 기본 생성자를 만들지 않아도 정상적으로 작동하는 경우가 있지만,&nbsp;&rsquo;엔티티에는 기본 생성자가 있어야 한다&rsquo;가 공식 스펙이기 때문에 반드시 기본 생성자를 만들어주는 것이 좋다. 그렇다면 JPA는 왜 엔티티에 기본 생성자를 만들도록 강제하고 있을까?</p>
<p>JPA는 데이터를 DB에서 조회해 온 뒤 객체를 생성할 때 **Reflection(리플렉션)**을 사용하기 때문이다. 리플렉션은 클래스 이름만 알면 생성자, 필드, 메서드 등 클래스의 모든 정보에 접근이 가능하다. 하지만 리플렉션이 가져올 수 없는 정보가 있는데 바로&nbsp;&rsquo;생성자의 매개변수 정보&rsquo;다. 때문에 리플렉션으로 생성할 객체에 모든 필드를 받는 생성자가 있더라도 리플렉션은 해당 생성자를 호출할 수가 없다. 리플렉션은 이러한 문제를 해결하기 위해 <b>기본 생성자로 객체를 생성하고 필드 값을 강제로 매핑</b>해주는 방식을 사용한다. 결론적으로 기본 생성자가 존재하지 않는다면 데이터베이스에서 조회해 온 값을 엔티티로 만들 때 객체 생성 자체에 실패하게 되기 때문에, JPA에서는 기본 스펙으로 기본 생성자를 반드시 생성해 줄 것을 정해놓고 있는 것이다.</p>
<h3>접근제어자 private이 불가능한 이유</h3>
<p>프록시 객체는 실제 객체에 대한 참조를 보관하여, 프록시 객체의 메서드를 호출했을 때 실제 객체의 메서드를 호출한다. 그래서 실제 객체 타입 자리에 들어가도 문제 없이 사용할 수 있다. 이러한 동작이 가능한 이유는 <b>프록시가 실제 객체를 상속</b>한 타입을 가지고 있기 때문이다. 프록시 객체가 실제 객체를 상속받고 있기 때문에 &lsquo;<b>기본 생성자는 최소 protected 접근 제한자를 가져야 한다</b>&rsquo; 는 규칙과 &lsquo;<b>엔티티 클래스는 final로 정의할 수 없다</b>&rsquo; 라는 규칙이 생겨나게 된 것이다. 만약 기본 생성자가 private이면 프록시 생성 시&nbsp;super를 호출할 수 없을 것이고, 엔티티를 final로 선언한다면 상속이 불가능하게 되기 때문이다.</p>
<h2>Spring Data JPA</h2>
<blockquote><span style="font-family: 'Noto Serif KR';">Spring Data JPA는 메소드 이름으로 쿼리를 생성할 수 있다.</span></blockquote>
<p>&nbsp;</p>
<p>Spring Data JPA는 메소드 이름을 분석해서 자체적으로 JPQL을 생성하고 실행한다. 이때, 엔티티의 필드명이 변경되면 인터페이스에 정의한 메서드 이름도 꼭 함께 변경해야 한다.</p>
<h3>@Query</h3>
<p>@Query 어노테이션을 사용하면 JPQL 쿼리를 직접 작성할 수 있다. 실행할 메서드에 정적 쿼리를 직접 작성하므로 이름 없는 Named 쿼리라 할 수 있다. 때문에 JPA Named 쿼리와 마찬가지로 애플리케이션 실행 시점에 문법 오류를 발견할 수 있다.</p>
<pre class="less"><code> @Query("select m from Member m where m.username= :username and m.age = :age")
 List&lt;Member&gt; findUser(@Param("username") String username, @Param("age") int age);
</code></pre>
<p>&nbsp;</p>
<p>이때, DTO로 직접 조회를 하기 위해서는 new 키워드와 함께 dto가 위치한 패키지 명을 작성해주어야한다.</p>
<pre class="sql"><code> @Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) " +
         "from Member m join m.team t")
List&lt;MemberDto&gt; findMemberDto();
</code></pre>
<p>반환값으로는 단건 및 컬렉션 모두 가능한데, 조회 결과가 없을 경우 <b>컬렉션은 빈 컬렉션으로 반환</b>되고 <b>단건의 경우 null 값으로 반환</b>된다. 즉, 조회 결과가 없어도 예외가 발생하지 않는다.</p>
<h3>파라미터 바인딩</h3>
<p>JPA에서&nbsp;@Query&nbsp;어노테이션을 사용할 때, 일반적으로&nbsp;@Param&nbsp;애너테이션을 사용하여 파라미터명과 매핑할 이름을 지정해주어야 한다. 하지만 파라미터가 한개이거나 파라미터 명과 매핑할 이름이 동일한 경우, 아래와 같이 @Param 어노테이션을 생략할 수 있다.</p>
<pre class="reasonml"><code> @Query("select m from Member m where m.username= :username and m.age = :age")
 List&lt;Member&gt; findUser(String username, int age);
</code></pre>
<p>Spring Data JPA 2.0 이상부터는 파라미터명과 매핑할 이름을 지정하지 않아도 자동으로 매핑된다고 한다.</p>
<p>아래 코드와 같이 <b>컬렉션 타입</b>이 파라미터로 들어오면 in절로 바인딩할 수 있다.</p>
<pre class="less"><code> @Query("select m from Member m where m.username in :names")
 List&lt;Member&gt; findByNames(@Param("names") List&lt;String&gt; names);
</code></pre>
<h2>페이징과 정렬</h2>
<p>Spring Spring에서는 Pagination을 지원하는&nbsp;Pageable&nbsp;****인터페이스를 제공한다. Pageable&nbsp;을 이용해서 페이지 번호, 페이지당 항목 수, 필요에 따라 정렬 정보를 추가로 지정할 수 있고 이렇게 지정한 정보들을 이용해서 Page 객체나 Slice객체로 반환할 수 있다.</p>
<h3>PageRequest</h3>
<p>Spring Data JPA에서 제공하는&nbsp;Pageable&nbsp;구현체 중 하나로, 페이지 정보를 생성하는 클래스이다.</p>
<p>페이지 번호, 페이지당 항목 수, 정렬 정보를 이용하여&nbsp;Pageable&nbsp;인터페이스를 구현할 수 있다.</p>
<ul>
<li><b>내장 함수 목록</b><b>size</b>&nbsp;: 한 페이지당 최대 항목 수<b>direction</b>&nbsp;: 정렬 방향(ASC, DESC)</li>
<li><b>properties</b>&nbsp;: 정렬 대상 속성명</li>
<li><b>sort</b>&nbsp;: 정렬 정보(생략 가능)</li>
<li><b>page</b>&nbsp;: 조회할 페이지 번호(0부터 시작)</li>
</ul>
<h3>Page</h3>
<p>Page 반환 타입은 total count를 계산하는 쿼리 결과를 포함하는 페이징이다.</p>
<p><figure class="imageblock alignCenter"><span><img height="161" src="https://blog.kakaocdn.net/dn/MRoLB/btsMMbnSpEH/uAYaFdWt1CIEQnZBfMS3F1/img.png" width="666" /></span></figure>
</p>
<p>위의 사진처럼, 아래에 몇 번째 페이지에 위치하고 있는지를 계산하기 위해서는 전체 항목의 개수를 세어야한다. 이러한 기능을 포함하고 있다면 반환 타입 Page를 사용하면 편리하게 처리할 수 있다.</p>
<pre class="angelscript"><code>// repository
public interface MemberRepository extends Repository&lt;Member, Long&gt; {
     Page&lt;Member&gt; findByAge(int age, Pageable pageable);
}

// service
PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));

Page&lt;Member&gt; page = memberRepository.findByAge(10, pageRequest);

</code></pre>
<p>위의 코드처럼 PageRequest&nbsp;객체를 생성하고 JpaRepository에서 Page&nbsp;객체를 반환하는 메서드에 파라미터로 전달하면,&nbsp;Pagination을 구현할 수 있다.</p>
<pre class="reasonml"><code>List&lt;Member&gt; content = page.getContent(); //조회된 데이터 

assertThat(content.size()).isEqualTo(3); //조회된 데이터 수 
assertThat(page.getTotalElements()).isEqualTo(5); //전체 데이터 수 
assertThat(page.getNumber()).isEqualTo(0); //페이지 번호 
assertThat(page.getTotalPages()).isEqualTo(2); //전체 페이지 번호 
assertThat(page.isFirst()).isTrue(); //첫번째 항목인가? 
assertThat(page.hasNext()).isTrue(); //다음 페이지가 있는가?
</code></pre>
<p>위는 page 인터페이스가 제공하는 함수들이다.</p>
<p>만약 엔티티를 여러개 조인한 채로 페이징 처리를 하게 된다면 Count 쿼리 또한 테이블을 조인한 후, 계산하게 된다. 이는 성능상으로 불필요한 join을 실행한 셈이다. 때문에 아래와 같이 Count Query를 따로 명시하여 최적화가 가능하다.</p>
<pre class="sql"><code> @Query(value = "select m from Member m left join m.team",
        countQuery = "select count(m.username) from Member m")
Page&lt;Member&gt; findMemberAllCountBy(Pageable pageable); 
</code></pre>
<p>이렇게 Count Query를 따로 명시하면, 전체 개수를 세는 count 쿼리에 대해서는 Team 엔티티와 조인하지 않고 쿼리를 발생시킨다.</p>
<h3>Slice</h3>
<p>Slice는 Page 보다 더 상위 인터페이스로 추가 count 쿼리 없이 다음 페이지만 확인이 가능하다.</p>
<p><figure class="imageblock alignCenter"><span><img height="250" src="https://blog.kakaocdn.net/dn/bj9zpd/btsMMabrGxB/2Hk9hilWE6QpWgIIRDYgyK/img.png" width="666" /></span></figure>
</p>
<p>위의 사진은 앞선 페이징 방식과 달리 더보기 버튼을 누르면 일정 개수의 글을 불러오는 방식이다. 이러한 방식은 전체 count 쿼리를 할 필요가 없으므로 반환타입으로 Slice를 사용하여 구현할 수 있다.</p>
<h2>벌크성 수정 쿼리</h2>
<h3>@Modifying</h3>
<p>지금까지 우리는 더티 체킹(dirty checking)으로 단일 데이터에 대해 수정을 할 수 있다고 배웠다. 하지만 여러 데이터들의 값들을 한번에 수정하고 싶다면 어떨까? 이처럼 대량의 데이터를 한 번에 수정하거나 삭제하는 방법을 <b>벌크 연산</b>이라고 한다. 벌크 연산을 사용하면 한 번의 쿼리로 여러 레코드를 수정하거나 삭제할 수 있기 때문에 네트워크 트래픽을 줄이고 데이터베이스 서버의 부하도 줄일 수 있다.</p>
<pre class="less"><code>@Modifying
@Query("update Member m set m.age = m.age + 1 where m.age &gt;= :age")
int bulkAgePlus(@Param("age") int age);
</code></pre>
<p>Spring JPA에서는 벌크성 수정, 삭제 쿼리를 작성할 때 반드시 @Modifying 어노테이션을 붙여주어야 한다. 해당 어노테이션을 붙여야지만 executeUpdate()로 자동 실행되며 만약 해당 어노테이션이 없을 경우에는 에러가 발생하게 된다.</p>
<h3>주의점</h3>
<p>이런 벌크 연산은 영속성 컨텍스트를 무시하고 DB에 쿼리를 직접적으로 실행하기 때문에 영속성 컨텍스트에 있는 엔티티의 상태와 DB 엔티티 상태의 부정합 문제가 발생할 수 있다. 때문에 만약 같은 트랜잭션 내에서 벌크 연산 직후에 해당 데이터들에 대해 다시 처리를 해야하는 상황이 존재한다면 영속성 컨텍스트를 반드시 초기화해주어야 한다.</p>
<pre class="autoit"><code>// default 값은 false이다.
@Modifying(clearAutomatically = true)
</code></pre>
<p>이때, 위와 같이 clearAutomatically = true 조건을 붙여주면 벌크성 쿼리를 실행하고 나면 자동으로 영속성 컨텍스트를 초기화 시켜줄 수 있다.</p>
<h2>@EntityGraph</h2>
<p>@EntityGraph는 fetch Join과 마찬가지로 JPA N+1 문제를 해결할 수 있는 방법 중 하나이다.</p>
<pre class="less"><code>//공통 메서드 오버라이드
@Override
@EntityGraph(attributePaths = {"team"})
List&lt;Member&gt; findAll();
//JPQL + 엔티티 그래프 
@EntityGraph(attributePaths = {"team"}) 
@Query("select m from Member m") 
List&lt;Member&gt; findMemberEntityGraph();

//메서드 이름으로 쿼리에서 특히 편리하다.
@EntityGraph(attributePaths = {"team"})
List&lt;Member&gt; findByUsername(String username)
</code></pre>
<p>위의 코드에서 확인할 수 있듯, &nbsp;@EntityGraph를 사용하여 attributePaths만 명시하면 직접 JPQL을 작성하지 않아도 자동으로 대상 엔티티와 left fetch join이 실행된다.</p>
<h3>FetchJoin과의 차이점</h3>
<p><figure class="imageblock alignCenter"><span><img height="349" src="https://blog.kakaocdn.net/dn/djcBT1/btsMNLnLTwp/RN0ZXkluDpbGOWyIKgAIx1/img.png" width="633" /></span></figure>
</p>
<p><b>Fetch Join</b>의 경우 기본으로 inner join 방식으로 조인하지만 (앞에 left 키워드를 붙여 left outer join으로도 조인이 가능하다.) <b>EntityGraph</b>의 경우에는 outer left join 방식만을 사용한다.</p>
<h3>JPA Hint</h3>
<p>JPA에서는 영속성 컨텍스트에 데이터를 저장할 때, 아래 사진처럼 스냅샷도 함께 저장하게 된다. 기존 값과의 비교를 통해 dirty checking 기능을 수행하기 위해서는 스냅샷이 반드시 필요하다.</p>
<p><figure class="imageblock floatLeft"><span><img height="182" src="https://blog.kakaocdn.net/dn/WVVID/btsMMavE6oz/CQassQXXMDu83AjQIfKnWK/img.png" width="385" /></span></figure>
</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>때문에 영속성 컨텐스트는 항상 스냅샷도 저장한다.</p>
<p>하지만 엔티티를 변경하지 않아도 되는 상황에서는 이는 불필요한 메모리 낭비일 뿐이다.</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>여기서 jpa Hint 를 사용하면 읽기 전용 메소드로 만들고 스냅샷을 사용 안하게 만들 수 있다.</p>
<pre class="less"><code>@QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))
 Member findReadOnlyByUsername(String username);
</code></pre>
<h3>Lock</h3>
<pre class="sql"><code>@Lock(LockModeType.PESSIMISTIC_WRITE)
List&lt;Member&gt; findByUsername(String name);
</code></pre>
<p>위의 코드처럼 @Lock 어노테이션을 사용하면 비관적 락 기능을 JPA에서 쉽게 구현할 수 있다.</p>