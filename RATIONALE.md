# Project 2: Technical Rationale

## Base Image Selection and Multi-Stage Build

For this project, I used Node.js 24 in the build stage and `nginx:alpine` in the final runtime stage. Node.js provides the npm environment needed to prepare the application, while Nginx serves the static website without requiring Node.js in the final container.

I selected Alpine-based Nginx because it provides a smaller runtime environment. Docker's multi-stage build approach allows me to separate development dependencies from the files needed to run the application [1].

My single-stage Docker image was **1.65 GB**, compared with **93.6 MB** for the multi-stage image. This represents an approximately **94.3% reduction** in image size. A smaller image reduces storage and transfer requirements.

## Docker Compose and Service Separation

I used Docker Compose to run two separate services: `web` (Nginx) and `cache` (Redis 7 Alpine). Nginx serves the website on port 8080, while Redis provides a separate cache service.

I used the official Redis image instead of installing Redis inside the web container. This keeps the services independent and allows them to be managed separately.

The project also includes a Redis health check and `depends_on` configuration. This allows Docker Compose to wait until Redis is healthy before starting the web service [2].

During testing, Redis returned `PONG`, and the website returned HTTP `200`. After stopping Redis, the website continued working because the static page does not depend on Redis at runtime.

## Connection to Week 3 Cloud Service Models

Week 3 introduced Infrastructure as a Service (IaaS), Platform as a Service (PaaS), and Function as a Service (FaaS).

Docker containers give developers control over application dependencies and runtime environments. Depending on the hosting platform, containers can run on infrastructure managed through IaaS or on a managed container platform with PaaS-like responsibilities.

Using a prebuilt Redis image also demonstrates how existing software components can be reused instead of installing and configuring every dependency manually [3].

## AWS SAM and Local Serverless Testing

I used AWS SAM CLI to run a Node.js 22 Lambda function locally. The command `sam local invoke HelloWorldFunction` returned status code `200` and the expected `hello world` message.

This demonstrates the FaaS model, where developers focus on function code while the cloud provider manages the execution infrastructure during deployment. AWS SAM makes it possible to test Lambda functions locally using Docker without deploying them to AWS [4].

## Conclusion

This project helped me understand how multi-stage builds reduce container image sizes, how Docker Compose manages separate services, and how AWS SAM supports local serverless development. I also tested the project from a fresh Git clone to confirm that the committed files could reproduce the expected results.

## References

[1] Docker Inc. (n.d.). *Multi-stage builds*. Docker Documentation.  
https://docs.docker.com/build/building/multi-stage/

[2] Docker Inc. (n.d.). *Control startup and shutdown order in Compose*. Docker Documentation.  
https://docs.docker.com/compose/how-tos/startup-order/

[3] Red Deer Polytechnic. (2026). *S03: Taxonomy – Service Models*. CPRO 2601 Cloud Computing. Course lecture slides.

[4] Amazon Web Services. (n.d.). *sam local invoke*. AWS SAM Developer Guide.  
https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-local-invoke.html
