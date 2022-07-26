# Express.js API with Postgres

- This project deploys a simple HTTP API built using [Express.js](https://expressjs.com/).
- The application runs in a [container workload](https://docs.stacktape.com/resources/container-workloads/) and uses a
  Postgres [relational database](https://docs.stacktape.com/resources/relational-databases/) to store the data. To
  simplify the database access, this project uses [Prisma](https://www.prisma.io/).

## Pricing

- Fixed price resources:

  - **Container workload** (~$0.012/hour, ~$9/month)
  - **Relational (SQL) database** ($0.017/hour, ~$12.5/month, [free-tier eligible](https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all))

- There are also other resources that might incur costs (with pay-per-use pricing). If your load won't get high, these costs will be close to $0.

## Prerequisites

If you're deploying from your local machine (not from a CI/CD pipeline), you need the following prerequisites:

- Stacktape installed. To install it, you can follow the [installation instructions](https://docs.stacktape.com/getting-started/setup-stacktape/).
- Docker. To install Docker on your system, you can follow [this guide](https://docs.docker.com/get-docker/).

- Node.js installed.
- **(optional) install [Stacktape VSCode extension](https://marketplace.visualstudio.com/items?itemName=stacktape.vscode-stacktape) with
  validation, autocompletion and on-hover documentation.**

## 1. Generate your project

The command below will bootstrap the project with pre-built application code and pre-configured `stacktape.yml` config file.

```bash
stp init --projectId expressjs-api-postgres
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

To test the application, you will need the URL of the API Gateway.

It's printed to the console as **mainApiGateway->url**.

### Create a post

Make a `POST` request to `<<your_http_api_gateway_url>>/post` with the JSON data in its body to save the post. Use your preferred HTTP client or
the following cURL command:

```bash
curl -X POST <<your_http_api_gateway_url>>/posts -H 'content-type: application/json' -d '{ "title": "MyPost", "content": "Hello!", "authorEmail": "info@stacktape.com"}'
```

If the above cURL command did not work, try escaping the JSON content:

```bash
curl -X POST <<your_http_api_gateway_url>>/posts -H 'content-type: application/json' -d '{ \"title\":\"MyPost\",\"content\":\"Hello!\",\"authorEmail\":\"info@stacktape.com\"}'
```

### Get all posts

Make a `GET` request to `<<your_http_api_gateway_url>>/posts` to get all posts.

```bash
curl <<your_http_api_gateway_url>>/posts
```

## 4. Run the application in development mode

To run a container in the development mode (locally on your machine), you can use the
[dev command](https://docs.stacktape.com/cli/commands/dev/).

```bash
stp dev --region <<your-region>> --stage <<stage>> --resourceName apiServer --container api-container
```

Stacktape runs the container as closely to the deployed version as possible:

- Maps all of the container ports specified in the `events` section to the host machine.
- Injects parameters referenced in the environment variables by `$ResourceParam` and `$Secret` directives to the
  running container.
- Injects credentials of the [assumed role](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html) to
  the container. This means that your locally running container will have the exact same IAM permissions as the deployed
  version.
- Pretty-prints logs (stdout/stderr) produced by the container to the terminal.

<br />

The container is rebuilt and restarted, when you either:

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
serviceName: expressjs-api-postgres
```

## 2. Resources

- Every resource must have an arbitrary, alphanumeric name (A-z0-9).
- Stacktape resources consist of multiple (sometimes more than 15) underlying AWS or 3rd party resources.

### 2.1 HTTP API Gateway

API Gateway receives requests and routes them to the container.

For convenience, it has [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) allowed.

```yml
resources:
  mainApiGateway:
    type: http-api-gateway
    properties:
      cors:
        enabled: true
```

### 2.2 Postgres relational database

The application data is stored in a Postgres database. The database is configured as follows:

- **Database credentials**. In this example, we input them directly. For production workloads, you should use
  [secrets](https://docs.stacktape.com/resources/secrets/) to store them securely.

- **Engine type**. We are using `postgres` engine. It uses a single-node database server - the simplest and cheapest
  option.

- **Instance size**. We are using the `db.t2.micro` instance. It has 1 vCPU, 1GB of memory, and is free-tier eligible
  (~$12.5/month without a free tier). To see the full list of available options, refer to
  [AWS instance type list](https://aws.amazon.com/rds/instance-types/).

By default, the version used for the database is the latest AWS-supported stable version (currently `13.4`). Minor
version upgrades are done automatically.

You can also configure many other aspects of your database, such as
[storage](https://docs.stacktape.com/resources/relational-databases/#storage),
[logging](https://docs.stacktape.com/resources/relational-databases/#logging),
[read replicas](https://docs.stacktape.com/resources/relational-databases/#read-replicas), or
[failover instances](https://docs.stacktape.com/resources/relational-databases/#multi-az-mode).

```yml
mainDatabase:
  type: relational-database
  properties:
    credentials:
      masterUserName: admin_user
      masterUserPassword: my_secret_password
    engine:
      type: postgres
      properties:
        primaryInstance:
          instanceSize: db.t2.micro
```

### 2.3 Container workload

Application code runs inside a container workload with a single container. The workload is configured as follows:

- [Container](https://docs.stacktape.com/resources/container-workloads/#containers). This container workload uses only a
  single container: `api-container`. The container is configured as follows:
  - **Packaging** - determines how the Docker container image is built. The easiest and most optimized way to build the
    image for a Typescript application is using `stacktape-image-buildpack`. We only need to configure `entryfilePath`.
    Stacktape automatically transpiles and builds the application code with all of its dependencies, builds the Docker
    image, and pushes it to a pre-created image repository on AWS. You can also use
    [other types of packaging](https://docs.stacktape.com/configuration/packaging/#packaging-container-workloads).
  - **Database connection string** - we are passing it to the container as an environment variable. The connection
    string can be easily referenced using a
    [$ResourceParam() directive](https://docs.stacktape.com/configuration/directives/#resource-param). This directive
    accepts a resource name (`mainDatabase` in this case) and the name of the
    [relational database referenceable parameter](https://docs.stacktape.com/resources/relational-databases/#referenceable-parameters)
    (`connectionString` in this case). If you want to learn more, refer to
    [referencing parameters](https://docs.stacktape.com/configuration/referencing-parameters/) guide and
    [directives](https://docs.stacktape.com/configuration/directives) guide.
  - We are configuring **events**(requests) that can reach the container. By configuring the path to `/{proxy+}`, the
    method to `'*'` and the containerPort to `3000`, the event integration routes all requests (no matter the method or
    path) coming to the HTTP API Gateway to port `3000` of the container.
- [Resources](https://docs.stacktape.com/resources/container-workloads#resources). Resources are shared between
  containers of container workload (in this case, we only have one container). The cheapest available resource
  configuration is `0.25` of virtual CPU and `512` MB of RAM.

You can also configure [scaling](https://docs.stacktape.com/resources/container-workloads#scaling). New (parallel)
container workload instance can be added when (for example) the utilization of your CPU or RAM gets larger than 80%. The
HTTP API Gateway will evenly distribute the traffic to all container workloads.

```yml
apiServer:
  type: container-workload
  properties:
    resources:
      cpu: 0.25
      memory: 512
    containers:
      - name: api-container
        packaging:
          type: stacktape-image-buildpack
          properties:
            entryfilePath: ./src/index.ts
        environment:
          - name: DB_CONNECTION_STRING
            value: $ResourceParam('mainDatabase', 'connectionString')
          - name: PORT
            value: 3000
        events:
          - type: http-api-gateway
            properties:
              containerPort: 3000
              httpApiGatewayName: mainApiGateway
              method: '*'
              path: /{proxy+}
```

## 3. Database migration hooks

To simplify database access and migrations, this project uses [Prisma](https://www.prisma.io/). If you're not familiar
with it, don't worry - it's very simple. [Prisma schema](https://www.prisma.io/docs/concepts/components/prisma-schema)
for this application is already configured at `prisma/schema.prisma` in the project directory.

### 3.1 Generate Prisma client

Prisma offers a database client that we can import into our code. To generate it, we use the `npx prisma generate`
command. To do it automatically every time before the stack is deployed, the command is saved as a
[script](https://docs.stacktape.com/configuration/scripts/) and then used inside a `before:deploy`
[hook](https://docs.stacktape.com/configuration/hooks/).

```yml
serviceName: posts-api-pg

scripts:
  generatePrismaClient:
    executeCommand: npx prisma generate

hooks:
  - triggers: ['before:deploy']
    scriptName: generatePrismaClient
```

### 3.2 Prisma database migration

To sync our Prisma schema with the database, we need to use `npx prisma db push` command. We can't do this before the
database is created, so we use the `after:deploy` hook.

We also need to pass the `DB_CONNECTION_STRING` environment variable to the script. We do it using the
[$ResourceParam()](https://docs.stacktape.com/configuration/directives/#resource-param) directive that will
automatically download the connection string value and pass it to the script.

```yml
scripts:
  generatePrismaClient:
    executeCommand: npx prisma generate
    migrateDb:
    executeCommand: npx prisma db push --skip-generate
    environment:
      - name: DB_CONNECTION_STRING
        value: $ResourceParam('mainDatabase', 'connectionString')

hooks:
  - triggers: ["before:deploy"]
    scriptName: generatePrismaClient
    - triggers: ["after:deploy"]
    scriptName: migrateDb
```

You can also execute the migration script anytime using

```bash
stp script:run --scriptName migrateDb --stage <<previously-used-stage>> --region <<previously-used-region>>
```