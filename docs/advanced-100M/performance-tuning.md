# Perfomance Tuning

## SQS to Senzing engine using stream-loader

1. Parameters
    1. **Scale:** Number of stream-loader containers. `SENZING_STREAM_LOADER_SCALE`
    1. **Threads:** Specified `SENZING_THREADS_PER_PROCESS`
    1. **mem:**
       Specified in
       [ecs-params-stream-loader.yaml](../../resources/advanced/ecs-params-stream-loader.yaml)
       `task_definition.task_size.mem_limit`.
    1. **cpu:**
       Specified in
       [ecs-params-stream-loader.yaml](../../resources/advanced/ecs-params-stream-loader.yaml)
       `task_definition.task_size.cpu_limit`.
    1. **dbType:** SENZING_AWS_DB_INSTANCE_CLASS
1. Results
    1. **Rate:** Messages inserted per second.
    1. **Memory:** AWS MemoryUtilization metric for service.
    1. **CPU:** AWS CPUUtilization metric for service.
    1. **dbCPU:** Percent database CPU used.
    1. **dbIO:** Maximum database Write IOPS.
    1. **IOrec:** IOs per record
    1. **Date tested:** Date of test.
    1. **Run:** How many records were used in the test.
    1. **Fn** Footnotes

### Results

#### Scale up

| Scale | Threads | mem |  cpu | dbType         | :arrow_right: | Rate | Memory | CPU | dbCPU | dbIO | IOrec | Date tested |  Run | Fn |
|------:|--------:|----:|-----:|----------------|:-------------:|-----:|-------:|----:|------:|-----:|------:|------------:|-----:|:--:|
|    12 |       8 | 8GB | 4096 | db.r5.16xlarge | :arrow_right: | 1137 |    80% | 76% |   39% | 406K |   357 |  2020-08-28 | 300K |    |

#### Footnotes

##### 1
