# Project 2: Technical Rationale

## 1. Multi-Stage Docker Build

I chose a multi-stage Dockerfile to separate the application build environment from the final runtime environment.

The first stage uses Node.js 24 to install dependencies and build the application. The second stage uses Nginx Alpine to serve the generated static files.

This approach keeps Node.js, npm, and build dependencies out of the final runtime image.

My image-size comparison showed:

- Single-stage image: 1.65 GB
- Multi-stage image: 93.6 MB

The multi-stage image was approximately 94% smaller. This reduces the amount of image data that needs to be transferred and stored.

## 2. Base Image Selection

I selected `node:24` for the build stage because the application uses npm to generate its static output.

For the runtime stage, I selected `nginx:alpine` because Nginx can serve static HTML efficiently without requiring the Node.js development environment.

The Alpine-based image also helps keep the final image smaller.

## 3. Docker Compose and Redis

I used Docker Compose to manage two services: `web` and `cache`.

The `web` service runs Nginx, while the `cache` service uses `redis:7-alpine`.

Redis has a health check that runs `redis-cli ping`. The `depends_on` configuration makes Docker Compose wait until Redis is healthy before starting the web service.

During testing, I stopped Redis and confirmed that Nginx continued returning HTTP 200. This demonstrated that startup dependencies are different from runtime dependencies. The static web application does not actually query Redis.

## 4. Local Serverless Function

I used AWS SAM CLI to create and invoke a Node.js 22 Lambda function locally.

The function returned HTTP status code 200 with a `hello world` response.

This demonstrated how serverless functions can be developed and tested locally using Docker without deploying to AWS.

## References

- Docker: Multi-stage builds  
  https://docs.docker.com/build/building/multi-stage/
- Docker: Compose startup order  
  https://docs.docker.com/compose/how-tos/startup-order/
- Nginx Docker Official Image  
  https://hub.docker.com/_/nginx
- AWS SAM: Local invocation  
  https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/using-sam-cli-local-invoke.html
