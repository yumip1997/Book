## List 구현체와 순회방식에 따른 속도 측정
List 인터페이스는 데이터를 핸들링 하는 데 사용되는 대표적인 자료구조입니다.

List의 대표적인 구현체는 ArrayList와 LinkedList가 있고, 이를 핸들링 하기 위한 방법으로 반복문이 사용됩니다.

List의 어떤 구현체와 어떤 순회 방식이 궁합이 좋은지 JMH을 통해 속도를 측정해보았습니다.

 

## 속도 측정

### ArrayList - 전통 for문 vs 향상된 for문
결과 : 전통 for문이 빠릅니다.

```
Benchmark                    Mode  Cnt  Score   Error  Units
LoopTest.enhancedForLoop     avgt   10  0.241 ± 0.007  ms/op
LoopTest.traditionalForLoop  avgt   10  0.177 ± 0.014  ms/op
```

이유 :  
ArrayList 순회 시 전통적 for문의 요소 접근 방식이 향상된 for문보다 더 빠르기 때문입니다. 전통적 for문의 경우 인덱스 기반으로 요소에 바로 접근하고, 향상된 for문의 경우 내부적으로 반복자를 사용하여 요소에 접근합니다.

향상된 for문의 경우 실질적으로는 다음과 같이 동작합니다.

```java
Iterator<Integer> iterator = integerList.iterator();
while (iterator.hasNext()){
	sum += iterator.next();
}
```
바로 get하느냐 hasNext를 체크하고나서 next 수행 후 요소에 접근하냐가 속도의 미묘한 차이를 줄 수 있다고 볼 수 있습니다.

결론 :  
ArrayList 순회 시, 전통적 for문을 사용하는 것이 가장 빠르지만 그 차이는 미미하다고 할 수 있습니다. 따라서 코드의 가독성과 간결성을 위해 향상된 for문을 사용하는 것이 좋습니다. 

소스 :  

```java
@State(Scope.Thread)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
public class LoopTest {

    final int LOOP_COUNT = 100000;
    final List<Integer> integerList = new ArrayList<>();
    int sum = 0;

    @Setup
    public void init(){
        for(int i=0;i<LOOP_COUNT;i++){
            integerList.add(i);
        }
    }

    @Benchmark
    public void traditionalForLoop(){
        int size = integerList.size();
        for(int i=0;i<size;i++){
            sum += integerList.get(i);
        }
    }

    @Benchmark
    public void enhancedForLoop(){
        for (Integer element : integerList) {
            sum += element;
        }
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(LoopTest.class.getSimpleName())
                .warmupIterations(10)           // 사전 테스트 횟수
                .measurementIterations(10)      // 실제 측정 횟수
                .forks(1)                      
                .build();

        new Runner(opt).run();
    }
}
 

 ```

### LinkedList - 전통 for문 vs 향상된 for문
결과 : 향상된 for문이 훨씬 빠릅니다.

```
Benchmark                    Mode  Cnt     Score     Error  Units
LoopTest.enhancedForLoop     avgt   10     1.015 ±   0.119  ms/op
LoopTest.traditionalForLoop  avgt   10  9541.285 ± 828.537  ms/op
```

이유 :  
LinkedList 경우 전통 for문으로 순회할 때 매번 첫 번째 요소부터 탐색을 시작하기 때문입니다. 예를 들어, integerList.get(i) 수행 시, 첫 번쨰 노드부터 탐색을 시작하여 i번째 노드를 찾습니다 하지만 향상된 for문의 경우 내부적으로 iterator를 사용하여 순차적으로 다음 요소로 이동하면서 순회하기에 첫 번쨰 요소부터 순회하지 않습니다.

 
결론 :  
LinkedList 순회 시, 향상된 for문을 사용하는 것이 좋습니다.

소스 :

```java
@State(Scope.Thread)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
public class LoopTest {

    final int LOOP_COUNT = 100000;
    final List<Integer> integerList = new LinkedList<>();
    int sum = 0;

    @Setup
    public void init(){
        for(int i=0;i<LOOP_COUNT;i++){
            integerList.add(i);
        }
    }

    @Benchmark
    public void traditionalForLoop(){
        int size = integerList.size();
        for(int i=0;i<size;i++){
            sum += integerList.get(i);
        }
    }

    @Benchmark
    public void enhancedForLoop(){
        for (Integer element : integerList) {
            sum += element;
        }
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(LoopTest.class.getSimpleName())
                .warmupIterations(10)           // 사전 테스트 횟수
                .measurementIterations(10)      // 실제 측정 횟수
                .forks(1)
                .build();

        new Runner(opt).run();
    }
}

```

### 최종 결론
1. ArrayList 순회 시, 가독성이 중요하다면 향상된 for문을 사용하자.

2. LinkedList 순회 시, 향상된 for문을 사용하자.