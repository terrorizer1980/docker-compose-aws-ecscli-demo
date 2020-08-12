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
    1. **mem_limit:**
       Specified in
       [ecs-params-stream-loader.yaml](../../resources/advanced/ecs-params-stream-loader.yaml)
       `task_definition.task_size.mem_limit`.
    1. **cpu_limit:**
       Specified in
       [ecs-params-stream-loader.yaml](../../resources/advanced/ecs-params-stream-loader.yaml)
       `task_definition.task_size.cpu_limit`.
1. Results
    1. **Rate:** Messages inserted per second.
    1. **Memory:** AWS MemoryUtilization metric for service.
    1. **CPU:** AWS CPUUtilization metric for service.
    1. **Cost:** Cost of ECS CPU and Memory (not database)
        1. ((100K / Rate) * mem_limit_cost) + ((100K / Rate) * cpu_limit_cost)
    1. **ACU:** Maximum AWS Capacity Units allocated for database.
    1. **%CPU:** Percent database CPU used of "Database Capacity Units".
    1. **DB use:** ACU * %CPU.
    1. **dbIO:** Maximum database Write IOPS.

    1. **Date tested:** Date of test.

### Results

| Threads | mem_limit | cpu_limit | :arrow_right: | Rate | Memory | CPU | Cost | ACU | %CPU | DB use | dbIO | Date tested |
|--------:|----------:|----------:|:-------------:|-----:|-------:|----:|-----:|----:|-----:|-------:|-----:|-------------|
|       8 |       8GB |      1024 | :arrow_right: |      |        |     |      |     |      |        |      |             |
|      10 |       8GB |      1024 | :arrow_right: |      |        |     |      |     |      |        |      |             |
|      12 |       8GB |      1024 | :arrow_right: |   35 |    55% | 99% |      |  02 |  36% |   0.72 |  10K |  2020-08-12 |


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

### Did not work

| Threads | mem_limit | cpu_limit | :arrow_right: | Rate | Memory | CPU | DB CPU    | ACUs | Date tested |
|--------:|----------:|----------:|:-------------:|-----:|-------:|----:|----------:|-----:|------------:|
<tr> <td colspan=3> **Inputs** <td colspan=1> :arrow_right: <td colspan=5> **Results** <td colspan=1> </tr>
|      16 |      12GB |      4096 | :arrow_right: |   00 |    00% | 00% | 00% of 00 |      |             |
|      16 |      16GB |      2048 | :arrow_right: |   00 |    00% | 00% | 00% of 00 |      |             |


## References

### AWS Console

1. [cloudwatch](https://console.aws.amazon.com/cloudwatch/home)
    1. [log groups](https://console.aws.amazon.com/cloudwatch/home?#logsV2:log-groups)
1. [ecs](https://console.aws.amazon.com/ecs/home)
1. [rds](https://console.aws.amazon.com/rds/home?#databases:)

### AWS documentation

1. [Monitoring Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_monitoring.html)
1. [Invalid CPU or memory value specified](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-cpu-memory-error.html)
