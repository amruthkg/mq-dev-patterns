
#  Copyright 2022, 2026 IBM Corp.
#  Licensed under the Apache License, Version 2.0 (the 'License');
#  you may not use this file except in compliance with the License.
#  You may obtain a copy of the License at
 
#  http://www.apache.org/licenses/LICENSE-2.0
 
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
#  limitations under the License.

# This Dockerfile has two separate stages.
# The first stage builds the Node program.
# The second stage is a runtime-only container that holds just the things we need to
# serve the compiled program.

###########################################################
# This starts the BUILD phase
###########################################################
FROM public.ecr.aws/docker/library/node:22-alpine AS builder
WORKDIR /app

# Copy manifest files first so npm ci is cached independently of source changes
COPY ./AWS-MQ-showcase/package.json ./AWS-MQ-showcase/package-lock.json ./

# Install dependencies from the lockfile; skip Husky hooks in CI/Docker
ENV HUSKY=0
RUN npm ci

# Copy application source and build
COPY ./AWS-MQ-showcase/ ./
RUN npm run build

###########################################################
# This starts the RUNTIME phase
###########################################################
# nginx-unprivileged runs as UID 101 (non-root) and listens on port 8080
FROM public.ecr.aws/nginx/nginx-unprivileged:stable-alpine AS frontendimage

# Set nginx configuration as per application requirements
COPY nginx.conf.template /etc/nginx/conf.d/
COPY nginx.conf.proxy.template /etc/nginx/conf.d/
# --chmod=755 ensures the entrypoint is executable regardless of host umask
COPY --chmod=755 docker-entrypoint.sh /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]

# Copy only the built static assets from the builder stage
COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD wget -qO- http://localhost:8080/ || exit 1

CMD ["nginx", "-g", "daemon off;"]
