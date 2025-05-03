<h2>Redis와 캐시</h2>
<h3>캐시란?</h3>
<p><figure class="imageblock alignCenter"><span><img height="273" src="https://blog.kakaocdn.net/dn/bT8rre/btsNJ4N28i8/kmcMwfi5KxjIoUNDu2GVtk/img.png" width="633" /></span></figure>
</p>
<p>캐시란, 데이터의 원본보다 더 빠르고 효율적으로 엑세스할 수 있는 임시 데이터 저장소를 말한다. 캐시는 자주 사용하는 데이터를 메모리에 저장해 디스크나 데이터베이스 접근을 줄이고, 이로 인해 응답 시간을 단축시켜 전체 시스템의 성능을 향상시킨다.</p>
<h3>캐시로서의 레디스</h3>
<p>레디스는 단순하게 키-값 형태로 데이터를 저장하므로, 굉장히 간단하며 자체적으로 다양한 자료 구조를 제공하기 때문에 애플리케이션에서 사용하던 list, hash 등의 자료 구조를 변환하는 과정 없이 레디스에 바로 저장할 수 있다. 또한 레디스는 모든 데이터를 메모리에 저장하는 인메모리 데이터 저장소이기 때문에 데이터를 검색하고 반환하는 것이 상당히 빠르다는 특징을 가지고 있다.</p>
<h2>캐싱 전략</h2>
<h3>읽기 전략</h3>
<h4>1. Look Aside</h4>
<p>애플리케이션에서 데이터를 읽을 때 주로 사용하는 전략으로 레디스를 캐시로 사용할 때 가장 일반적으로 배치하는 방법이다.</p>
<p><figure class="imageblock widthContent"><span><img height="426" src="https://blog.kakaocdn.net/dn/4dmok/btsNJ6rzq9a/AHrsPOVApfntZnP1bY0ZM1/img.png" width="1478" /></span></figure>
</p>
<p>애플리케이션은 먼저 데이터가 캐시에 존재하는지를 확인한 뒤, 캐시에 데이터가 존재하면(Cache hit) 캐시에서 데이터를 읽어온다. 만약 찾고자 하는 데이터가 없다면(Cache miss) 애플리케이션은 직접 데이터베이스에 접근하여 찾고자하는 데이터를 가져오고 이를 캐시에 저장하는 과정을 거친다. 이처럼 찾고자 하는 데이터가 레디스에 없을 때에만, 레디스에 데이터가 저장되기 때문에 Lazy Loading이라고도 부른다.</p>
<p>&nbsp;</p>
<p>해당 구조의 장점은 레디스에 만약 문제가 생겨 접근이 불가능한 상황이 발생하더라도 바로 서비스 장애로 이어지지 않고 데이터베이스에서 데이터를 가지고 올 수 있다는 점이다. 하지만, 기존의 애플리케이션에서 레디스를 통해 데이터를 가져오는 연결이 매우 많았다면 모든 커넥션이 한꺼번에 원본 데이터베이스로 몰려 많은 부하가 발생하게 되고, 이로 인해 원본 데이터베이스의 응답이 느려지거나 리소스를 많이 차지하는 등 전체 애플리케이션의 성능에 영향을 미칠 수 있다.&nbsp;</p>
<blockquote><b>Cache Warming (캐시 워밍)</b><br />초기에는 데이터에 접근할 때, 캐시 미스가 빈번하게 일어나 데이터베이스에 재접근하는 과정을 통해 지연이 초래되어 성능에 영향을 미칠 수 있다. 따라서 미리 데이터베이스에서 캐시로 데이터를 밀어넣어주는 작업을 하기도 하는데, 이를 캐시 워밍(cache warming)이라고 한다.</blockquote>
<h4>2. Read Through</h4>
<p>Read Through 패턴은 캐시에서만 데이터를 읽어오는 방식이다.&nbsp;</p>
<p><figure class="imageblock alignCenter"><span><img height="151" src="https://blog.kakaocdn.net/dn/bCsDBX/btsNKuMbBIh/mATM9YaCKQcSgGRDknDOXk/img.png" width="532" /></span></figure>
</p>
<p>Cache miss가 발생할 경우, 데이터베이스에서 데이터를 조회하고 자체적으로 캐시에를 업데이트를 한 후, 데이터를 반환한다. 해당 방식은 읽기 작업은 전적으로 캐시에 의존하기 때문에 Redis 서버에 장애가 발생했을 경우, 서비스 전체 장애로 이어질 수 있다.</p>
<h3>쓰기 전략</h3>
<h4>1. Write Back(Behind)</h4>
<p>만약, 쓰기가 빈번하게 발생하는 서비스라면 Write Behind 방식을 고려해볼 수 있다. 데이터베이스에 대량의 쓰기 작업이 발생하면 이는 많은 디스크 I/O를 유발해 성능 저하가 발생할 수 있다.</p>
<p><figure class="imageblock alignCenter"><span><img height="321" src="https://blog.kakaocdn.net/dn/btF28c/btsNJTlnTSB/TfY6cfNaeUzqogWPL1AS81/img.png" width="432" /></span></figure>
</p>
<p>Write Back 방식은 먼저 데이터를 빠르게 접근할 수 있는 캐시에 데이터를 업데이트한 뒤, 이후에는 건수나 특정 시간 간격 등에 따라 비동기적으로 데이터베이스에 업데이트하는 것이다. 저장되는 데이터가 실시간으로 정확한 데이터가 아니어도 되는 경우 이 방법이 유용하다. 예를 들어 유튜브와 같은 스트리밍 사이트의 동영상 좋아요 수는 매번 실시간 집계가 필요하지 않다. 때문에 좋아요를 누른 데이터를 우선 레디스에 저장해둔 다음 5분 간격으로 이를 집계해 데이터베이스에 저장하는 과정을 거친다면, 데이터베이스의 성능을 향상시켜 애플리케이션의 성능도 향상시킬 수 있다. 하지만, 이 방법은 DB와 캐시 간의 데이터 불일치 문제가 생길 수 있으며 아까와 같은 상황에서 캐시에 문제가 생겨 데이터가 날아갈 경우 최대 5분 동안의 데이터가 날아갈 수 있다는 위험성이 존재한다.</p>
<h4>2. Write Through</h4>
<p><figure class="imageblock alignCenter"><span><img height="217" src="https://blog.kakaocdn.net/dn/cL3kUK/btsNKbF8bzE/xTFxrqsKvS4ueuK5IUFh6K/img.png" width="587" /></span></figure>
</p>
<p>Write Through 방식은 데이터베이스에 업데이터할 때마다 매번 캐시에도 데이터를 함께 업데이트 시키는 방식이다. 때문에 캐시는 항상 최신 데이터를 가지고 있어 데이터의 일관성을 유지할 수 있다는 장점이 있지만, 매 요청마다 각각의 저장소에 write 작업이 발생하기 때문에 시간이 많이 소요될 수 있다는 단점이 있다. 더하여 캐시는 다시 사용될 만한 데이터가 저장되는 것이 좋다. 자주 사용되지 않은 데이터가 Redis에 저장된다면 이는 리소스 낭비이기 때문에 TTL을 설정하여 데이터를 관리하는 것을 권장한다.</p>
<h4>3. Write-Around</h4>
<p><figure class="imageblock alignCenter"><span><img height="279" src="https://blog.kakaocdn.net/dn/bAsahU/btsNLoEHcb8/Eg7BIIZOoREUaElgxPZWZk/img.png" width="579" /></span></figure>
</p>
<p>Write-Around 패턴은 쓰기 작업 시에 데이터를 캐시에 저장하지 않고 DB에만 저장하는 것을 말한다. 즉, Write-Around는 쓰기 시 캐시를 갱신하지 않기 때문에, 기존에 캐시에 해당 데이터가 있었다면 오래된 값이 계속 남아 있게 되어 불일치가 발생할 수&nbsp;있다.</p>
<p><figure class="imageblock widthContent"><span><img height="314" src="https://blog.kakaocdn.net/dn/dGEgEv/btsNJ41AcgT/yIvTmLZnstSI01gZglY4rk/img.png" width="986" /></span></figure>
</p>
<p>위의 사진과 같이 Write Around 패턴은 주로 Look Aside, Read Through와 결합해서 사용된다고 한다.</p>