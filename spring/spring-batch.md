# Spring Batch (Java) vs Spring Batch Plus (Kotlin)

이 문서는 **Java 11 기반 Spring Batch**와 **Kotlin 기반 Spring Batch Plus**를 사용하여 동일한 배치 Job을 구현한 예제와 구조적 차이를 설명합니다.

---

## 📌 공통 요구사항

- `chunkSize = 10`, `pageSize = 10`
- JPA 기반 페이징 처리
- Job과 Step은 각각 `@JobScope`, `@StepScope` 로 설정
- Reader, Processor, Writer 각각 별도 클래스로 구성
- Processor, Writer는 단순히 로그만 출력
- Reader:
  - Java: `AbstractPagingItemReader<T>` 상속
  - Kotlin: `ItemReader<List<T>>` 구현

---

## 📘 Java 11 + Spring Batch

### 🧩 JobConfig

```java
@Configuration
@RequiredArgsConstructor
public class SampleJobConfig {

    private final Step sampleStep;

    @Bean
    @JobScope
    public Job sampleJob(JobBuilderFactory jobBuilderFactory) {
        return jobBuilderFactory.get("sampleJob")
                .start(sampleStep)
                .build();
    }
}
```

### 🧩 StepConfig

```java
@Configuration
@RequiredArgsConstructor
public class SampleStepConfig {

    private final SampleReader reader;
    private final SampleProcessor processor;
    private final SampleWriter writer;

    @Bean
    public Step sampleStep(StepBuilderFactory stepBuilderFactory) {
        return stepBuilderFactory.get("sampleStep")
                .<SampleEntity, SampleEntity>chunk(10)
                .reader(reader)
                .processor(processor)
                .writer(writer)
                .build();
    }
}
```

### 🧩 Reader

```java
@Component
@StepScope
public class SampleReader extends AbstractPagingItemReader<SampleEntity> {

    private final SampleRepository repository;

    public SampleReader(SampleRepository repository) {
        this.repository = repository;
        setPageSize(10);
    }

    @Override
    protected void doReadPage() {
        if (results == null) results = new ArrayList<>();
        results.clear();
        PageRequest pageRequest = PageRequest.of(getPage(), getPageSize());
        results.addAll(repository.findAll(pageRequest).getContent());
    }
}
```

### 🧩 Processor

```java
@Component
@StepScope
public class SampleProcessor implements ItemProcessor<SampleEntity, SampleEntity> {
    @Override
    public SampleEntity process(SampleEntity item) {
        System.out.println("Processing: " + item);
        return item;
    }
}
```

### 🧩 Writer

```java
@Component
@StepScope
public class SampleWriter implements ItemWriter<SampleEntity> {
    @Override
    public void write(List<? extends SampleEntity> items) {
        items.forEach(item -> System.out.println("Writing: " + item));
    }
}
```

---

## 🟢 Kotlin + Spring Batch Plus

### 🧩 JobConfig

```kotlin
@Configuration
class SampleJobConfig(
    private val sampleStep: Step
) {
    @Bean
    @JobScope
    fun sampleJob(jobBuilderFactory: JobBuilderFactory): Job =
        jobBuilderFactory.get("sampleJob")
            .start(sampleStep)
            .build()
}
```

### 🧩 StepConfig

```kotlin
@Configuration
class SampleStepConfig(
    private val reader: SampleReader,
    private val processor: SampleProcessor,
    private val writer: SampleWriter
) {
    @Bean
    fun sampleStep(stepBuilderFactory: StepBuilderFactory): Step =
        stepBuilderFactory.get("sampleStep")
            .chunk<List<SampleEntity>, List<SampleEntity>>(10)
            .reader(reader)
            .processor(processor)
            .writer(writer)
            .build()
}
```

### 🧩 Reader

```kotlin
@StepScope
@Component
class SampleReader(
    private val repository: SampleRepository
) : ItemReader<List<SampleEntity>> {

    private var page = 0

    override fun read(): List<SampleEntity>? {
        val result = repository.findAll(PageRequest.of(page++, 10)).content
        return if (result.isEmpty()) null else result
    }
}
```

### 🧩 Processor

```kotlin
@StepScope
@Component
class SampleProcessor : ItemProcessor<List<SampleEntity>, List<SampleEntity>> {
    override fun process(items: List<SampleEntity>): List<SampleEntity> {
        items.forEach { println("Processing: $it") }
        return items
    }
}
```

### 🧩 Writer

```kotlin
@StepScope
@Component
class SampleWriter : ItemWriter<List<SampleEntity>> {
    override fun write(items: List<List<SampleEntity>>) {
        items.flatten().forEach { println("Writing: $it") }
    }
}
```

---

## 🔍 차이점 요약

| 항목 | Java (Spring Batch) | Kotlin (Spring Batch Plus) |
|------|---------------------|-----------------------------|
| Reader 상속/구현 | `AbstractPagingItemReader<T>` | `ItemReader<List<T>>` |
| Paging 처리 | 내부 상태 기반 (`getPage()`) | 외부에서 page 수동 증가 |
| Chunk 처리 타입 | `<T, T>` | `<List<T>, List<T>>` |
| 데이터 흐름 | 개별 엔티티 처리 | 리스트 단위 처리 |

---

## ✅ 결론

- Java의 `spring-batch`는 복잡하지만 견고한 표준 기반 구현에 적합하며, 세세한 설정 제어에 강점을 가집니다.
- Kotlin의 `spring-batch-plus`는 `List<T>` 단위의 Reader/Writer 흐름으로 **페이징 기반 대량 처리에 더 간단하고 효율적**입니다.
- 팀의 언어 선택, 데이터 처리 구조에 따라 적절한 방식 선택이 필요합니다.
---

## ⚙️ Processor 구조 차이 및 장단점

Spring Batch에서 가장 큰 차이점은 `Processor` 처리 단위입니다.  
Java에서는 개별 아이템 단위 (`T`), Kotlin(Spring Batch Plus)에서는 리스트 단위 (`List<T>`)로 처리합니다.

---

### ✅ Java 방식 (Item 단위 처리)

**특징**: `ItemProcessor<T, T>` — 각 아이템을 하나씩 순차적으로 처리

#### 장점
- **단순하고 명확한 흐름**: 한 번에 하나씩 처리하여 추적과 디버깅이 쉬움
- **Spring Batch의 표준 방식**: Retry, Skip 등 에러 제어 설정과 자연스럽게 호환됨
- **도메인 로직 분리 용이**: 엔티티 단위로 로직을 적용하기 쉬움

#### 단점
- **낮은 처리 효율**: 많은 양의 데이터를 처리할 때 반복 호출로 인한 오버헤드 발생
- **병렬 처리 어려움**: 리스트 기반 연산이 어려움
- **성능 최적화 한계**: 배치 단위 최적화가 힘듦

---

### ✅ Kotlin 방식 (List 단위 처리)

**특징**: `ItemProcessor<List<T>, List<T>>` — chunk 단위로 묶어서 한 번에 처리

#### 장점
- **고성능 처리 가능**: 호출 횟수 감소로 처리 속도 향상
- **집합 처리 유리**: `filter`, `map`, `groupBy` 등 컬렉션 연산 활용 가능
- **병렬 처리 확장성**: coroutine, parallelStream 등과 쉽게 결합 가능

#### 단점
- **에러 추적 어려움**: 문제가 생긴 개별 아이템 추적이 어려움
- **Spring Batch 기본 전략과 다름**: Skip, Retry 등은 별도 처리 필요
- **학습 난이도 상승**: Spring Batch의 표준에서 벗어난 구조
