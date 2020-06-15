# docker-compose-aws-ecscli-demo

## Contents

1. [Prerequisites](#prerequisites)
    1. [Install ECS CLI](#install-ecs-cli)
    1. [Multi-factor authentication](#multi-factor-authentication)
    1. [Clone repository](#clone-repository)
1. [Tutorial](#tutorial)
    1. [Identify metadata](#identify-metadata)
    1. [Configure ECS CLI](#configure-ecs-cli)
    1. [Create cluster](#create-cluster)
    1. [View tasks](#view-tasks)
    1. [View services](#view-services)
1. [Cleanup](#cleanup)
    1. [Bring down task](#bring-down-task)
    1. [Bring down cluster](#bring-down-cluster)
    1. [Clean logs](#clean-logs)
    1. [Verify cleanup in AWS console](#verify-cleanup-in-aws-console)
1. [References](#references)

## Prerequisites

### Install ECS CLI

To install `ecs-cli`, follow steps at
[Configuring the Amazon ECS CLI](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_installation.html).

### Multi-factor authentication

:thinking: **Optional:** If multi-factor authentication is used to access AWS,
see [How to set AWS multi-factor authentication credentials](https://github.com/Senzing/knowledge-base/blob/master/HOWTO/set-aws-mfa-credentials.md).

### Clone repository

For more information on environment variables,
see [Environment Variables](https://github.com/Senzing/knowledge-base/blob/master/lists/environment-variables.md).

1. Set these environment variable values:

    ```console
    export GIT_ACCOUNT=senzing
    export GIT_REPOSITORY=docker-compose-aws-ecscli-demo
    export GIT_ACCOUNT_DIR=~/${GIT_ACCOUNT}.git
    export GIT_REPOSITORY_DIR="${GIT_ACCOUNT_DIR}/${GIT_REPOSITORY}"
    ```

1. Using the environment variables values just set, follow steps in [clone-repository](https://github.com/Senzing/knowledge-base/blob/master/HOWTO/clone-repository.md) to install the Git repository.

## Tutorial

### Identify metadata

#### AWS metadata

1. :pencil2: Set AWS metadata.
   Example:

    ```console
    export AWS_REGION=us-east-1
    export AWS_KEYPAIR=aws-default-key-pair
    ```

#### Identify project

1. :pencil2: Choose a prefix used in AWS object names.
   Example:

    ```console
    export AWS_PROJECT=project01
    ```

#### EULA

To use the Senzing code, you must agree to the End User License Agreement (EULA).

1. :warning: This step is intentionally tricky and not simply copy/paste.
   This ensures that you make a conscious effort to accept the EULA.
   Example:

    <pre>export SENZING_ACCEPT_EULA="&lt;the value from <a href="https://github.com/Senzing/knowledge-base/blob/master/lists/environment-variables.md#senzing_accept_eula">this link</a>&gt;"</pre>

#### Synthesize variables

1. Create additional environment variables.
   Example:

    ```console
    export SENZING_AWS_ECS_CLUSTER=${AWS_PROJECT}-cluster
    export SENZING_AWS_ECS_CLUSTER_CONFIG=${AWS_PROJECT}-config-name
    export SENZING_AWS_ECS_PARAMS_FILE=${GIT_REPOSITORY_DIR}/resources/beginner/ecs-params.yaml
    ```

### Configure ECS CLI

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [configure](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-configure.html).
   Example:

    ```console
    ecs-cli configure \
       --cluster ${SENZING_AWS_ECS_CLUSTER} \
       --config-name ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
       --default-launch-type EC2 \
       --region ${AWS_REGION}
    ```

1. :thinking: **Optional:** To view configuration values, see `~/.ecs/config`.

    ```console
    cat ~/.ecs/config
    ```

### Create cluster

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-up.html).
   Example:

    ```console
    ecs-cli up \
      --capability-iam \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --force \
      --instance-type t2.large \
      --keypair ${AWS_KEYPAIR} \
      --size 1
    ```

1. :thinking: **Optional:** View aspects of AWS ECS cluster in AWS console.
    1. [ecs](https://console.aws.amazon.com/ecs/home)
    1. [cloudformation](https://console.aws.amazon.com/cloudformation/home?#/stacks)
    1. [ec2](https://console.aws.amazon.com/ec2/v2/home)
        1. [auto scaling groups](https://console.aws.amazon.com/ec2autoscaling/home?#/details)
        1. [instances](https://console.aws.amazon.com/ec2/v2/home?#Instances)
        1. [launch configurations](https://console.aws.amazon.com/ec2/autoscaling/home?#LaunchConfigurations)

### Find security group ID

1. Find the AWS security group for the EC2 instance used in ECS.
   Run
   [aws](https://docs.aws.amazon.com/cli/latest/reference/index.html)
   [cloudformation](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/index.html)
   [list-stack-resources](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/list-stack-resources.html)
   Example:

    ```console
    export SENZING_AWS_EC2_SECURITY_GROUP=$( \
      aws cloudformation list-stack-resources \
        --stack-name amazon-ecs-cli-setup-${SENZING_AWS_ECS_CLUSTER} \
      | jq --raw-output ".StackResourceSummaries[] | select(.LogicalResourceId == \"EcsSecurityGroup\").PhysicalResourceId" \
    )
    ```

1.
1. :thinking: **Optional:** View security group ID.
   Example:

    ```console
    echo ${SENZING_AWS_EC2_SECURITY_GROUP}
    ```

### Open inbound ports

:warning: **Warning:** The following inbound port specifications are **wide open**.
Meaning they can be accessed from anywhere.
For demonstration purposes, this is fine.
For production purposes it is not fine.

1. Open inbound ports.
   Run
   [aws](https://docs.aws.amazon.com/cli/latest/reference/index.html)
   [ec2](https://docs.aws.amazon.com/cli/latest/reference/ec2/index.html)
   [authorize-security-group-ingress](https://docs.aws.amazon.com/cli/latest/reference/ec2/authorize-security-group-ingress.html).
   Example:

    ```console
    aws ec2 authorize-security-group-ingress \
      --group-id ${SENZING_AWS_EC2_SECURITY_GROUP} \
      --ip-permissions \
        IpProtocol=tcp,FromPort=5432,ToPort=5432,IpRanges='[{CidrIp=0.0.0.0/0,Description="PostgreSQL"}]'

    aws ec2 authorize-security-group-ingress \
      --group-id ${SENZING_AWS_EC2_SECURITY_GROUP} \
      --ip-permissions \
        IpProtocol=tcp,FromPort=5672,ToPort=5672,IpRanges='[{CidrIp=0.0.0.0/0,Description="RabbitMQ service"}]'

    aws ec2 authorize-security-group-ingress \
      --group-id ${SENZING_AWS_EC2_SECURITY_GROUP} \
      --ip-permissions \
        IpProtocol=tcp,FromPort=8250,ToPort=8250,IpRanges='[{CidrIp=0.0.0.0/0,Description="Senzing API server"}]'

    aws ec2 authorize-security-group-ingress \
      --group-id ${SENZING_AWS_EC2_SECURITY_GROUP} \
      --ip-permissions \
        IpProtocol=tcp,FromPort=8251,ToPort=8251,IpRanges='[{CidrIp=0.0.0.0/0,Description="Senzing Web App"}]'

    aws ec2 authorize-security-group-ingress \
      --group-id ${SENZING_AWS_EC2_SECURITY_GROUP} \
      --ip-permissions \
        IpProtocol=tcp,FromPort=8254,ToPort=8254,IpRanges='[{CidrIp=0.0.0.0/0,Description="Senzing X-Term"}]'

    aws ec2 authorize-security-group-ingress \
      --group-id ${SENZING_AWS_EC2_SECURITY_GROUP} \
      --ip-permissions \
        IpProtocol=tcp,FromPort=9171,ToPort=9171,IpRanges='[{CidrIp=0.0.0.0/0,Description="phpPgAdmin"}]'

    aws ec2 authorize-security-group-ingress \
      --group-id ${SENZING_AWS_EC2_SECURITY_GROUP} \
      --ip-permissions \
        IpProtocol=tcp,FromPort=9178,ToPort=9178,IpRanges='[{CidrIp=0.0.0.0/0,Description="Senzing Jupyter notebooks"}]'

    aws ec2 authorize-security-group-ingress \
      --group-id ${SENZING_AWS_EC2_SECURITY_GROUP} \
      --ip-permissions \
        IpProtocol=tcp,FromPort=15672,ToPort=15672,IpRanges='[{CidrIp=0.0.0.0/0,Description="RabbitMQ user interface"}]'
    ```

1. :thinking: **Optional:** View Security Group.
   Run
   [aws](https://docs.aws.amazon.com/cli/latest/reference/index.html)
   [ec2](https://docs.aws.amazon.com/cli/latest/reference/ec2/index.html)
   [describe-security-groups](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-security-groups.html).
   Example:

    ```console
    aws ec2 describe-security-groups \
      --group-ids ${SENZING_AWS_EC2_SECURITY_GROUP}
    ```

1. :thinking: **Optional:** View Security Group in AWS console.
    1. View [ec2 instances](https://console.aws.amazon.com/ec2/v2/home?#Instances)
    1. Choose "ECS instance" instance
    1. **Security groups:**, click on security group for ${SENZING_AWS_EC2_SECURITY_GROUP}.

### Install Senzing task

Install Senzing onto `/opt/senzing`.

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-up.html)
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/beginner/docker-compose-init.yaml \
      --project-name ${AWS_PROJECT}-project-name-init \
      up \
        --create-log-groups \
        --launch-type EC2
    ```

1. This task is a "job", not a long-running service.
   When the task state is `STOPPED`, the job has finished.

1. :thinking: **Optional:** View progress.
    1. [ecs](https://console.aws.amazon.com/ecs/home)
        1. Select ${SENZING_AWS_ECS_CLUSTER}
        1. Click "Update Cluster" to update information.
        1. Click "Tasks" tab.
        1. If task is seen, it is still "RUNNING".  Wait until task is complete.
    1. [ec2](https://console.aws.amazon.com/ec2/v2/home)
        1. [instances](https://console.aws.amazon.com/ec2/v2/home?#Instances)

### Create Postgres service

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
   to provision Postgres database service.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/beginner/docker-compose-postgres.yaml \
      --project-name ${AWS_PROJECT}-project-name-postgres \
      service up \
        --create-log-groups \
        --launch-type EC2
    ```

1. :thinking: **Optional:** View service definition.
   Example:

    ```console
    aws ecs describe-services \
      --cluster ${SENZING_AWS_ECS_CLUSTER} \
      --services ${AWS_PROJECT}-project-name-postgres
    ```

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address definition.
   This information will be used in subsequent steps.
   Example:

    ```console
    export SENZING_POSTGRES_HOST=$( \
      ecs-cli ps \
        --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      | grep  postgres \
      | awk '{print $3}' \
      | awk -F \: {'print $1'} \
    )
    ```

1. :thinking: **Optional:** View `SENZING_POSTGRES_HOST` value.
   Example:

    ```console
    echo $SENZING_POSTGRES_HOST
    ```

### Create Senzing database schema task

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-up.html)
   to create Senzing database schema.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/beginner/docker-compose-postgres-init.yaml \
      --project-name ${AWS_PROJECT}-project-name-postgres-init \
      up \
        --create-log-groups \
        --launch-type EC2
    ```

1. This task is a "job", not a long-running service.
   When the task state is `STOPPED`, the job has finished.

1. :thinking: **Optional:** View progress.
    1. [ecs](https://console.aws.amazon.com/ecs/home)
        1. Select ${SENZING_AWS_ECS_CLUSTER}
        1. Click "Update Cluster" to update information.
        1. Click "Tasks" tab.
        1. If task is seen, it is still "RUNNING".  Wait until task is complete.

### Create phpPgAdmin service

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
   to provision phpPgAdmin database client service.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/beginner/docker-compose-phppgadmin.yaml \
      --project-name ${AWS_PROJECT}-project-name-phppgadmin \
      service up \
        --create-log-groups \
        --launch-type EC2
    ```

1. View phpPgAdmin.
   Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address and port.
   Example:

    ```console
    ecs-cli ps \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
    | grep phppgadmin
    ```

### Create RabbitMQ service

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
   to provision RabbitMQ service.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/beginner/docker-compose-rabbitmq.yaml \
      --project-name ${AWS_PROJECT}-project-name-rabbitmq \
      service up \
        --create-log-groups \
        --launch-type EC2
    ```

1. :thinking: **Optional:** View service definition.
   Example:

    ```console
    aws ecs describe-services \
      --cluster ${SENZING_AWS_ECS_CLUSTER} \
      --services ${AWS_PROJECT}-project-name-rabbitmq
    ```

1. View RabbitMQ.
   Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address and port.
   This information will be used in subsequent steps.
   Example:

    ```console
    ecs-cli ps \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
    | grep rabbitmq
    ```

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address definition.
   Example:

    ```console
    export SENZING_RABBITMQ_HOST=$( \
      ecs-cli ps \
        --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      | grep  rabbitmq \
      | awk '{print $3}' \
      | awk -F \: {'print $1'} \
    )
    ```

1. Verify `SENZING_RABBITMQ_HOST`.
   Example:

    ```console
    echo $SENZING_RABBITMQ_HOST
    ```

### Create Mock data generator task

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-up.html)
   to send messages to RabbitMQ.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/beginner/docker-compose-mock-data-generator.yaml \
      --project-name ${AWS_PROJECT}-project-name-mock-data-generator \
      up \
        --create-log-groups \
        --launch-type EC2
    ```

1. This task is a "job", not a long-running service.
   When the task state is `STOPPED`, the job has finished.
   However, this is a long-running job.
   There is no need to wait for its completion.

### Run init-container task

Configure Senzing in `/etc/opt/senzing` and `/var/opt/senzing`.

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-up.html)
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/beginner/docker-compose-init-container.yaml \
      --project-name ${AWS_PROJECT}-project-name-init-container \
      up \
        --create-log-groups \
        --launch-type EC2
    ```

1. This task is a "job", not a long-running service.
   When the task state is `STOPPED`, the job has finished.

1. :thinking: **Optional:** View progress.
    1. [ecs](https://console.aws.amazon.com/ecs/home)
        1. Select ${SENZING_AWS_ECS_CLUSTER}
        1. Click "Update Cluster" to update information.
        1. Click "Tasks" tab.
        1. If task is seen, it is still "RUNNING".  Wait until task is complete.
    1. [ec2](https://console.aws.amazon.com/ec2/v2/home)
        1. [instances](https://console.aws.amazon.com/ec2/v2/home?#Instances)

### Create Stream loader service

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
   to provision stream-loader service.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/beginner/docker-compose-stream-loader.yaml \
      --project-name ${AWS_PROJECT}-project-name-stream-loader \
      service up \
        --create-log-groups \
        --launch-type EC2
    ```

1. :thinking: **Optional:** View service definition.
   Example:

    ```console
    aws ecs describe-services \
      --cluster ${SENZING_AWS_ECS_CLUSTER} \
      --services ${AWS_PROJECT}-project-name-rabbitmq
    ```

### Create API server service

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
   to provision Senzing API server service.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/beginner/docker-compose-apiserver.yaml \
      --project-name ${AWS_PROJECT}-project-name-apiserver \
      service up \
        --create-log-groups \
        --launch-type EC2
    ```

1. :thinking: **Optional:** View service definition.
   Example:

    ```console
    aws ecs describe-services \
      --cluster ${SENZING_AWS_ECS_CLUSTER} \
      --services ${AWS_PROJECT}-project-name-apiserver
    ```

1. View API server.
   Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address and port.
   This information will be used in subsequent steps.
   Example:

    ```console
    ecs-cli ps \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
    | grep apiserver
    ```

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address definition.
   Example:

    ```console
    export SENZING_IP_ADDRESS_APISERVER=$( \
      ecs-cli ps \
        --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      | grep  apiserver \
      | awk '{print $3}' \
      | awk -F \: {'print $1'} \
    )
    ```

1. Verify `SENZING_IP_ADDRESS_APISERVER`.
   Example:

    ```console
    echo $SENZING_IP_ADDRESS_APISERVER
    ```

1. :thinking: **Optional:** Verify Senzing API server is running.
   A JSON response should be given to the following `curl` request.
   Example:

    ```console
    curl -X GET "http://${SENZING_IP_ADDRESS_APISERVER}:8250/heartbeat"
    ```

### Create Web App service

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
   to provision Senzing Web App service.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/beginner/docker-compose-webapp.yaml \
      --project-name ${AWS_PROJECT}-project-name-webapp \
      service up \
        --create-log-groups \
        --launch-type EC2
    ```

1. :thinking: **Optional:** View service definition.
   Example:

    ```console
    aws ecs describe-services \
      --cluster ${SENZING_AWS_ECS_CLUSTER} \
      --services ${AWS_PROJECT}-project-name-webapp
    ```

1. View Web app.
   Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address and port.
   Example:

    ```console
    ecs-cli ps \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
    | grep webapp
    ```

### Create Jupyter notebook service

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
   to provision Senzing Web App service.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/beginner/docker-compose-jupyter.yaml \
      --project-name ${AWS_PROJECT}-project-name-jupyter \
      service up \
        --create-log-groups \
        --launch-type EC2
    ```

1. :thinking: **Optional:** View service definition.
   Example:

    ```console
    aws ecs describe-services \
      --cluster ${SENZING_AWS_ECS_CLUSTER} \
      --services ${AWS_PROJECT}-project-name-jupyter
    ```

1. View Jupyter.
   Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address and port.
   Example:

    ```console
    ecs-cli ps \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
    | grep jupyter
    ```

### Create Senzing X-Term service

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
   to provision Senzing X-Term service.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/beginner/docker-compose-xterm.yaml \
      --project-name ${AWS_PROJECT}-project-name-xterm \
      service up \
        --create-log-groups \
        --launch-type EC2
    ```

1. :thinking: **Optional:** View service definition.
   Example:

    ```console
    aws ecs describe-services \
      --cluster ${SENZING_AWS_ECS_CLUSTER} \
      --services ${AWS_PROJECT}-project-name-xterm
    ```

1. View X-Term.
   Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address and port.
   Example:

    ```console
    ecs-cli ps \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
    | grep xterm
    ```

### View tasks

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html).
   Example:

    ```console
    ecs-cli ps \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG}
    ```

1. View tasks in AWS Console:
    1. [ecs](https://console.aws.amazon.com/ecs/home)
        1. Select ${SENZING_AWS_ECS_CLUSTER}
        1. Click "Update Cluster" to update information.
        1. Click "Tasks" tab.
1. View logs:
   [cloudwatch](https://console.aws.amazon.com/cloudwatch/home)
   &gt; [log groups](https://console.aws.amazon.com/cloudwatch/home?#logsV2:log-groups)
   &gt; [senzing-docker-compose-aws-ecscli-demo](https://console.aws.amazon.com/cloudwatch/home?#logsV2:log-groups/log-group/senzing-docker-compose-aws-ecscli-demo)

### View services

1. To find IP addresses and ports, run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html).
   Example:

    ```console
    ecs-cli ps \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG}
    ```

   Open a web browser to the various `http://ip-address:port` locations.

1. [Senzing API in Swagger editor](http://editor.swagger.io/?url=https://raw.githubusercontent.com/Senzing/senzing-rest-api/master/senzing-rest-api.yaml)

## Cleanup

FIXME: Not complete.

### Bring down init task

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   down.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/beginner/docker-compose-init.yaml \
      --project-name ${AWS_PROJECT}-project-name-init \
      down \
        --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG}
    ```

1. This task is a "job", not a long-running service.
   When the task state is `STOPPED`, the job has finished.

### Bring down task

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   down.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/beginner/docker-compose.yaml \
      --project-name ${AWS_PROJECT}-project-name-main \
      down \
        --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG}
    ```

### Bring down cluster

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [down](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-down.html).
   Example:

    ```console
    ecs-cli down \
      --force \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG}
    ```

### Remove tasks


https://console.aws.amazon.com/ecs/home?region=us-east-1#/taskDefinitions

### Clean logs

1. Logs:
   [cloudwatch](https://console.aws.amazon.com/cloudwatch/home)
   &gt; [log groups](https://console.aws.amazon.com/cloudwatch/home?#logsV2:log-groups)
   &gt; [senzing-docker-compose-aws-ecscli-demo](https://console.aws.amazon.com/cloudwatch/home?#logsV2:log-groups/log-group/senzing-docker-compose-aws-ecscli-demo)



### Verify cleanup in AWS console

1. Verify in AWS Console:
    1. [ec2](https://console.aws.amazon.com/ec2/v2/home)
        1. [auto scaling groups](https://console.aws.amazon.com/ec2/autoscaling/home?#AutoScalingGroups)
        1. [instances](https://console.aws.amazon.com/ec2/v2/home?#Instances)
        1. [launch configurations](https://console.aws.amazon.com/ec2/autoscaling/home?#LaunchConfigurations)
        1. [network interfaces](https://console.aws.amazon.com/ec2/v2/home?#NIC)
        1. [security groups](https://console.aws.amazon.com/ec2/v2/home?#SecurityGroups)
    1. [cloudformation](https://console.aws.amazon.com/cloudformation/home?#/stacks)
    1. [cloudwatch](https://console.aws.amazon.com/cloudwatch/home)
        1. [log groups](https://console.aws.amazon.com/cloudwatch/home?#logsV2:log-groups)
    1. [ecs](https://console.aws.amazon.com/ecs/home)
        1. [clusters](https://console.aws.amazon.com/ecs/home?#/clusters)
        1. [task definitions](https://console.aws.amazon.com/ecs/home?#/taskDefinitions)

## References

1. [AWS](https://aws.amazon.com/)
   &gt; [Documentation](https://docs.aws.amazon.com/index.html)
   &gt; [Amazon ECS](https://docs.aws.amazon.com/ecs/index.html)
   &gt; [Developer Guide](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)
    1. [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
        1. [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
            1. [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
                1. [rm](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-rm.html)
                1. [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
            1. [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-up.html)
        1. [configure](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-configure.html)
        1. [down](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-down.html)
        1. [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
        1. [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-up.html)
1. [AWS](https://aws.amazon.com/)
   &gt; [Documentation](https://docs.aws.amazon.com/index.html)
   &gt; [AWS CLI](https://docs.aws.amazon.com/cli/)
    1. [aws](https://docs.aws.amazon.com/cli/latest/reference/index.html)
        1. [cloudformation](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/index.html)
            1. [list-stack-resources](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/list-stack-resources.html)
        1. [ec2](https://docs.aws.amazon.com/cli/latest/reference/ec2/index.html)
            1. [authorize-security-group-ingress](https://docs.aws.amazon.com/cli/latest/reference/ec2/authorize-security-group-ingress.html)
            1. [create-security-group](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-security-group.html)
            1. [describe-security-groups](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-security-groups.html)
        1. [ecs](https://docs.aws.amazon.com/cli/latest/reference/ecs/index.html)
            1. [describe-services](https://docs.aws.amazon.com/cli/latest/reference/ecs/describe-services.html)
1. AWS console
    1. [cloudformation](https://console.aws.amazon.com/cloudformation/home?#/stacks)
    1. [cloudwatch](https://console.aws.amazon.com/cloudwatch/home)
        1. [log groups](https://console.aws.amazon.com/cloudwatch/home?#logsV2:log-groups)
    1. [ec2](https://console.aws.amazon.com/ec2/v2/home)
        1. [auto scaling groups](https://console.aws.amazon.com/ec2autoscaling/home?#/details)
        1. [instances](https://console.aws.amazon.com/ec2/v2/home?#Instances)
        1. [launch configurations](https://console.aws.amazon.com/ec2/autoscaling/home?#LaunchConfigurations)
        1. [network interfaces](https://console.aws.amazon.com/ec2/v2/home?#NIC)
    1. [ecs](https://console.aws.amazon.com/ecs/home)
1. [Installing the Amazon ECS CLI](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_installation.html).
1. [Using the awslogs Log Driver](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_awslogs.html)
1. YAML file formats
    1. [Using Docker Compose File Syntax](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-parameters.html)
    1. [Using Amazon ECS Parameters](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-ecsparams.html)
1. [Tutorial: Creating a Cluster with an EC2 Task Using the Amazon ECS CLI](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-cli-tutorial-ec2.html)
