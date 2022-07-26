# Kafka stream processing

- This project deploys a simple Lambda-based stream-processing app.
- The application uses 2 [Lambda functions](https://docs.stacktape.com/resources/lambda-functions/),
  [Upstash Kafka topic](https://docs.stacktape.com/resources/upstash-kafka-topics/) and
  [HTTP API Gateway](https://docs.stacktape.com/resources/http-api-gateways/). The first Lambda function is triggered by
  an HTTP request. It writes the message from the HTTP request to the Kafka topic. The Kafka topic batches the messages.
  The second lambda function processes the batched messages.

## Pricing

- The infrastructure required for this application uses exclusively "serverless", pay-per-use infrastructure. If your load won't get high, these costs will be close to $0.

## Prerequisites

If you're deploying from your local machine (not from a CI/CD pipeline), you need the following prerequisites:

- Upstash account. To create one, navigate to [Upstash console](https://console.upstash.com/login).
- Stacktape installed. To install it, you can follow the [installation instructions](https://docs.stacktape.com/getting-started/setup-stacktape/).

- Node.js installed.
- **(optional) install [Stacktape VSCode extension](https://marketplace.visualstudio.com/items?itemName=stacktape.vscode-stacktape) with
  validation, autocompletion and on-hover documentation.**

## 1. Generate your project

The command below will bootstrap the project with pre-built application code and pre-configured `stacktape.yml` config file.

```bash
stp init --projectId kafka-stream-processing
```

## 2. Deploy your stack

- To provision all the required infrastructure and to deploy your application to the cloud, all you need is a single
  command.
- The deployment will take ~5-15 minutes. Subsequent deploys will be significantly faster.

```bash
stp deploy --stage <<stage>> --region <<region>>
```

`stage` is an arbitrary name of your environment (for example **staging**, **production** or **dev-john**)

`region` is the AWS region, where your stack will be deployed to. All the available regions are listed below.

<br />

| Region name & Location     | code           |
| -------------------------- | -------------- |
| Europe (Ireland)           | eu-west-1      |
| Europe (London)            | eu-west-2      |
| Europe (Frankfurt)         | eu-central-1   |
| Europe (Milan)             | eu-south-1     |
| Europe (Paris)             | eu-west-3      |
| Europe (Stockholm)         | eu-north-1     |
| US East (Ohio)             | us-east-2      |
| US East (N. Virginia)      | us-east-1      |
| US West (N. California)    | us-west-1      |
| US West (Oregon)           | us-west-2      |
| Canada (Central)           | ca-central-1   |
| Africa (Cape Town)         | af-south-1     |
| Asia Pacific (Hong Kong)   | ap-east-1      |
| Asia Pacific (Mumbai)      | ap-south-1     |
| Asia Pacific (Osaka-Local) | ap-northeast-3 |
| Asia Pacific (Seoul)       | ap-northeast-2 |
| Asia Pacific (Singapore)   | ap-southeast-1 |
| Asia Pacific (Sydney)      | ap-southeast-2 |
| Asia Pacific (Tokyo)       | ap-northeast-1 |
| China (Beijing)            | cn-north-1     |
| China (Ningxia)            | cn-northwest-1 |
| Middle East (Bahrain)      | me-south-1     |
| South America (São Paulo)  | sa-east-1      |

## 3. Test your application

After a successful deployment, some information about the stack will be printed to the console (**URLs** of the deployed services, links to **logs**, **metrics**, etc.).

1. Make multiple requests to https://YOUR_MAIN_GATEWAY_URL/produce/YOUR_MESSAGE
2. Go to kafkaConsumer -> logs (AWS link will be printed to the console) and see how the messages are handled in
   batches.

## 4. Run the application in development mode

To run functions in the development mode (remotely on AWS), you can use the
[dev command](https://docs.stacktape.com/cli/commands/dev/). For example, to develop and debug lambda function `kafkaProducer`, you can use

```bash
stp dev --region <<your-region>> --stage <<stage>> --resourceName kafkaProducer
```

The command will:

- quickly re-build and re-deploy your new function code
- watch for the function logs and pretty-print them to the terminal

The function is rebuilt and redeployed, when you either:

- type `rs + enter` to the terminal
- use the `--watch` option and one of your source code files changes

## 5. Hotswap deploys

- Stacktape deployments use [AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html) under the hood. It
  brings a lot of guarantees and convenience, but can be slow for certain use-cases.

- To speed up the deployment, you can use the --hotSwap flag that avoids Cloudformation.
- Hotswap deployments work only for source code changes (for lambda function, containers and batch jobs) and for content uploads to buckets.
- If the update deployment is not hot-swappable, Stacktape will automatically fall back to using a Cloudformation deployment.

```bash
stacktape deploy --hotSwap --stage <<stage>> --region <<region>>
```

## 6. Delete your stack

- If you no longer want to use your stack, you can delete it.
- Stacktape will automatically delete every infrastructure resource and deployment artifact associated with your stack.

```bash
stp delete --stage <<stage>> --region <<region>>
```

# Stack description

Stacktape uses a simple `stacktape.yml` configuration file to describe infrastructure resources, packaging, deployment
pipeline and other aspects of your services.

You can deploy your services to multiple environments (stages) - for
example `production`, `staging` or `dev-john`. A stack is a running instance of a service. It consists of your application
code (if any) and the infrastructure resources required to run it.

The configuration for this service is described below.

## 1. Service name

You can choose an arbitrary name for your service. The name of the stack will be constructed as
`{service-name}-{stage}`.

```yml
serviceName: kafka-stream-processing
```

## 2. Resources

- Every resource must have an arbitrary, alphanumeric name (A-z0-9).
- Stacktape resources consist of multiple (sometimes more than 15) underlying AWS or 3rd party resources.

### 2.1 HTTP API Gateway

API Gateway receives requests and invokes our kafkaProducer function.

You can specify [more properties](https://docs.stacktape.com/resources/http-api-gateways/) on the gateway, in our case
we are going with the default setup.

```yml
resources:
  mainApiGateway:
    type: http-api-gateway
```

### 2.2 Upstash Kafka topic

Upstash Kafka topic is a message hub within our application. Both consumer and producer functions are using it.

You can specify [more properties](https://docs.stacktape.com/resources/upstash-kafka-topics/) on the topic, i.e specify
custom Upstash Kafka cluster or set the number of topic partitions. In this case, we are keeping the defaults.

```yml
kafkaTopic:
  type: upstash-kafka-topic
```

### 2.3 Functions

Core of our application consists of two serverless functions:

- **producer function** - produces messages into kafka topic (triggered by HTTP API gateway)
- **consumer function** - consumes messages from kafka topic (triggered when there are messages in the topic)

#### 2.3.1 Producer function

Producer function is configured as follows:

- **Packaging** - determines how the lambda artifact is built. The easiest and most optimized way to build the lambda
  from Typescript/Javascript is using `stacktape-lambda-buildpack`. We only need to configure `entryfilePath`. Stacktape
  automatically transpiles and builds the application code with all of its dependencies, creates the lambda zip
  artifact, and uploads it to a pre-created S3 bucket on AWS. You can also use
  [other types of packaging](https://docs.stacktape.com/configuration/packaging/#packaging-lambda-functions).
- **Environment variables** - We are passing information about the topic to the function using an environment variables.
  Topic parameters can be easily referenced using a
  [$ResourceParam() directive](https://docs.stacktape.com/configuration/directives/#resource-param). This directive
  accepts a resource name (`kafkaTopic` in this case) and the name of the
  [upstash kafka topic referenceable parameter](https://docs.stacktape.com/resources/upstash-kafka-topics/#referenceable-parameters).
  If you want to learn more, refer to
  [referencing parameters](https://docs.stacktape.com/configuration/referencing-parameters/) guide and
  [directives](https://docs.stacktape.com/configuration/directives) guide.
- **Events** - Events determine how is function triggered. In this case, we are triggering the function when an event
  (HTTP request) is delivered to the HTTP API gateway to URL path `/produce/{message}`, where `{message}` is a path
  parameter(message can be arbitrary value). The event(request) including the path parameter is passed to the function
  handler as an argument.

```yml
kafkaProducer:
  type: function
  properties:
    packaging:
      type: stacktape-lambda-buildpack
      properties:
        entryfilePath: ./src/kafka-producer.ts
    events:
      - type: http-api-gateway
        properties:
          httpApiGatewayName: mainApiGateway
          path: /produce/{message}
          method: GET
    environment:
      - name: KAFKA_ENDPOINT
        value: $ResourceParam('kafkaTopic', 'tcpEndpoint')
      - name: TOPIC_NAME
        value: $ResourceParam('kafkaTopic', 'topicName')
      - name: USERNAME
        value: $ResourceParam('kafkaTopic', 'username')
      - name: PASSWORD
        value: $ResourceParam('kafkaTopic', 'password')
```

#### 2.3.2 Consumer function

Consumer function is configured as follows:

- **Packaging** - determines how the lambda artifact is built. The easiest and most optimized way to build the lambda
  from Typescript/Javascript is using `stacktape-lambda-buildpack`. We only need to configure `entryfilePath`. Stacktape
  automatically transpiles and builds the application code with all of its dependencies, creates the lambda zip
  artifact, and uploads it to a pre-created S3 bucket on AWS. You can also use
  [other types of packaging](https://docs.stacktape.com/configuration/packaging/#packaging-lambda-functions).
- **Events** - Events determine how is function triggered(invoked). In this case, we are triggering the function when
  there are messages in the upstash kafka topic. We are also configuring `batchSize`(amount of messages that can be
  included in one invocation) and `maxBatchWindowSeconds`(amount of seconds, messages are collected, before the function
  is invoked).

```yml
kafkaConsumer:
  type: function
  properties:
    packaging:
      type: stacktape-lambda-buildpack
      properties:
        entryfilePath: ./src/kafka-consumer.ts
    events:
      - type: kafka-topic
        properties:
          upstashKafkaTopic: kafkaTopic
          batchSize: 5
          maxBatchWindowSeconds: 10
```