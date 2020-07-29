# Perfomance Tuning

## File to SQS

These performance metrics are for the
[Run Stream producer task](README.md#run-stream-producer-task).

### SQS ingestion using stream-producer json-to-sqs

- **cpu_limit:**
  Specified in
  [ecs-params-stream-producer.yaml](../../resources/advanced/ecs-params-stream-producer.yaml)
  `task_definition.task_size.cpu_limit`
- **mem_limit:**
  Specified in
  [ecs-params-stream-producer.yaml](../../resources/advanced/ecs-params-stream-producer.yaml)
  `task_definition.task_size.mem_limit`
- **Rate:** Messages queued per second.
- **Threads:**
  Specified in
  [docker-compose-stream-producer.yaml](../../resources/advanced/docker-compose-stream-producer.yaml)
  `SENZING_THREADS_PER_PROCESS`.
- **Internal queue:**
  Specified in
  [docker-compose-stream-producer.yaml](../../resources/advanced/docker-compose-stream-producer.yaml)
  `SENZING_READ_QUEUE_MAXSIZE`.

| Threads | mem_limit | cpu_limit | Internal queue | :arrow_right: | Rate |
|--------:|----------:|----------:|---------------:|:-------------:|-----:|
|       4 |       4GB |       512 |             50 | :arrow_right: |  170 |
|       8 |       8GB |      1024 |             50 | :arrow_right: |  280 |
|       8 |      16GB |      2048 |             50 | :arrow_right: |  375 |
|      16 |      16GB |      2048 |             50 | :arrow_right: |  380 |
|      16 |      30GB |      4096 |             50 | :arrow_right: |  385 |
|      30 |      30GB |      4096 |             50 | :arrow_right: |  385 |


### SQS ingestion using stream-producer json-to-sqs-batch

| Rate | Threads | mem_limit | cpu_limit | Internal queue | :arrow_right: |
|--------:|----------:|----------:|---------------:|:-------------:|-----:|
|      16 |      16GB |      2048 |            200 | :arrow_right: | 1700 |

## SQS to Senzing engine using stream-loader

- **cpu_limit:** Specified ecs-params.yaml `task_definition.task_size.cpu_limit`
- **CPUUtilization:** Percent CPU used
- **DB capacity:** Specified Database capacity
- **DB CPU:** Percent database CPU used
- **mem_limit:** Specified ecs-params.yaml `task_definition.task_size.mem_limit`
- **MemoryUtilization:** Percent memory used
- **Rate:** Messages inserted per second
- **Threads:** Specified SENZING_THREADS_PER_PROCESS

| Threads | mem_limit | cpu_limit | DB capacity | :arrow_right: | Rate | MemoryUtilization | CPUUtilization | DB CPU |
|--------:|----------:|----------:|------------:|:-------------:|-----:|------------------:|---------------:|-------:|
|       4 |      16GB |      2048 |           8 | :arrow_right: |   20 |               18% |            23% |    25% |
|       8 |      16GB |      2048 |           8 | :arrow_right: |   40 |               20% |            55% |    42% |
|      10 |      16GB |      2048 |           8 | :arrow_right: |   60 |               23% |            84% |    60% |

## References

### AWS Console

1. [cloudwatch](https://console.aws.amazon.com/cloudwatch/home)
    1. [log groups](https://console.aws.amazon.com/cloudwatch/home?#logsV2:log-groups)
1. [ecs](https://console.aws.amazon.com/ecs/home)
1. [rds](https://console.aws.amazon.com/rds/home?#databases:)

### AWS documentation

1. [Monitoring Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_monitoring.html)
1. [Invalid CPU or memory value specified](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-cpu-memory-error.html)
