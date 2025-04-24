<p><a href="https://wing1008.tistory.com/43" rel="noopener" target="_blank">이전의 포스팅</a>에서 MySQL의 InnoDB에서는 Repeatable Read 격리 수준에서도 Phantom Read(유령 읽기) 문제가 발생하지 않는다는 내용을 언급했었다. 이번 포스팅에서는 그 이유와 InnoDB 스토리지 엔진의 잠금에 대해 알아보고자 한다.</p>
<h2>InnoDB 스토리지 엔진의 잠금</h2>
<p><b>InnoDB 스토리지 엔진</b>은 MySQL에서 제공하는 잠금과는 별개로 <b>스토리지 엔진 내부에서 레코드 기반의 잠금 방식을 탑재</b>하고 있다.&nbsp;</p>
<h3>레코드 락</h3>
<p>일반적으로 레코드 자체만을 잠그는 것을 <b>레코드 락(Record Lock)</b>이라고 한다. 다만, 한가지 중요한 점은 <b>InnoDB 스토리지 엔진은 레코드 자체가 아니라 인덱스의 레코드를 잠근다.&nbsp;</b>인덱스 레코드에 락을 거는 것과 테이블 레코드에 락을 거는 것은 큰 차이가 있다.</p>
<h4>인덱스와 잠금</h4>
<p>언급 했듯이, InnoDB의 잠금은 레코드를 잠그는 것이 아니라 인덱스를 잠그는 방식으로 처리된다. 즉, 변경해야 할 레코드를 찾기 위해 검색한 인덱스의 레코드를 모두 락을 걸어야 한다.&nbsp;</p>
<p><figure class="imageblock alignCenter"><span><img height="363" src="https://blog.kakaocdn.net/dn/YN0Q3/btsNxCY6o1L/YdRUUq7EkjcMLAAeTCXxM0/img.png" width="350" /></span></figure>
</p>
<p>사진과 같이, 테이블에 last_name이 Kim인 구성원이 503명이 있고 이름이 Lisa인 사람은 딱 1명만 존재한다. 이때, last_name에만 인덱스가 걸려져있는 상황에서 last_name이 kim이고 이름이 Lisa인 구성원의 정보를 변경하는 UPDATE쿼리를 실행한다고 생각해보자.</p>
<p>UPDATE 문에 의해 영향받는 레코드는 1건이다. 하지만 1건을 업데이트 하기 위해 503건의 인덱스 레코드에 잠금이 걸린다. 왜냐하면 MySQL은 테이블 레코드가 아닌 인덱스에 잠금을 걸기 때문이다. 인덱스는 last_name으로만 구성되어 있기 때문에, 해당 레코드를 갱신하기 위해서는 <b><span style="color: #333333;">인덱스를 통해 검색되는 모든 레코드에 잠금</span></b>을 걸게 된다. 만약 인덱스가 하나도 없다면, 테이블을 풀 스캔하면서 모든 레코드를 잠그게 된다. 때문에 MySQL에서의 인덱스 설계는 매우 중요하다.</p>
<h3>갭 락</h3>
<p>일반 상용 DBMS와 다르게 InnoDB 스토리지 엔진에서는 레코드 락 뿐 아니라 레코드와 레코드 사이의 간격을 잠그는 갭 락 이라는 것이 존재한다. <b>갭 락(Gap Lock)</b>은 레코드 자체가 아니라 레코드와 바로 인접한 레코드 사이의 간격만을 잠그는 것을 의미한다. 즉, 갭 락을 통해 InnoDB는&nbsp;<b>레코드와 레코드 사이의 간격에 새로운 레코드가 생성(Insert)되는 것을 제어</b>할 수 있다. 갭 락 그 자체보다는 이어서 설명할 넥스트 키 락의 일부로 자주 사용된다.</p>
<p><figure class="imageblock alignCenter"><span><img height="380" src="https://blog.kakaocdn.net/dn/yHQE2/btsNyt8dD8k/kwlADOqbF5KWkU3uYIBKkk/img.png" width="206" /></span></figure>
</p>
<p>예를 들어, 테이블에서 20이상 25이하의 조건으로 데이터를 검색한다면, 현존하는 레코드인 20와 22에 걸리는 락이 바로 레코드 락이다. 그리고 아직 실존하지 않는 21, 23, 24 그리고 25가 추가될 수 있는 공간에 걸리는 락이 갭 락인 것이다.</p>
<p>&nbsp;</p>
<p>InnoDB에서는 대부분 보조 인덱스를 이용한 변경 작업은 이어서 설명할 갭 락(Gap Lock) 또는 넥스트 키 락(Next key Lock)을 사용하지만 프라이머리 키 또는 유니크 인덱스에 의한 변경 작업에서는 갭(Gap)에 대해서는 잠그지 않고 레코드 자체에 대해서만 락을 건다. 정확히는, 프라이머리 키나 유니크 인덱스의 경우에는 유일하다는 걸 보장받고 있기 때문에, 정확한 일치 조건(=)으로 검색을 할 경우, 갭(Gap)에 대해서는 잠그지 않고 레코드 자체에 대해서만 락을 건다.</p>
<h4><a href="https://dev.mysql.com/doc/refman/8.4/en/innodb-locks-set.html?utm_source=chatgpt.com" rel="noopener" target="_blank">MySQL 공식 문서</a></h4>
<blockquote>For a unique index with a unique search condition, InnoDB locks only the index record found, not the gap before it.<br /><br />​For other search conditions, and for non-unique indexes, InnoDB locks the index range scanned, using gap locks or next-key locks to block insertions by other sessions into the gaps covered by the range.</blockquote>
<h3>넥스트 키 락</h3>
<p><b>넥스트 키 락(Next key Lock)</b>은 레코드 락과 갭 락을 합쳐 놓은 형태의 잠금이다. innodb_locks_unsafe_for_binlog 시스템 변수는 트랜잭션이 UPDATE/DELETE 시 사용하는 잠금 범위를 줄여서 성능을 올릴 수 있게 하는 설정으로, 해당 설정을 비활성화하면<span style="color: #666666;">(default)</span> 변경을 위해 검색하는 레코드에는 넥스트 키 락 방식으로 잠금이 걸린다. 즉, InnoDB는 UPDATE, DELETE, SELECT ... FOR UPDATE에서 <b>넥스트 키 락</b>을 기본적으로 사용한다.</p>
<p>하지만 넥스트 키 락과 갭 락은 락의 범위를 넓게 가져가기 때문에 데드락이 발생하거나 다른 트랜잭션을 기다리게 만드는 일이 자주 발생한다고 한다. 가능하다면 바이너리 로그 포맷을 ROW 형태로 바꿔서 넥스트 키 락이나 갭 락을 줄이는 것이 좋다.</p>
<h3>자동 증가 락</h3>
<p>MySQL에서는 자동 증가하는 숫자 갑을 추출(채번)하기 위해 AUTO_INCREMENT라는 칼럼 속성을 제공한다. 동시에 여러 레코드가 INSERT되는 경우, 저장되는 각 레코드는 중복되지 않고 저장된 순서대로 증가하는 일련번호 값을 가져야 한다. 때문에 InnoDB 스토리지 엔진에서는 이를 위해 내부적으로 AUTO_INCREMENT락(자동 증가 락)이라는 테이블 수준의 잠금을 사용한다.<br />자동 증가 락은 새로운 레코드를 저장하는 쿼리에서만 필요하며, UPDATE나 DELETE 등의 쿼리에서는 걸리지 않는다. 다른 잠금과 달리 자동 증가 락은 트랜잭션과 관계없이 INSERT나 REPLACE 문장에서 AUTO_INCREMENT 값을 가져오는 순간만 락이 걸렸다가 즉시 해제된다. 자동 증가 락은 아주 짧은 시간동안 걸렸다가 해제되는 잠금이라서 대부분의 경우 문제가 되지 않는다.</p>
<p>&nbsp;</p>
<hr contenteditable="false" />
<h2>갭락이 Phantom Read를 방지한다!</h2>
<p>결론부터 말하면 InnoDB의<b> MVCC와&nbsp;갭 락&nbsp;메커니즘이 Repeatable Read 격리 수준에서도 Phantom Read 문제를 방지</b>한다.</p>
<p><figure class="imageblock alignCenter"><span><img height="802" src="https://blog.kakaocdn.net/dn/oMN3Q/btsNwvMLsqB/KG4w9RJpGWtVrkDYpgxPr1/img.png" width="653" /></span></figure>
</p>
<p>트랜잭션 B가 `where id &gt;= 500`을 조건으로 데이터를 읽은 순간, InnoDB는 갭 잠금을 설정하게 된다. 즉, id 500보다 큰 데이터가 들어갈 수 있는 갭(레코드와 레코드 사이)에 잠금이 설정되어 레코드 삽입이 차단되게 된다. 특히, 갭 락은 "존재하지 않는 레코드 사이의 공간"도 잠그기 때문에, id = 501과 같은 아직 없는 데이터라도, 조건 범위에 포함된다면 삽입이 불가능해진다. 때문에 트랜잭션 A가 id가 501인 데이터를 삽입하고자 해도 갭 락으로 인해 트랜잭션B가 커밋되기 전까지 삽입이 블로킹되고, 따라서 트랜잭션 B가 다시 동일한 쿼리를 실행해도, 여전히 Lisa만 조회되어 팬텀 리드 현상이 방지되는 것이다.</p>
<p>&nbsp;</p>