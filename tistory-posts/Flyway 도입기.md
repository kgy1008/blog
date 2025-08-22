<h2>도입한 계기</h2>
<p>프로젝트를 진행하면서 데이터베이스 스키마 변경이 빈번하게 발생했다. 1차 MVP를 배포한 이후에는 사용자 피드백에 따라 새로운 테이블이 추가되었고, 개발 도중 기획이 바뀌면서 기존의 테이블 연관관계가 끊어지는 경우도 있었다. 시간이 지날수록 변경 범위가 커지고 데이터 구조 자체가 자주 달라지다 보니, 매번 데이터베이스에 직접 접속해 쿼리를 작성해 수정하는 과정이 번거롭게 느껴졌다. 게다가 코드만 수정하고 DB 스키마를 반영하지 않아 에러가 발생하는 일도 잦았다. 이런 경험을 겪으며 <b>&ldquo;DB 스키마도 Git처럼 형상 관리할 수 없을까?&rdquo;</b>라는 고민을 하게 되었고, 그 과정에서 Flyway를 알게 되어 프로젝트에 도입하게 되었다.</p>
<h2>Flyway란?</h2>
<p>Flyway는 오픈소스 데이터베이스 마이그레이션 툴이다. 즉, 데이터베이스 변경 사항을 추적하고, 업데이트나 롤백을 보다 쉽게 할 수 있도록 도와주는 도구이다.</p>
<h3>Flyway 동작 방식</h3>
<p>Flyway의 마이그레이션은 <b>버전 번호</b>를 기준으로 순차적으로 실행된다.</p>
<p><figure class="imageblock widthContent"><span><img height="228" src="https://blog.kakaocdn.net/dn/nTvBU/btsP1QT1mgH/hPaZxIsc1L7IL1cRKPB5c1/img.png" width="833" /></span></figure>
</p>
<p>애플리케이션이 구동될 때 Flyway가 마이그레이션 파일을 자동으로 반영한다. 이때 <b>JPA Entity 클래스와 마이그레이션 파일을 반드시 함께 작성해야 하며</b>, 둘 중 하나라도 누락되면 애플리케이션이 정상적으로 구동되지 않는다.</p>
<p><figure class="imageblock widthContent"><span><img height="452" src="https://blog.kakaocdn.net/dn/Z1Uzg/btsP0mGsL9q/rETdDgbteCBTb5t2GmMVA0/img.png" width="1112" /></span></figure>
</p>
<p>이때, 마이그레이션 파일의 이름은 Flyway 규칙을 따라야 한다. 기본 규칙은 위의 사진(공식문서)과 같다.</p>
<p>간단히 정리하면, <code>V {버전번호}\_{설명}.sql</code> 형식이다.</p>
<h3>baseline-on-migrate 설정</h3>
<p>Flyway는 기본적으로 마이그레이션 스크립트를 순서대로 모두 적용한다. 따라서 스크립트에 작성한 내용이 이미 데이터베이스에 반영되어 있는 경우 에러가 발생할 수 있다.</p>
<p>이런 상황에서는 baseline-on-migrate 설정을 활용할 수 있다.</p>
<ul>
<li>해당 옵션을 false로 두면, 이전에 적용된 스크립트와 충돌이 발생하여 오류가 날 수 있다.</li>
<li>반대로 true로 설정하면, Flyway가 자동으로 현재 데이터베이스에 존재하는 마이그레이션 스크립트 중 가장 최신 버전을 기준으로 삼는다.</li>
</ul>
<p>즉, baseline-on-migrate = true로 두면 이미 반영된 스크립트는 기준 버전으로 처리하고, 그 이후의 새로운 마이그레이션 스크립트만 실행하게 된다. 이를 통해 불필요한 충돌을 방지할 수 있다.</p>
<h3>flyway_schema_history 테이블</h3>
<p>flyway를 적용한 채로 코드를 실행하게 되면, 아래와 같은 테이블이 자동으로 생성된다.</p>
<p><figure class="imageblock widthContent"><span><img height="510" src="https://blog.kakaocdn.net/dn/cMdMgt/btsP1wIgvld/V3nKxc90DYIUbBsd5ZkMj0/img.png" width="2246" /></span></figure>
</p>
<p>flyway_schema_history 테이블은 Flyway가 <b>데이터베이스 마이그레이션을 추적하고 관리하기 위한 메타데이터 테이블</b>이다.<br />여기에는 각 마이그레이션 스크립트의 <b>버전, 실행 여부, 실행 상태</b> 등이 기록된다. 이를 통해 Flyway는 데이터베이스에 어떤 마이그레이션이 이미 적용되었는지 추적하고, 새로운 스크립트 실행 시 <b>적용된 스크립트는 건너뛰고 순차적으로 실행</b>할 수 있다.</p>
<p>만약 flyway_schema_history 테이블이 없는 상태라면, Flyway는 <b>SQL 파일과 실제 데이터베이스 스키마를 비교</b>하여 현재 버전을 추론한다. 그리고 아직 실행되지 않은 버전의 SQL 파일이 있으면 이를 실행하여 데이터베이스를 업데이트한다.</p>
<h4>이미 생성된 마이그레이션 스크립트를 다시 실행하기</h4>
<p style="background-color: #ffffff; color: #353638; text-align: left;">특정 버전의 마이그레이션 스크립트를 다시 실행하고 싶다면, <b>flyway_schema_history</b> 테이블에서 해당 버전의 레코드를 삭제하면 된다. 예를 들어, V4부터 다시 실행하고 싶다면 flyway_schema_history 테이블에서 V4에 해당하는 레코드를 삭제한다. 그러면 Flyway는 데이터베이스에 저장된 버전 정보와 flyway_schema_history 테이블의 레코드를 비교하여 V4 마이그레이션 스크립트를 다시 실행하게 된다.</p>
<p>단, 이 작업은 반드시 주의해야 한다. 레코드를 삭제하면 해당 마이그레이션 스크립트가 실제로 적용한 데이터베이스 변경 사항은 롤백되지 않기 때문이다. 따라서 데이터 유실이나 불일치가 발생할 수 있다.</p>
<p>따라서 <b>데이터베이스 상태와 마이그레이션 스크립트의 내용이 일치하도록 사전에 적절한 조치를 취한 후</b> 해당 레코드를 삭제하는 것이 안전하다.</p>
<blockquote>⚠️ 주의<br />운영 환경에서는 이 방식이 예기치 못한 문제를 유발할 수 있으므로 가급적 권장하지 않는다. 운영 환경에서는 새로운 마이그레이션 스크립트를 작성하여 적용하는 방법을 우선적으로 고려하는 것이 좋다.</blockquote>
<h4><span style="background-color: #ffffff; color: #000000; text-align: left;">이미 실행된 버전의 sql 파일을 수정해도 될까?</span></h4>
<p style="background-color: #ffffff; color: #353638; text-align: left;">방금 실행한 V2 버전의 파일에서 쿼리를 잘못 작성했다고 가정해보자. &ldquo;그럼 그냥 V2 파일을 슬쩍 수정하면 되지 않을까?&rdquo; <b>절대 안된다! </b>Flyway의 핵심 규칙은 <b>버전은 오직 증가만 한다</b>는 것이다.&nbsp; 절대 이미 적용된 <b>파일들을 수정해서는 안 된다. </b><span style="color: #333333; text-align: start;">잘못된 부분을 고치려면<span>&nbsp;</span></span><b><b>다음 버전에서 수정하는 방식</b>으로 진행해야 한다.</b></p>
<p>앞으로 있을 모든 스키마 변화 역시 <b>새로운 버전 파일에 기록</b>해야 한다는 점을 꼭 기억하자.</p>