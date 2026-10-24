# Project 2: Technical Rationale

## Base Image Selection and Multi-Stage Build

I used Node.js 24 for the build stage because the application requires npm to prepare its files. For the final runtime image, I selected `nginx:alpine` because Nginx can serve the static website without needing the full Node.js development environment. Alpine also helps keep the image lightweight.

I compared the single-stage and multi-stage Docker images. The single-stage image was **1.65 GB**, while the multi-stage image was only **93.6 MB**, a reduction of approximately **94.3%**.

The multi-stage build separates development tools from the runtime environment. Only the built application files are copied into the final Nginx image. This reduces storage requirements, image transfer time, and unnecessary software in the final container [1].

## Docker Compose and Redis Service Separation

I used Docker Compose to manage two services: `web`, which runs Nginx, and `cache`, which runs Redis 7 Alpine.

I kept Redis separate because Nginx and Redis have different responsibilities. Nginx serves the website, while Redis provides a caching service. Keeping them separate allows each service to be configured, monitored, and managed independently.

This design connects to the Week 3 discussion of cloud service models. Containers provide control over the application environment while allowing developers to reuse pre-built components. By using `redis:7-alpine`, I can include a ready-made Redis service without installing or maintaining Redis inside the web container [2].

I also configured a Redis health check and used `depends_on` so that the web container waits for Redis to become healthy during startup [3].

During testing, Redis returned `PONG`, and the website returned HTTP `200`. When I stopped Redis, the website continued working because the static Nginx page does not actually use Redis for caching.

## AWS SAM and Local Testing

I used AWS SAM CLI to execute a Node.js 22 Lambda function locally. The function returned HTTP status `200` with the expected `hello world` message.

This demonstrates Function as a Service (FaaS), where developers provide function code while the cloud provider manages the execution infrastructure. AWS SAM allowed me to test the function locally without deploying it to AWS [4].

I also tested the project from a fresh Git clone. The Docker Compose services started successfully, and the Lambda function returned the expected response. This confirmed that the project could be reproduced using the committed files.

## Conclusion

This project demonstrated how Docker multi-stage builds reduce image size, how Docker Compose manages separate services, and how AWS SAM supports local serverless testing. It also helped me understand the relationship between containers, cloud service models, and developer responsibilities.

## References

[1] Docker Inc. (n.d.). *Multi-stage builds*.  
https://docs.docker.com/build/building/multi-stage/

[2] Red Deer Polytechnic. (2026). *S03: Taxonomy – Service Models*. CPRO 2601 Cloud Computing. Course lecture slides.

[3] Docker Inc. (n.d.). *Control startup and shutdown order in Compose*.  
https://docs.docker.com/compose/how-tos/startup-order/

[4] Amazon Web Services. (n.d.). *sam local invoke*. AWS SAM Developer Guide.  
https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-local-invoke.html
