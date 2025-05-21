# Batch 용어 설명
### Job 
- 독립적으로 실행할 수 있는 고유하며 순서가 지정된 스텝의 목록
- 애플리케이션 실행시 Job으로 인식되는 bean들이 자동으로 실행된다.
- 1개 이상의 Step을 포함하여 원하는 동작을 실행시킬 수 있다
- 배치 처리 과정 중 전체 계층의 최상단에 위치도

### Step
- job의 구성요소로 자체적인 입력/출력/처리를 가질 수 있다.
- ==tasklet 또는 Chunk==기반 처리를 포함하여 step안에서 수행될 기능들을 명시할 수 있다.
- 트렌젝션은 step내에서 이루어진다. 때문에, 독립되도록 의도적으로 설계된 것이다.

### tasklet
- Step의 작업 단위를 Tasklet으로 정의
- 주로 간단한 작업(단일 데이터 처리, 파일 삭제 등)에 적합하다.

``` java
@Component
public class SimpleTasklet implements Tasklet {
    @Override
    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
        System.out.println("Tasklet 방식으로 작업 수행");
        return RepeatStatus.FINISHED;
    }
}

@Bean
public Step step1(StepBuilderFactory stepBuilderFactory) {
    return stepBuilderFactory.get("step1")
        .tasklet(new SimpleTasklet())
        .build();
}

@Bean
public Job job(JobBuilderFactory jobBuilderFactory, Step step1) {
    return jobBuilderFactory.get("job")
        .start(step1)
        .build();
}

```


### Chunk
- 대량 데이터를 일정 크기(chunk)로 나누어 처리한다.
- Reader / Processor / Writer로 구성된다.
``` java
@Bean
public FlatFileItemReader<String> reader() {
    return new FlatFileItemReaderBuilder<String>()
        .name("fileReader")
        .resource(new ClassPathResource("input.txt"))
        .lineMapper(new DefaultLineMapper<String>() {
            {
                setLineTokenizer(new DelimitedLineTokenizer());
                setFieldSetMapper(new PassThroughFieldSetMapper());
            }
        })
        .build();
}

@Bean
public ItemProcessor<String, String> processor() {
    return item -> "Processed " + item;
}

@Bean
public FlatFileItemWriter<String> writer() {
    return new FlatFileItemWriterBuilder<String>()
        .name("fileWriter")
        .resource(new FileSystemResource("output.txt"))
        .lineAggregator(new PassThroughLineAggregator<>())
        .build();
}

@Bean
public Step step(StepBuilderFactory stepBuilderFactory) {
    return stepBuilderFactory.get("step")
        .<String, String>chunk(10)
        .reader(reader())
        .processor(processor())
        .writer(writer())
        .build();
}

@Bean
public Job job(JobBuilderFactory jobBuilderFactory, Step step) {
    return jobBuilderFactory.get("job")
        .start(step)
        .build();
}

```

### Job Listener
- 잡 리스너를 이용해서 스프링batch 생명주기의 여러로직을 추가할 수 있다.
ex) beforeJob , afterJob 등등

# hfbatJobScheduler(스케줄러)가 Job으로 등록되어있는 녀석들을 순차적으로 실행될할 때 , 누가 트리거 역할을 하는지?

=> 스케줄러는 Jenkins에서 SSH스크립트를 통해 주기적으로 실행한다. Controller에 POST주소가 맵핑되어있는 이유는 테스트로 직접 실행하기 위함이다.

---