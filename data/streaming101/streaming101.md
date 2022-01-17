# Streaming 101

## Batch vs Streaming

- Batch

  - batch: 집단, 무리
  - 일괄 처리. 쌓여있는 데이터를 한 번에 처리
  - 일정 시간 동안 데이터 적재 -> 모은 데이터를 분석 시스템에 주입하여 분석
  - bounded data
  - streaming보다 먼저 생김

  ![batch](./batch.png)

- Streaming
  - stream: 강, 흐름
  - 지속적으로 흘러들어오는 데이터를 그때그때 처리
  - 데이터가 들어올 때마다 분석 시스템에 주입하여 분석
  - unbounded data

![streaming](./streaming_fixed_window.png)

## time concepts of data

- event time
  - the time at which events actually occurred
- processing time
  - the time at which events are observed in the system
- problem
  - `event time != processing time`
    - event가 늦게 도착할 수 있다
    - 언제 event가 도착할지 모른다.
    - event time 순서대로 데이터가 쌓이지 않는다.
    - (processing time - event time)가 어떤 숫자보다 작다고 보장할 수 없다. \
      (processing time - event time)는 일정하지 않고 다르다
  - why?
    - Shared resource limitations(network congestion, network partitions,shared CPU)
    - Software causes(distributed system logic, contention, ...)
    - environment: mobile, airplane mode, ....
  - ex)
    - mobile tetris game
    - game score sent to server every time people play the game
    - event time: 게임이 끝나서 점수가 결정되는 시점
    - processing time: 게임 점수가 서버에 도착한 시점

![time skew](./time_skew.png)

## Data processing patterns

### Batch

- Fixed Size Window
  - how
    - slice by fixed size window => set of small bounded data
    - repeated runs of batch engine on each of small bounded data
      ![batch fixed](./batch-fixed.png)
  - problem
    - what about late data?
    - need further mitigation \
      (delay the run until all events arrive, \
      re-run the entire batch)
- Session
  - session: period of activity(ex. 30 min of activity of a user)
    ![batch session](./batch-session.png)
  - problem
    - sessions are split across batches
    - need to increase batch size -> increased latency
    - additional logic to glue up sessions in previous runs -> increased complexity

### Streaming

- time agnostic(시간과 상관 없는)
  - ex) filtering
    ![filtering](./filtering.png)
- [approximation algorithm](#streaming-algorithm)
  - approximate Top-N, streaming K-means
  - low overhead, but complicated
  - usually processing-time based
  - if provable error bounds are dependent on data order, no use
    ![approximation algorithm](./approximation.png)
- windowing
  - 전체 범위를 일정 단위로 조각낸 것
    ![windowing patterns](./windowing_patterns.png)
    - fixed window(tumbling window)
    - sliding window
      - fixed window의 일반화 버전
      - defined by fixed length, and a fixed period
      - if period < length, window overlap
      - if period == length, it is fixed window
    - session
      - dynamic window
      - sequence of events terminated by a gap of inactivty greater than som etimeout
      - analyzing user behavior over time, by grouping related events
      - lengths are not predefined a prior
  - by processing time
    - buffer up incoming data into windows until some amount of processing time has passed
      ![processing_time](./streaming_fixed_window.png)
    - simple
    - window completeness is clear
    - when analyzing by the srouce as it is observed
    - problem
      - ex)
        mobile app
        distributed input source, when unhealthy
  - by event time
    - 이벤트 발생 시간 기준
      ![event_time](./streaming_event_time.png)
      ![session](./streaming_session.png)
    - buffering
    - extended window lifetime
    - solved with consistent persistent storage & decent in-memory caching
    - completeness
      - hard to know the end of the window
      - watermark
      - trigger

## Streaming concepts

> - X-axis: event time
> - Y-axis: processing time (on the Y axis).
> - thus, real time: bottom to top
> - circle: inputs. the number inside the circle: the value of the record.

What / Where / When / How

### What: transformation

- "What results are calculated?"
- ex) integer sum, filter, building histograms, training machine learning models
  ![transformation](./types_transformation.png)
  > composite: combination of transformations

### Where: window

- "Where in event time are results calculated?"
- fixed, sliding, and sessions
- time-agnostic

![windowing](./windowing_batch.png)

### When: Watermark, Trigger

- “When in processing time are results materialized?”
- when the input for the given window is complete?
- when to calculate?

#### watermark

- input completeness에 대한 보장
- watermark 이전에 발생한 데이터는 모두 도착했다 \
  watermark보다 작은 event time을 갖는 데이터는 모두 도착했다
- peferct watermark, heuristic watermark
  ![watermark](./watermark.png)
- drawbacks

  - too slow
    - takes longer time to get output
  - too fast
    - missing data
  - ex)
    - perfect : [12:02, 12:04)'s output comes around 12:09,
      while first input was around 12:04. \
    - heuristic : [12:02, 12:04)'s output comes around 12:07,
      but, missing data in [12:00, 12:02)

- temporal notions of input completeness in the event-time domain.
- input with event times less than E have been observed.
- it’s an assertion that no more data with event times less than E will ever be seen again.
- https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html#handling-late-data-and-watermarking

#### trigger

    - 언제 output을 계산할지
    - when output for a window should happen in processing time
    - type
      - Watermark progress
        - when watermark reaches the end of the window
      - Processing time progress
      - Element counts
      - Punctuations(other data-dependent triggers): specific record like EOF element, flush event, etc.
      - compoiste signals:
        - Repetitions: regular periodic updates
        - Conjunctions (logical AND)
        - Disjunctions (logical OR),
        - Sequences, which fire a progression of child triggers in a predefined order.

#### allowed lateness

- 언제 window를 종료할지
  > place a bound on how late any given record may be (relative to the watermark)
- inefficient/impossible to hold the intermediate state until the end of the window(no end, memory limit)
- event time 기준의 late data
- watermark는 A시점 이전 데이터는 다 도착했다. \
  allowed lateness는 watermark보다 X만큼 늦게 들어온 데이터까지만 취급한다.

## 참고자료 및 사진 출처

- https://www.oreilly.com/radar/the-world-beyond-batch-streaming-101
- https://www.oreilly.com/radar/the-world-beyond-batch-streaming-102

## todo

- lambda, kappa architecture

---

## Additional

- streaming algorithm(approximation) <a name="streaming-algorithm"></a>
  - constraint
    - memory constraint
    - (limited processing time per item)
    - small number of passes.
  - similar with online learning. \
    streaming algorithm can defer action until a group of points arrive, whereas online learning make decision as each point arrives
  - ex)
    - streaming linear regression
      - update the regression on each data input
    - streaming k-means
      - diverse implementation
      - ex) [pyspark implementation](https://databricks.com/blog/2015/01/28/introducing-streaming-k-means-in-spark-1-2.html)
        - mini-batch k-meanns with half-life & dying cluster
        - half-life: the time it takes before past data contributes to only one half of the current model. \
          ex) 0.5 batch half life -> 2 batches, 5 batch half life -> 10 batches to finish the change
        - dying cluster: invalid cluster
    - Frequency moments
      - The kth frequency moment of a set of frequencies $\mathbf {a}$ is defined as \
        ${\displaystyle F_{k}(\mathbf {a} )=\sum_{i=1}^{n}a_{i}^{k}}$
      - $F_0$: distinct count
      - $F_1$: sum
      - $F_k$: statistical properties
    - https://en.wikipedia.org/wiki/Streaming_algorithm
