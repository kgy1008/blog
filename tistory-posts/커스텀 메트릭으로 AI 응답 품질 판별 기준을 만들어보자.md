<h2>문제 상황</h2>
<p>Edukit 서비스는 교사들을 위해 AI를 활용하여 학생부 작성 업무를 돕는 서비스다. 하지만 이 서비스를 만드는 구성원은 현직 교사가 아닌 개발자 3명이었기 때문에, AI가 생성하는 응답의 품질을 명확하게 판단하기 어려웠다. 이를 보완하기 위해 설문조사를 통해 만족도를 수집했지만, 운영 서버에 새 기능을 배포할 때마다 질문을 초기화해야 하는 번거로움이 있었고, 사용자 수에 비해 응답률도 높지 않아 한계가 있었다. 이러한 상황이 반복되면서 서비스 개선 과정에서도 응답 품질에 대한 확신을 점점 잃게 되었다.<br />이때 멘토링을 통해&nbsp;<b>커스텀 메트릭</b>이라는 개념을 알게 되었고, 이를 활용하면 실제 사용 패턴을 기반으로 AI 응답 품질을 정량적으로 확인할 수 있지 않을까? 라는 생각을 하게 되었다. 본 글에서는 이러한 기능을 추가한 과정을 다뤄보고자 한다.</p>
<h2>해결 방안</h2>
<h3>가설 1</h3>
<blockquote>&lt;가설&gt;<br />AI 응답 품질이 좋을수록 API 실제 호출 대비 실제 응답 완성률이 높을 것이다.<br /><br />&lt;측정 방법&gt;<br />완성률 = (최종 저장 완료 수) / (AI 생성 요청 수) &times; 100%</blockquote>
<p><figure class="imageblock widthContent"><span><img height="1110" src="https://blog.kakaocdn.net/dn/G3i51/btsQIJzduuf/vf3IOResvRHQJp6GEAWpgK/img.png" width="794" /></span></figure>
</p>
<p>우리 서비스는 위와 같이 각 생기부 작성 항목 별로 학생의 특성을 입력하고 생성 버튼을 누르면 3가지 버전의 AI 생성 응답 값을 제공하고 있다.&nbsp;</p>
<p><figure class="imageblock widthContent"><span><img height="710" src="https://blog.kakaocdn.net/dn/blfoqS/btsQIBnDxRv/paYueOscOyPhmhDcB7K17k/img.png" width="2316" /></span><figcaption>실제 최종본을 저장하는 부분</figcaption>
</figure>
</p>
<p>그리고 이렇게 생성된 3가지 버전 중 마음에 드는 문장이나 버전을 선택해서 사용자가 해당 학생에 대한 최종본을 저장할 수 있는 플로우이다. 이렇게 AI를 이용하여 생성 요청을 보낸 호출 횟수 대비 실제 최종본을 저장하는 API 호출 응답 횟수가 적다면, AI 응답 품질이 좋지 않다는 가설을 세웠고 따라서 AI 생성 API 응답 호출과 생기부 최종 완료 저장본 개수를 커스텀 메트릭으로 추가하여 완성율을 계산한 뒤, 그라파나 대시보드에 시각화하였다.</p>
<h4>구현</h4>
<p>커스텀 메트릭은 비즈니스 로직과는 분리해서 구현하는 것이 유지보수 성과 코드 가독성 면에서 좋을 것이라 판단하였고 따라서 AOP를 활용하여 비즈니스 로직과 부가 로직이 명확히 구분되도록 분리하였다.</p>
<pre class="java" id="code_1758274070137"><code>@Component
@RequiredArgsConstructor
public class StudentRecordMetricsAspect {

    private final MeterRegistry meterRegistry;

    private static final String COMPLETION_METRIC = "student_record_completion_total";
    private static final String AI_GENERATION_REQUEST_METRIC = "student_record_ai_generation_requests_total";

   	public void recordCompletion(final StudentRecordType type, final String description) {
        if (isCompleted(type, description)) {
            meterRegistry.counter(COMPLETION_METRIC,
                    "type", type.name(), "action", "completion")
                    .increment();
        }
    }

    public void recordAIGenerationRequest(final StudentRecordType type) {
        meterRegistry.counter(AI_GENERATION_REQUEST_METRIC,
                "type", type.name(), "action", "ai_generation")
                .increment();
    }

    private boolean isCompleted(final StudentRecordType type, final String description) {
        if (description == null || description.trim().isEmpty()) {
            return false;
        }

        int minBytes = (type == StudentRecordType.SUBJECT) ? 1000 : 750;
        return description.getBytes(StandardCharsets.UTF_8).length &gt;= minBytes;
    }
}</code></pre>
<p>&nbsp;</p>
<p>단순히 사용자의 저장 시도만 카운트하는 방식으로는 실제로 의미 있는 지표를 얻기 어렵다고 판단하였다. 사용자가 AI가 생성한 결과를 단순히 확인만 하고 저장하지 않을 수도 있고, 의미 없는 내용을 입력한 뒤 저장하는 경우도 존재하기 때문이다.</p>
<p>이를 보완하기 위해 updateStudentRecord()와 같은 저장 메서드에 커스텀 메트릭 수집 로직을 적용하였다. 다만 저장된 최종본의 바이트 수가 기준 바이트 수의 절반을 넘지 못한다면, 유효하지 않은 저장으로 판단하여 지표에서 제외하였다. 구체적으로 일반 항목은 기준 바이트 수를 1500바이트로 설정하고, 이의 절반인 750바이트를 최소 유효 기준으로 삼았다. 반면 SUBJECT(세부능력특기사항) 항목의 경우 기본 바이트 수 제한이 2000바이트이므로, 기준을 1000바이트로 설정하였다.</p>
<p>최종적으로 완성률 계산식은 다음과 같이 정의된다.</p>
<blockquote>(student_record_completion_total / student_record_ai_generation_requests_total) * 100</blockquote>
<p>여기서 student_record_ai_generation_requests_total은 AI가 생성 요청을 받은 횟수이고, student_record_completion_total은 실제로 데이터베이스에 최종본을 저장한 횟수다. 예를 들어 AI 생성 요청이 총 100번 있었고, 그 중 70번은 사용자가 유요한 저장을 시도했으면 응답 완성률은 70%인 것이다.</p>
<h3>가설 2</h3>
<blockquote>&lt;가설&gt;<br />AI 품질이 좋을수록 재생성 요청 비율이 낮을 것이다.<br /><br />&lt;측정 방법&gt;<br />- 동일 recordId에 대한 연속 AI 생성 요청 횟수 <br />- 재생성률 = (재생성 요청 수) / (첫 생성 요청 수)</blockquote>
<p>사용자는 자신이 작성하고자 하는 페이지에 들어가 AI 요청을 한다. 이때, 해당 요청이 같은 recordId로&nbsp; 요청된다면, 이는 재요청을 보냈음을 의미한다. 여기서 만약 같은 생기부 항목에 대해 재요청 횟수가 많다면 이는 곧 AI 응답 품질이 마음에 들지 않았기 떄문이라는 가설을 세웠다.</p>
<h4>구현</h4>
<pre class="java" id="code_1758278469955"><code>public void recordFirstGeneration(final StudentRecordType type) {
        meterRegistry.counter(AI_FIRST_GENERATION_METRIC,
                "type", type.name(), "action", "first_generation")
                .increment();
    }

    public void recordRegeneration(final StudentRecordType type) {
        meterRegistry.counter(AI_REGENERATION_METRIC,
                "type", type.name(), "action", "regeneration")
                .increment();
    }
    
    @Around("@annotation(com.edukit.common.annotation.AIGenerationMetrics)")
    public Object collectAIGenerationMetrics(final ProceedingJoinPoint joinPoint) throws Throwable {
        Object[] args = joinPoint.getArgs();

        if (args.length &gt;= 3) {
            long recordId = (Long) args[1];
            StudentRecordType recordType = (StudentRecordType) args[2];

            try {
                boolean isFirstGeneration = generationTrackingService.isFirstGeneration(recordId);

                // 전체 AI 생성 요청 카운트
                metricsService.recordAIGenerationRequest(recordType);

                // 첫 생성 vs 재생성 구분 메트릭
                if (isFirstGeneration) {
                    metricsService.recordFirstGeneration(recordType);
                    log.debug("First generation request for recordId: {}", recordId);
                } else {
                    metricsService.recordRegeneration(recordType);
                    log.debug("Regeneration request for recordId: {}", recordId);
                }

            } catch (Exception e) {
                log.warn("Error collecting AI generation metrics for recordId: {}", recordId, e);
            }
        }
        return joinPoint.proceed();
    }</code></pre>
<h4>결과</h4>
<p><figure class="imageblock widthContent"><span><img height="1232" src="https://blog.kakaocdn.net/dn/bPsAWX/btsQJwfl1Hy/DuzzbU5jICtH6GdgIB8BGk/img.png" width="2364" /></span></figure>
</p>
<p>아래와 같이 각 생활기록부 항목 별로 재요청 비율을 시각화하였다. 이를 통해 재요청 비율이 상대적으로 높은 세부능력특기사항에 대한 AI 응답 품질이 떨어짐을 파악할 수 있고 이를 기반으로 AI 프롬프팅을 고도화하거나 개선점을 고민하는 등 서비스를 발전시킬 수 있는 정량적인 지표로 활용할 수 있을 것이라 판단된다.</p>