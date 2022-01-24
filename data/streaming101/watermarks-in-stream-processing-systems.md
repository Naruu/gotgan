# Watermarks in stream processing systems

## What is watermark

- the temporal completeness of an out-oforder data stream
- lower bound of the event times of all unreceived events
- the lowest timestamp that may yet appear in
  a stream
- peferct watermark(conformant watermark), heuristic watermark(nonconformant watermark)
- formally, \
   for a node N of a dataflow graph G and a totally ordered time domain T , let I be the sequence of input elements arriving at N. \
   The arrival timing of these elements is given by a processing time function $p : I → T$\
  Each element is tagged with an event timestamp, given by the function $e : I → T$ \
  A watermark for N is a function $w : T → T$ satisfying the following properties.
  - Monotonicity: w is monotone. It must never “move backwards”.
  - Conformance: $for all i ∈ I, w(p(i)) < e(i)$ \ A conformant
    watermark must bound event times from below
  - Liveness: $w(t)$ has no upper bound.

## How watermark is generated

- making confront watermark is impractical
- we need to estimate the lag and generate heuristic watermark
- bounded lag
  - assume that the event will arrive no later than certain bounded constant
  - max event time ever seen + timeout delay
  - Apache Kafka streams, Apache Spark Structured Streaming
- timeout
  - when we first see the element with event time t, we wait for some constant duration and change the watermark to time t.
- stastical model
  - model the behavior of the event source, and adapt the watermark delay accordingly
  - ex)
    1. calculate a rolling histogram of the element lag
    2. fit it to a statisical moel(ex. gamma distribution)
    3. approximates the delay with the certain quantile
    4. re-evaluate the fit periodcially
  - Google Cloud Dataflow: histogram

> https://vldb.org/pvldb/vol14/p3135-begoli.pdf \
> page 5. Figure 1, 2

- custom logic
  - utilize input source’s unique characteristics
  - a conformant watermark generator for a Kafka topic whose static set of partitions are known to contain monotonically increasing timestamps,
  - Google Cloud Dataflow’s bespoke watermark generator for Google Pub/Sub

## How watermark is propagated

- data pipeline is multistaged, directed acyclic graph. \
  Each node has its watermark. \
  How should the downstream node's watermark decided?
- ex) min {input watermarks, the start times for all incomplete hourly
  windows buffered in state}
- use pattern-specific methods \
  (can not automatically infer watermark propagation straints)
  > probably depends on the type of source, opeation, etc?

## How Watermarks are Consumed

- triggering
- late data: data that has smaller event timestamp than the watermark(data that arrived too late)
- how to handle late data?

  - drop the data
    - result may not be correct
  - recompute the result
    - some framework suppports recomputation(Google Cloud Dataflow), some do not(Apache Kafka, Apache Spark)
    - developer should take care of it by themselves.
    ```java
    // Google Cloud DataFlow
    PCollection<KV<String, Integer>> scores = input
    .apply(Window.into(FixedWindows.of(Duration.standardMinutes(2)))
               .triggering(
                 AtWatermark()
                   .withEarlyFirings(AtPeriod(Duration.standardMinutes(1)))
                   .withLateFirings(AtCount(1)))
               .withAllowedLateness(Duration.standardMinutes(1))) // handle late data
    .apply(Sum.integersPerKey());
    ```

## Implementations(frameworks)

- comomon
  - distributed data stream processors
  - supports watermark based on event time proccessing semantics
- Flink
  - watermark is a metadata inside the datastream themselves
- Cloud Dataflow
  - watermark is handled in seperate aggregator node. Global control

# reference

Tyler Akidau, Edmon Begoli, Slava Chernyak, Fabian Hueske, Kathryn Knight, Kenneth Knowles, Daniel Mills, and Dan Sotolongo. 2021. Watermarks in stream processing systems: semantics and comparative analysis of Apache Flink and Google cloud dataflow. Proc. VLDB Endow. 14, 12 (July 2021), 3135–3147. DOI:https://doi.org/10.14778/3476311.3476389
