<h2>지연 로딩? 즉시 로딩?</h2>
<p>다들 3차 세미나 기억나시나요?</p>
<pre class="less"><code>@Entity
@Getter
@NoArgsConstructor
public class Post extends BaseTimeEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String content;

    @ManyToOne(fetch = FetchType.LAZY)
    private Blog blog;
}
</code></pre>
<p>위의 코드는 3차 세미나에서 진행했던 코드 중 일부분입니다!</p>
<p>저는 개인적으로 3차 세미나 내용을 복습하면서 fetch = FetchType.LAZY 가 무엇인지, 왜 지연 로딩 방식으로 설정했는지 궁금해서 공부를 하다보니 N+1 문제에 대해서도 알게 되어 이 내용에 관해 아티클을 작성해보고자 합니다:D</p>
<h3>프록시 객체 (Proxy Object)</h3>
<p>먼저 프록시 객체에 대한 개념부터 간단히 살펴봅시다.</p>
<blockquote>  프록시 객체는 엔티티의 실제 데이터를 데이터베이스에서 가져오는 시점을 지연시키기 위해 원본(타겟) 객체를 대신해서 호출될 가짜 객체이다.</blockquote>
<p><figure class="imageblock alignCenter"><span><img height="149" src="https://blog.kakaocdn.net/dn/blxnZT/btsMMcUjAz2/McjGA06kXVhs0WPOoQ1Ghk/img.png" width="547" /></span></figure>
</p>
<p>프록시 객체는 클라이언트 코드와 실제 데이터베이스에서 로드된 엔티티 객체(타겟 객체) 사이에 위치하기 때문에 클라이언트는 실제 엔티티 객체에 직접 접근하지 않고, 프록시 객체를 통해 간접적으로 접근하게 됩니다.</p>
<p>쉽게 비유하자면 타겟 객체를 집 주인이라고 생각했을 때, 프록시 객체는 집 주인을 대신해서 계약을 요청받는 중개인으로 생각하면 되겠네요!</p>
<h3>지연 로딩 (Lazy Loading)</h3>
<p>지연 로딩은 엔티티가 로드될 때, 연관된 엔티티를 즉시 로드하지 않고 필요한 시점에 연관된 객체의 데이터를 로드하는 방식입니다. @OneToMany 랑 @ManyToMany 는 기본 설정이 지연로딩이라고 합니다:D</p>
<pre class="kotlin"><code>@ManyToOne(fetch = FetchType.LAZY)  // 지연 로딩 설정 방법
</code></pre>
<p>지연 로딩 방식에서는 연관된 엔티티 데이터는 실제로 접근할 때까지 로드되지 않습니다. 즉, 클라이언트 코드가 객체의 메서드를 호출해야 비로소 프록시 객체는 그 순간 데이터베이스에 접근하여 실제 데이터를 로드하게 됩니다.</p>
<p>코드로 한번 살펴봅시다!</p>
<pre class="livescript"><code>public Blog getBlogByPostId(Long postId) {
    Post post = postRepository.findById(postId).orElseThrow(() -&gt; new NotFoundException("Post not found"));
    Blog blog = post.getBlog();  
    return blog;
}
</code></pre>
<p>Post 엔티티 내에서 Blog에 대한 접근은 FetchType.LAZY 즉, 지연 로딩으로 설정되어 있기 때문에 Post 객체만 먼저 로드되고 Blog에 대한 프록시 객체가 생성되게 됩니다. 그 후, getter 함수를 통해 post.getBlog() 를 호출하면, 프록시 객체는 실제 데이터가 필요한 시점이기 때문에 비로소 데이터베이스에 접근하여 Blog 데이터를 로드하게 되는 것이죠!</p>
<h3>즉시 로딩 (Eager Loading)</h3>
<p>즉시 로딩이란 말 그대로 데이터를 조회할 때, 연관된 모든 객체의 데이터까지 한 번에 불러오는 방식입니다. @ManyToOne 랑 @OneToOne 는 기본 설정이 즉시 로딩이라고 하네요:D</p>
<pre class="kotlin"><code>@OneToMany(fetch = FetchType.EAGER)  // 즉시 로딩 설정 방법
</code></pre>
<p>역시 코드로 한번 살펴봅시다.</p>
<p>아까의 코드에서 만약 Post 엔티티와 Blog 엔티티가 FetchType.EAGER 즉, 즉시 로딩으로 설정되어 있다면 어 떻게 될까요? postRepository.findById(postId) 메서드를 통해 Post 엔티티를 조회할 때 즉시 Post 데이터와 연관된 Blog 데이터가 함께 로드됩니다. 프록시 객체의 생성 없이 별도의 쿼리를 통해 바로 Blog 엔티티에 접근하는 것이죠!</p>
<hr contenteditable="false" />
<p>이처럼 즉시 로딩 방식은 지연 로딩 방식에 비해 연관된 데이터가 필요한 작업에서 빠르게 처리를 할 수 있다는 장점이 있겠네요! 하지만!!! 특히 실무에서는 <b>즉시 로딩을 가급적 사용하지 않는다</b>고 합니다. 이 이유로는 앞으로 설명할 JPA N+1 문제와도 연관이 있습니다.</p>
<h2>JPA N+1 문제</h2>
<h3>JPA N+1 문제란?</h3>
<p>JPA N+1 문제란 <b>데이터를 조회할 때, 1개의 쿼리로 요청이 처리할 것으로 기대했으나 의도하지 않은 N개의 쿼리가 추가적으로 더 발생하는 현상</b>을 말합니다.</p>
<p>말로만 들으니 너무 모호한 것 같네요. 이 역시 코드를 한번 살펴봅시다.</p>
<pre class="reasonml"><code>public void getAllBlogTitleByPostId(Long postId) {
    List&lt;Post&gt; posts = postRepository.findByPostId(postId);
    for (Post post : posts) {
			    System.out.println(post.getBlog().getTitle());
		}
}
</code></pre>
<p>만약 <b>즉시로딩 관계</b>라면, 해당 메소드를 실행시키면 다음 순서에 따라 쿼리문이 발생하겠죠!</p>
<ol>
<li>주어진 postId에 대응하는 모든 Post 객체들을 데이터베이스로부터 로드하는 쿼리를 발생시킨다. 이때 findByPostId 와 같이 JPQL으로 엔티티를 조회할 경우, Fetch 전략을 무시하고 SQL문을 실행하게 된다.</li>
<li>먼저 조회한 Post 엔티티에 연관관계가 설정된 Blog 엔티티가 존재하고 즉시로딩 관계이기 때문에 즉시 Blog를 로드하는 추가 쿼리가 실행된다. (N번의 쿼리 발생)</li>
</ol>
<p>이처럼 즉시로딩 관계에서는 Post를 조회하는 1개의 쿼리를 기대했으나 의도하지 않은 N개의 쿼리가 추가적으로 더 발생하는 N+1문제가 발생하게 됩니다.</p>
<p>그렇다면 <b>지연로딩 관계</b>에서는 N+1문제가 발생하지 않을까요? 다시 위의 코드를 지연로딩 관계라고 생각하고 차근차근 살펴봅시다!</p>
<ol>
<li>주어진 postId에 대응하는 모든 Post 객체들을 데이터베이스로부터 로드하는 쿼리를 발생시킨다. (이때 Post와 Blog는 지연로딩 관계이기 때문에 Blog 객체는 즉시 로드되지 않고 프록시 객체로 존재합니다.)</li>
<li>리스트에서 각 Post 객체에 대해 post.getBlog().getTitle() 메서드를 호출할 때마다, 각각의 Post 객체에 대해 개별적으로 Blog를 로드하는 추가 쿼리가 실행된다. (N번의 쿼리 발생)</li>
</ol>
<p>결과적으로 지연로딩 관계에서도 마찬가지로 첫번째 Post 객체를 로드하는 쿼리 1개와 각 Post 객체의 Blog를 로드하는 추가적인 쿼리 N개(각 Post 마다 1개)가 발생하게 되어 총 N+1회의 쿼리가 발생하게 됩니다.</p>
<hr contenteditable="false" />
<p>이처럼 JPA N+1 문제는 fetch 타입이 원인인 것은 아니지만(즉시 로딩, 지연 로딩 모두 발생할 수 있음) 즉시 로딩일 경우, 특히 개발자가 제어할 수 없는 쿼리가 실행되고 더 자주 N+1문제에 마주하기 때문에 사용을 지양한다고 하네요:D</p>
<h3>해결 방법</h3>
<p>대용량의 데이터가 존재할 때, JPA N+1문제가 발생한다면 엄청나게 많은 쿼리문이 실행되게 될 것이고 이는 심각한 문제를 유발하게 되겠죠. 그럼 해당 문제를 어떻게 해결할 수 있을까요?</p>
<p>바로 Fetch Join을 이용하여 해결할 수 있습니다!!</p>
<blockquote>  Fetch Join은 연관된 엔티티나 컬렉션을 한 번에 같이 조회하는 기능이다.</blockquote>
<p>&nbsp;</p>
<p>즉, Fetch Join은 조회의 주체가 되는 엔티티와 그 관련 엔티티들까지 함께 조회하기 때문에 한 번의 쿼리로 필요한 정보를 모두 가져올 수 있게 됩니다:)</p>
<p>Fetch Join을 사용하는 방법은 1. 쿼리문에 직접 fetch를 명시하는 방법과 2. @EntityGraph 라는 어노테이션을 사용하여 가져오는 방법이 있습니다.</p>
<p>앞에서 설명한 코드를 JPA N+1 문제가 발생하지 않도록 고쳐봅시다!</p>
<pre class="kotlin"><code>import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import java.util.List;

public interface PostRepository extends JpaRepository&lt;Post, Long&gt; {

		// 첫번째 방법 (JPQL 사용)
    @Query("SELECT p FROM Post p JOIN FETCH p.blog WHERE p.postId = :postId")
    List&lt;Post&gt; findByPostId(Long postId);
    
    // 두번째 방법 (@EntityGraph 어노테이션 사용)
    @EntityGraph(attributePaths = {"blog"})
    List&lt;Post&gt; findByPostId(Long postId);
}
</code></pre>
<p>지금까지 프록시 객체의 개념부터 JPA N+1 문제와 해결방법까지 알아보았습니다. 실무에서는 즉시 로딩은 최대한 사용하지 말고 지연 로딩과 fetch join을 함께 사용할 것을 권장하고 있습니다.</p>