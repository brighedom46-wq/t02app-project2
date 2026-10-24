# Project 2: Technical Rationale

## 1. Project Overview

This project demonstrates containerization using Docker, multi-stage image builds, service orchestration with Docker Compose, and local serverless function execution using AWS SAM CLI.

The application includes a static website served by Nginx, a separate Redis cache service, and a Node.js 22 AWS Lambda function. All components were developed and tested locally without deploying to a cloud provider.

## 2. Base Image Selection and Multi-Stage Builds

I chose Node.js 24 for the build stage because the application uses npm to prepare its files. For the final runtime stage, I selected `nginx:alpine` because Nginx can serve static content without requiring the full Node.js development environment.

The Alpine-based runtime image helps keep the final container lightweight.

I compared two Docker builds and recorded the following results:

| Build Type | Image Size |
|---|---|
| Single-stage | 1.65 GB |
| Multi-stage | 93.6 MB |

The multi-stage image was approximately **94.3% smaller**.

This reduction is possible because the multi-stage Dockerfile separates build-time tools from the final runtime environment. Only the required application output is copied into the Nginx image.

As explained in the Docker documentation and Week 3 lecture, smaller images can reduce image transfer time, storage requirements, and the amount of unnecessary software included in the runtime environment [1, 2].

## 3. Containers and Cloud Service Models

The Week 3 lecture explains how containers relate to Infrastructure as a Service (IaaS) and Platform as a Service (PaaS) [2].

With IaaS, the developer manages the operating system and application environment. With PaaS, the provider manages more of the underlying platform, allowing developers to focus mainly on application code and configuration.

Containers offer a balance between control and convenience. They share the host operating system kernel, so they do not require a complete guest operating system like a virtual machine. However, developers still control the contents of the container image, including the runtime, libraries, and application dependencies.

I used Docker because this control makes the application environment reproducible and portable across compatible systems.

Using Docker does not automatically make an application IaaS or PaaS. The service model depends on who manages the underlying infrastructure and runtime.

## 4. Docker Compose and Service Separation

I used Docker Compose to manage two separate services:

- **Web:** An Nginx container that serves the static website on port 8080.
- **Cache:** A Redis 7 Alpine container with a health check.

I kept Redis separate from Nginx because each service has a different responsibility. Nginx serves web content, while Redis provides caching functionality.

Separating the services allows each container to have its own image, configuration, lifecycle, and health status.

This also connects to the Week 3 discussion of cloud service models. Containers allow developers to decide which components they manage directly and which pre-built dependencies they use.

By selecting `redis:7-alpine`, I reused an existing Redis image rather than installing a caching server inside the web container. This demonstrates how containerized applications can combine independently managed components [2, 3].

In this project, Redis is included to demonstrate service orchestration and dependency management. The static website does not actively use Redis for caching.

## 5. Health Checks and Dependency Testing

I configured Docker Compose so that the web service waits for Redis to become healthy before starting.

Redis readiness was verified using:

```bash
docker compose exec cache redis-cli ping
```

The result was:

```text
PONG
```

I also tested what would happen if Redis stopped after both containers were running.

After stopping the cache service, the Nginx website continued returning HTTP status `200`.

This demonstrated that a startup dependency is not necessarily a runtime dependency. Although Docker Compose waits for Redis during startup, the static website can continue serving content without it.

Docker's documentation explains how health checks and dependency conditions can control service startup order [3].

## 6. AWS SAM and Function as a Service

I used AWS SAM CLI to invoke a Node.js 22 Lambda function locally.

AWS Lambda is an example of Function as a Service (FaaS). As discussed in Week 3, developers supply function code while the cloud provider manages the execution infrastructure [2, 4].

I tested the function using:

```bash
cd local-fn
sam local invoke HelloWorldFunction
```

The function returned:

```json
{
  "statusCode": 200,
  "body": "{\"message\":\"hello world\"}"
}
```

This confirmed that the function executed successfully in the local Lambda environment.

AWS SAM uses Docker to run the function locally, allowing development and testing without deploying the application to AWS [5].

## 7. Shared Responsibility and Security

The Week 3 lecture explains that responsibility changes depending on the service model, but it never disappears [2].

When using Docker, I remain responsible for the application code, container configuration, dependencies, and appropriate security controls.

The multi-stage Dockerfile reduces unnecessary software in the final image. However, a smaller image does not automatically guarantee security, so dependencies and base images still need to be maintained.

Using separate services also makes it easier to manage individual components without combining unrelated responsibilities into one container.

## 8. Testing and Reproducibility

I tested the project using Docker Compose, container inspection commands, Redis health checks, and AWS SAM local invocation.

I also cloned the Git repository into a new directory to verify reproducibility.

From the fresh clone, I successfully:

- Built and started the Docker Compose services.
- Confirmed Redis reached a healthy state.
- Received HTTP status `200` from the Nginx website.
- Stopped Redis and verified the web service continued responding.
- Invoked the AWS Lambda function and received the expected response.

These tests demonstrated that the committed project files were sufficient to rebuild and run the application locally.

## 9. Conclusion

This project helped me understand how containers provide control over application environments while avoiding the overhead of separate virtual-machine operating systems.

The multi-stage Docker build reduced my image size from 1.65 GB to 93.6 MB. Docker Compose allowed me to manage Nginx and Redis independently, while the health check and dependency failure test helped me understand service relationships.

AWS SAM also demonstrated how a serverless function can be tested locally before cloud deployment.

Overall, the project connects the Week 3 concepts of service models, shared responsibility, and container portability with practical implementation and testing.

## 10. References

[1] Docker Inc. (n.d.). *Multi-stage builds*. Docker Documentation.  
https://docs.docker.com/build/building/multi-stage/

[2] Red Deer Polytechnic. (2026). *S03: Taxonomy – Service Models*. CPRO 2601 Cloud Computing. Course lecture slides.

[3] Docker Inc. (n.d.). *Control startup and shutdown order in Compose*. Docker Documentation.  
https://docs.docker.com/compose/how-tos/startup-order/

[4] Amazon Web Services. (n.d.). *What is AWS Lambda?* AWS Lambda Developer Guide.  
https://docs.aws.amazon.com/lambda/latest/dg/welcome.html

[5] Amazon Web Services. (n.d.). *sam local invoke*. AWS SAM Developer Guide.  
https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-local-invoke.html
