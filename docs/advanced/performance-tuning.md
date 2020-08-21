# Perfomance Tuning

## File to SQS

These performance metrics are for the
[Run Stream producer task](README.md#run-stream-producer-task).

### SQS ingestion using stream-producer json-to-sqs

- **Threads:**
  Specified in
  [docker-compose-stream-producer.yaml](../../resources/advanced/docker-compose-stream-producer.yaml)
  `SENZING_THREADS_PER_PRINT`.
- **mem_limit:**
  Specified in
  [ecs-params-stream-producer.yaml](../../resources/advanced/ecs-params-stream-producer.yaml)
  `task_definition.task_size.mem_limit`
- **cpu_limit:**
  Specified in
  [ecs-params-stream-producer.yaml](../../resources/advanced/ecs-params-stream-producer.yaml)
  `task_definition.task_size.cpu_limit`
- **Internal queue:**
  Specified in
  [docker-compose-stream-producer.yaml](../../resources/advanced/docker-compose-stream-producer.yaml)
  `SENZING_READ_QUEUE_MAXSIZE`.
- **Rate:** Messages queued per second.

| Threads | mem_limit | cpu_limit | Internal queue | :arrow_right: | Rate |
|--------:|----------:|----------:|---------------:|:-------------:|-----:|
|       4 |       4GB |       512 |             50 | :arrow_right: |  170 |
|       8 |       8GB |      1024 |             50 | :arrow_right: |  280 |
|       8 |      16GB |      2048 |             50 | :arrow_right: |  375 |
|      16 |      16GB |      2048 |             50 | :arrow_right: |  380 |
|      16 |      30GB |      4096 |             50 | :arrow_right: |  385 |
|      30 |      30GB |      4096 |             50 | :arrow_right: |  385 |

### SQS ingestion using stream-producer json-to-sqs-batch

| Threads | mem_limit | cpu_limit | Internal queue | :arrow_right: | Rate |
|--------:|----------:|----------:|---------------:|:-------------:|-----:|
|      16 |      16GB |      2048 |            200 | :arrow_right: | 1700 |
|      30 |      30GB |      4096 |            200 | :arrow_right: | 1750 |

## SQS to Senzing engine using stream-loader

1. Parameters
    1. **Threads:** Specified SENZING_THREADS_PER_PROCESS
    1. **mem:**
       Specified in
       [ecs-params-stream-loader.yaml](../../resources/advanced/ecs-params-stream-loader.yaml)
       `task_definition.task_size.mem_limit`.
    1. **cpu:**
       Specified in
       [ecs-params-stream-loader.yaml](../../resources/advanced/ecs-params-stream-loader.yaml)
       `task_definition.task_size.cpu_limit`.
1. Results
    1. **Scale:** Maximum tasks observed
    1. **Target:** Autoscale `TargetValue`
    1. **Rate:** Messages inserted per second.
    1. **Memory:** AWS MemoryUtilization metric for service.
    1. **CPU:** AWS CPUUtilization metric for service.
    1. **Cost:** Cost of ECS CPU and Memory per Million (not database)
       based on [AWS Fargate Pricing ](https://aws.amazon.com/fargate/pricing/)
        1. ((1M / Rate) * mem_limit_cost) + ((1M / Rate) * cpu_limit_cost)
    1. **ACU:** Maximum AWS Capacity Units allocated for database.
    1. **%CPU:** Percent database CPU used of "Database Capacity Units".
    1. **DB use:** ACU * %CPU.
    1. **dbIO:** Maximum database Write IOPS.
    1. **IOrec:** IOs per record
    1. **Date tested:** Date of test.
    1. **Run:** How many records were used in the test.
    1. **Fn** Footnotes

### Results

#### Scale of 1

| Scale | Threads | mem | cpu  | :arrow_right: | Rate | Memory | CPU | ACU | %CPU | DB use | dbIO | IOrec | Date tested | Run | Fn |
|------:|--------:|----:|-----:|:-------------:|-----:|-------:|----:|----:|-----:|-------:|-----:|------:|------------:|----:|:--:|
|     1 |       8 | 6GB | 1024 | :arrow_right: |   35 |    62% | 98% |   2 |  48% |   0.96 |  10K |   285 |  2020-08-14 |  1K |    |
|     1 |       8 | 8GB | 1024 | :arrow_right: |   33 |    46% | 89% |   2 |  35% |   0.70 |   9K |   272 |  2020-08-12 |  1K |    |
|     1 |       8 | 8GB | 2048 | :arrow_right: |   38 |    46% | 51% |   2 |  50% |   1.00 |  11K |   289 |  2020-08-12 |  1K |    |
|     1 |       8 | 8GB | 4096 | :arrow_right: |   49 |    48% | 30% |   8 |  64% |   5.12 |  14K |   285 |  2020-08-13 |  1K |    |
|     1 |      10 | 8GB | 1024 | :arrow_right: |   34 |    51% | 99% |   2 |  42% |   0.84 |  10K |   294 |  2020-08-12 |  1K |    |
|     1 |      10 | 8GB | 2048 | :arrow_right: |   45 |    51% | 60% |   4 |  50% |   2.00 |  13K |   288 |  2020-08-12 |  1K |    |
|     1 |      10 | 8GB | 4096 | :arrow_right: |   47 |    51% | 36% |  16 |  70% |  11.00 |  15K |   319 |  2020-08-13 |  1K |    |
|     1 |      12 | 8GB | 1024 | :arrow_right: |   35 |    55% | 99% |   2 |  36% |   0.72 |  10K |   285 |  2020-08-12 |  1K |    |
|     1 |      12 | 8GB | 2048 | :arrow_right: |   50 |    56% | 82% |  16 |  25% |   4.00 |  15K |   300 |  2020-08-13 |  1K |    |
|     1 |      12 | 8GB | 4096 | :arrow_right: |   53 |    56% | 34% |   8 |  70% |   5.60 |  15K |   283 |  2020-08-13 |  1K |    |
|     1 |      14 | 8GB | 2048 | :arrow_right: |   55 |    60% | 90% |   8 |  56% |   4.48 |  16K |   290 |  2020-08-13 |  1K |    |
|     1 |      14 | 8GB | 4096 | :arrow_right: |   54 |    60% | 65% |  16 |  42% |   6.72 |  25K |   463 |  2020-08-13 |  1K |    |
|     1 |      16 | 8GB | 4096 | :arrow_right: |   84 |    64% | 69% |  16 |  47% |   7.52 |  28K |   333 |  2020-08-13 |  1K |    |
|     1 |      18 | 8GB | 4096 | :arrow_right: |   77 |    67% | 52% |  16 |  38% |   6.08 |  21K |   272 |  2020-08-13 |  1K |    |
|     1 |      20 | 8GB | 4096 | :arrow_right: |   87 |    72% | 62% |  16 |  63% |  10.08 |  24K |   275 |  2020-08-13 |  2K |    |
|     1 |      22 | 8GB | 4096 | :arrow_right: |   98 |    75% | 78% |  16 |  64% |  10.24 |  30K |   306 |  2020-08-14 |  2K |    |
|     1 |      24 | 8GB | 4096 | :arrow_right: |   95 |    79% | 79% |  32 |  51% |  16.32 |  30K |   315 |  2020-08-14 |  2K |    |

#### Scale up

| Scale | Target | Threads | mem |  cpu | :arrow_right: | Rate | Memory | CPU | ACU | %CPU | DB use | dbIO | IOrec | Date tested | Run | Fn |
|------:|-------:|--------:|----:|-----:|:-------------:|-----:|-------:|----:|----:|-----:|-------:|-----:|------:|------------:|----:|:--:|
|    10 |  50.0% |       8 | 6GB | 1024 | :arrow_right: |  368 |    59% | 95% |  64 |  28% |  17.92 | 112K |   305 |  2020-08-14 |  1M |    |
|    27 |  50.0% |       8 | 8GB | 1024 | :arrow_right: |  495 |    43% | 88% |  64 |  52% |  33.28 | 145K |   292 |  2020-08-17 |  1M |    |
|    43 |  50.0% |       8 | 8GB | 1024 | :arrow_right: |  778 |    76% | 30% | 192 |  59% | 113.28 | 220K |   282 |  2020-08-17 |  1M |    |
|    60 |  50.0% |       8 | 8GB | 1024 | :arrow_right: | 1511 |    44% | 93% | 192 |  67% | 128.64 | 460K |   304 |  2020-08-17 |  2M |    |
|    99 |  50.0% |       8 | 8GB | 1024 | :arrow_right: | 1900 |    47% | 60% | 384 |  45% | 172.80 | 570K |   300 |  2020-08-18 |  5M |    |
|    99 |  20.0% |       8 | 8GB | 2048 | :arrow_right: |  830 |    47% | 30% | 384 |      |        | 420K |       |  2020-08-21 |  6M | [2](#2) |
|    25 |  50.0% |      24 | 8GB | 4096 | :arrow_right: | 2054 |    84% | 69% | 384 |  43% | 165.12 | 590K |   287 |  2020-08-20 |  5M |    |
|    50 |  50.0% |      24 | 8GB | 4096 | :arrow_right: | 1983 |    92% | 59% | 384 |  79% | 303.36 | 620K |   312 |  2020-08-20 |  5M | [1](#1) |

#### Footnotes

##### 1

1. Scale 50, Threads 24 was very unstable.
1. Of 7M run size, 600K were in dead-letter queue.

##### 2

1. ACU went up to 384 but came down to 192 before end of run.
1. of 5M, 2M ended up in dead-letter queue.

#### Cost

For cost analysis, see
[cost-calculations.pdf](cost-calculations.pdf).

### Archive results

| Threads | mem_limit | cpu_limit | :arrow_right: | Rate | Memory | CPU | DB CPU    | ACUs | Date tested |
|--------:|----------:|----------:|:-------------:|-----:|-------:|----:|----------:|-----:|------------:|
|       4 |      16GB |      2048 | :arrow_right: |   20 |    18% | 23% | 25% of 08 |      |             |
|       8 |      16GB |      2048 | :arrow_right: |   40 |    20% | 55% | 42% of 08 |      |             |
|      10 |      16GB |      2048 | :arrow_right: |   60 |    23% | 84% | 60% of 08 |      |             |
|      11 |      16GB |      2048 | :arrow_right: |   70 |    25% | 89% | 65% of 08 |      |             |
|      12 |       8GB |      2048 | :arrow_right: |   50 |    38% | 65% | 51% of 08 |      |             |
|      12 |      12GB |      4096 | :arrow_right: |   53 |    37% | 32% | 51% of 02 |      |  2020-08-10 |
|      12 |      16GB |      2048 | :arrow_right: |   70 |    27% | 93% | 63% of 08 |      |             |
|      16 |      30GB |      4096 | :arrow_right: |   69 |    16% | 42% | 61% of 08 |      |             |
|      20 |      30GB |      4096 | :arrow_right: |  103 |    18% | 74% | 59% of 16 |      |             |


## References

### AWS Fargate Pricing

Using
[AWS Fargate Pricing](https://aws.amazon.com/fargate/pricing/).

| Resource rate       | Pricing      |
|---------------------|:-------------|
| per vCPU per hour   | $0.04048     |
| per GB per hour     | $0.00182423  |
| per vCPU per second | $0.000011244 |
| per GB per second   | $0.000001235 |

Using
[Amazon Aurora Pricing](https://aws.amazon.com/rds/aurora/pricing/)

| Resource rate      | Pricing      |
|--------------------|:-------------|
| per ACU per hour   | $0.06        |
| per ACU per second | $0.000016667 |

### AWS Console

1. [cloudwatch](https://console.aws.amazon.com/cloudwatch/home)
    1. [log groups](https://console.aws.amazon.com/cloudwatch/home?#logsV2:log-groups)
1. [ecs](https://console.aws.amazon.com/ecs/home)
1. [rds](https://console.aws.amazon.com/rds/home?#databases:)

### AWS documentation

1. [Monitoring Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_monitoring.html)
1. [Invalid CPU or memory value specified](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-cpu-memory-error.html)
