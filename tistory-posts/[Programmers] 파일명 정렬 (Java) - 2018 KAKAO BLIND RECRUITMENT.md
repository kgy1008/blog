<h2 style="color: #000000; text-align: start;">문제 링크</h2>
<p><a href="https://school.programmers.co.kr/learn/courses/30/lessons/17686" rel="noopener&nbsp;noreferrer" target="_blank">https://school.programmers.co.kr/learn/courses/30/lessons/17686</a></p>
<figure contenteditable="false" id="og_1750863446037"><a href="https://school.programmers.co.kr/learn/courses/30/lessons/17686" rel="noopener" target="_blank">
<div class="og-image">&nbsp;</div>
<div class="og-text">
<p class="og-title">프로그래머스</p>
<p class="og-desc">SW개발자를 위한 평가, 교육의 Total Solution을 제공하는 개발자 성장을 위한 베이스캠프</p>
<p class="og-host">programmers.co.kr</p>
</div>
</a></figure>
<p style="color: #333333; text-align: start;">문제가 일단 굉장히 길다. 정규표현식을 사용해서 푸는 방법도 있는 것 같지만, 나는.. 정규표현식을 잘 사용하지 못하기 때문에 다른 방법으로 풀었다.</p>
<h2 style="color: #000000; text-align: start;">구현 과정</h2>
<p style="color: #333333; text-align: start;">여기서의 핵심은 파일명을 크게 HEAD, NUMBER, TAIL로 나눈다는 점과 이것을 기준으로 정렬을 한다는 점이다. Word라는 객체를 생성해서 정렬의 기준이 되는 HEAD와 NUMBER를 필드로 둔다면 쉽게 정렬을 할 수 있을 것 같아 초기에는 아래와 같이 내부 클래스를 작성했다. Word 객체에 head, number, tail 필드를 만들어 놓고 list에 word 객체를 담아 head와 number을 기준으로 정렬을 한 후, 배열에 각 필드들을 조합하여 최종적으로 결과를 반환하는 방식으로 구현했다.</p>
<pre class="java" id="code_1750864018124"><code>private class Word {
    String head;
    String number;
    String tail;
        
    Word(String head, String number, String tail) {
        this.head = head;
        this.number = number;
        this.tail = tail;
    }
}</code></pre>
<p>&nbsp;</p>
<h3>파일명 구성 요소 파싱</h3>
<p>문제를 해결하기 위한, 가장 핵심은 입력된 파일명에서 HEAD, NUMBER, TAIL 이렇게 3개의 부분으로 정확히 파싱하는 것이다. 문제 조건을 다시 차근차근 정리해보자.</p>
<p>&nbsp;</p>
<p>1. HEAD는 숫자가 아닌 '문자'로 이루어져있으며 최소 한 글자 이상이다.</p>
<p>2. NUMBER는 한 글자에서 최대 다섯 글자 사이의 '연속된' 숫자로 이루어져있으며 최소 숫자 하나 이상을 포함하고 있다.</p>
<p>3. TAIL은 HEAD와 NUMBER를 제외한 나머지 부분으로 문자와 숫자 모두 가능하다.</p>
<p>&nbsp;</p>
<p>여기서의 핵심은 HEAD는 무조건 문자로만 이루어져있다는 점과 HEAD와 NUMBER는 모든 파일명이 다 가지고 있는 구성요소라는 점이다. 때문에 입력된 파일명의 앞글자부터 끝까지 순회하면서, 숫자가 시작되기 직전까지는 HEAD, 숫자가 나오기 시작한 시점부터 숫자의 최대 길이 5가 되기 직전까지는 NUMBER 필드가 됨을 눈치챌 수 있다. 그 이후의 문자열은 substring함수를 통해 TAIL로 파싱이 가능하다.</p>
<pre class="java" id="code_1750866295263"><code>for (String file : files) {
    String head = "";
    String number = "";
    int i = 0;
            
    // HEAD 추출
    while (i &lt; file.length() &amp;&amp; !Character.isDigit(file.charAt(i))) {
        head += file.charAt(i);
        i++;
    }
            
    // NUMBER 추출 (최대 5자리)
    while (i &lt; file.length() &amp;&amp; Character.isDigit(file.charAt(i)) &amp;&amp; number.length() &lt; 5) {
        number += file.charAt(i);
        i++;
    }
            
    // TAIL 추출
    String tail = file.substring(i);
 
    list.add(new Word(head, number, tail));
}</code></pre>
<p>자바에서는 Character.isDigit(char c)라는 함수가 존재한다. 해당 메서드를 통해서 간편하게 주어진 문자 c가 숫자인지 판별할 수 있다.</p>
<h3>정렬</h3>
<p>정렬 조건도 찬찬히 분석하면 아래와 같다.</p>
<p>&nbsp;</p>
<p>1. 우선적으로 HEAD 부분을 기준으로 사전 순 정렬한다. 이때, 대소문자 구분은 하지 않는다.</p>
<p>2. HEAD 부분이 같을 경우, NUMBER 값을 기준으로 정렬한다. 이때, 숫자 앞의 0은 무시된다 (012와 12는 같은 값으로 처리)</p>
<p>3. 1,2번 과정을 거쳤음에도 동일한 우선순위를 가질 경우, 입력으로 주어진 순서를 유지한다.</p>
<p>&nbsp;</p>
<p>1번 조건의 경우 String 객체의 compareTo 메서드를 사용하면 사전식으로 간단하게 정렬을 할 수 있다.</p>
<p>2번 조건 역시 문자열을 정수형으로 변환하고 Integer.compare 메서드를 사용하면 숫자 값을 기준으로 비교가 가능하다. <span style="color: #456771;">(Java의 Integer.parseInt() 메서드는 문자열을 10진수 정수로 변환해주는 메서드이다. 때문에 앞자리에 0이 있어도 수학적으로 의미가 없기 때문에 문제 조건에서 원하는 정확한 숫자 값으로 반환할 수 있다.)</span></p>
<p>3번은 Java의 List.sort(Comparator) 또는 Collections.sort(...)을 사용하여 정렬을 하면 된다. 이 2개의 메서드는 기본적으로 안정 정렬(stable sort)으로 동작하기 때문에 정렬 기준이 같은 경우 입력된 원래 순서를 그대로 유지하면서 정렬이 수행한다.</p>
<p>&nbsp;</p>
<p>이를 구현한 코드는 아래와 같다.</p>
<pre class="java" id="code_1750867119673"><code>list.sort((o1, o2) -&gt; {
     String a1 = o1.head.toLowerCase();
     String a2 = o2.head.toLowerCase();
            
    int result = a1.compareTo(a2);
    if (result == 0) {
        int num1 = Integer.parseInt(o1.number);
        int num2 = Integer.parseInt(o2.number);
        return Integer.compare(num1, num2);
    }
    return result;
});</code></pre>
<p>&nbsp;</p>
<h3 style="color: #000000; text-align: start;">최종 코드</h3>
<pre class="java" id="code_1750864072939"><code>import java.util.*;

class Solution {
    public String[] solution(String[] files) {
        List&lt;Word&gt; list = new ArrayList&lt;&gt;();
        
        for (String file : files) {
            String head = "";
            String number = "";
            int i = 0;
            
            // HEAD 추출
            while (i &lt; file.length() &amp;&amp; !Character.isDigit(file.charAt(i))) {
                head += file.charAt(i);
                i++;
            }
            
            // NUMBER 추출 (최대 5자리)
            while (i &lt; file.length() &amp;&amp; Character.isDigit(file.charAt(i)) &amp;&amp; number.length() &lt; 5) {
                number += file.charAt(i);
                i++;
            }
            
            // TAIL 추출
            String tail = file.substring(i);
 
            list.add(new Word(head, number, tail));
        }
        
 
        list.sort((o1, o2) -&gt; {
            String a1 = o1.head.toLowerCase();
            String a2 = o2.head.toLowerCase();
            
            int result = a1.compareTo(a2);
            if (result == 0) {
                int num1 = Integer.parseInt(o1.number);
                int num2 = Integer.parseInt(o2.number);
                return Integer.compare(num1, num2);
            }
            return result;
        });
        
        String[] result = new String[list.size()];
        for (int i = 0; i &lt; list.size(); i++) {
            Word word = list.get(i);
            result[i] = word.head + word.number + word.tail;
        }
        return result;
    }
    
    private class Word {
        String head;
        String number;
        String tail;
        
        Word(String head, String number, String tail) {
            this.head = head;
            this.number = number;
            this.tail = tail;
        }
    }
}</code></pre>
<p style="color: #333333; text-align: start;">&nbsp;</p>
<p>제출을 하고 다시 코드를 보니 Word 객체 필드에 tail 대신, 원본 문자열을 저장하는 것이 더 나은 코드인 것 같다. 정렬의 기준이 되지도 않는 tail을 필드로 저장한 이유는 순수히 후에 결과를 반환할 때, 각 필드를 조합하여 원본 문자열로 복원하기 위함이었는데 처음부터 원본 파일명을 객체 필드로 들고 있다면, String 객체의 생성 횟수도 줄이면서 더 깔끔한 코드가 될 수 있을 것 같다.</p>
<h3>정규 표현식을 이용한 코드</h3>
<p>참고로 정규 표현식을 활용한 코드는 아래와 같다.</p>
<pre class="java" id="code_1750865571308"><code>import java.util.*;
import java.util.regex.*;

class Solution {
    private static final Pattern pattern = Pattern.compile("([^\\d]+)(\\d+)(.*)");
    
    public String[] solution(String[] files) {
        Arrays.sort(files, (o1,o2) -&gt; {
            String[] arr1 = parse(o1);
            String[] arr2 = parse(o2);
            int result = arr1[0].toUpperCase().compareTo(arr2[0].toUpperCase());
            if (result == 0) {
                int a = Integer.parseInt(arr1[1]);
                int b = Integer.parseInt(arr2[1]);
                return Integer.compare(a,b);
            }
            return result;
        });
        return files; 
    }
    
    public static String[] parse(String filename) {
        Matcher matcher = pattern.matcher(filename);
        if (matcher.matches()) {
            String head = matcher.group(1);
            String number = matcher.group(2);
            String tail = matcher.group(3);
            return new String[]{head, number, tail};
        }

        return new String[]{filename, "", ""};
    }
}</code></pre>