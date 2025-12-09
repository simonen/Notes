---
tags:
  - containers
---

Example

`Dockerfile`
```
FROM python:3.11-alpine  
  
WORKDIR /usr/src/app  
  
COPY . .  
  
RUN pip install --no-cache-dir -r requirements.txt  
  
EXPOSE 3000  
  
CMD ["python", "./app.py"]
```

Inspect the image history and build layers

```bash
docker image history "IMAGE"
```

```
IMAGE          CREATED         CREATED BY                     SIZE 
fb2fb3717ea0   22 hours ago    CMD ["python" "app.py"]
<missing>      22 hours ago    COPY . . # buildkit
<missing>      22 hours ago    RUN /bin/sh -c pip install -r requirements.t… 
```
####  Image Optimization

1. Use official images (`python:3.11, debian:bookworm, alpine`)
2. Consider **distroless** images for maximum security

https://hub.docker.com/_/python For tags

`python:3.11-alpine`: very lightweight base image
##### Minimize Layers

Combine instructions for less layers, but do not sacrifice readability

`Dockerfile`
```
RUN apt-get update && apt-get install -y \
    curl \
    git \
 && rm -rf /var/lib/apt/lists/*
```

##### Order Instructions for Caching

> Docker builds layers in order. Changing any early step invalidates all later layers.

Bad ordering example

```
.
├── app.py
├── requirements.txt
└── static/
```

```
FROM python:3.11

WORKDIR /app

COPY . .                  # ❌ Copy everything first
RUN pip install -r requirements.txt

CMD ["python", "app.py"]
```

1.  `COPY . .`: copies all files into `/app`
2. `RUN pip install -r requirements.txt` install dependencies
3. Image built successfully 

If `app.py` is changed:
- Docker detects `COPY . .` has changed because `app.py` is included
- That invalidates all subsequent layers (including the dependency installation)
- Docker reinstalls Flask and Requests even though `requirements.txt` didn't change.

`Initial build`
```
 => [1/4] FROM docker.io/library/python:3.11@sha256:ef1a0dc..dd5edc    52.2s
 => [2/4] WORKDIR /app
 => [3/4] COPY . .
 => [4/4] RUN pip install -r requirements.txt
```

Rebuild

```
 => [1/4] FROM docker.io/library/python:3.11@sha256:ef1a087
 => CACHED [2/4] WORKDIR /app
 => [3/4] COPY . .
 => [4/4] RUN pip install -r requirements.txt
```

Optimized Ordering

```
FROM python:3.11

WORKDIR /app

COPY requirements.txt .   # ✅ Copy only dependencies file first
RUN pip install -r requirements.txt

COPY . .                  # ✅ Copy app code later

CMD ["python", "app.py"]
```

The dependency installation is cached unless  `requirements.txt`  changes.

```
=> [1/5] FROM docker.io/library/python:3.11@sha256:ef1a0
=> CACHED [2/5] WORKDIR /app
=> CACHED [3/5] COPY requirements.txt .
=> CACHED [4/5] RUN pip install -r requirements.txt
=> [5/5] COPY . .
```

> Copy files that are often changed last to avoid 

##### Use Explicit Versions

- Pin dependencies in `requirements.txt` or `apg-get install pkg=1.2.3` to avoid unexpected updates
- Avoid `:latest` in base images - it can break builds unpredictably.

##### Security

- Do not run as root unless necessary:

`Dockerfile`
```
RUN adduser -D appuser
USER appuser
```

- Avoid embedding secrets in the `Dockerfile` (user environment variables or Docker secrets)

##### Multi-stage Builds for Smaller Images

In the builder stage, you usually have compiles and temp files you don't want in the final image

- First stage: Build the app (with compilers, tools).
- Second stage: Copy only the compiled app into a minimal runtime image:

`Dockerfile`
```
# ---------- Build stage ----------
FROM python:3.11-alpine AS builder

WORKDIR /app

# Install build dependencies for compiling Python packages
RUN apk add --no-cache build-base

# Only copy requirements to leverage caching
COPY requirements.txt .

# Install dependencies into a temporary directory
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# ---------- Runtime stage ----------
FROM python:3.11-alpine

WORKDIR /app

# Copy installed packages from builder stage into Python's standard location
COPY --from=builder /install /usr/local

# Copy application code last to avoid cache busting
COPY . .

EXPOSE 3000

CMD ["python", "./app.py"]
```

1. Stage 1 (builder)
	- Installs `build-base` (C compiler + headers) only for building dependencies like `psycopg2` or `numpy`
	- Installs packages into `/install` so we can copy them later without the `pip` or build tools in the final image.
2. Stage 2 (runtime)
	- Starts from a clean `python:3.11-alpine` image without build tools
	- Copies only the built dependencies and source code



#### Build an Apache Image 

Image is based on Amalinux, installs and runs apache web server

The official Apache images run a helper script to execute the Apache web server. This script acts as a wrapper command that can contain additional pre-run commands. It also makes it easier to use `httpd-foreground` in multiple scripts instead of the raw 
`CMD ["httpd", "-D", "FOREGROUND"]`

`httpd-foreground`
```
#!/bin/sh
set -e

# Apache gets grumpy about PID files pre-existing
rm -f /usr/local/apache2/logs/httpd.pid

exec httpd -DFOREGROUND "$@"
```

`Dockerfile`
```
FROM almalinux:10

RUN dnf update -y && dnf install -y httpd && \
	dnf clean all && \
	
COPY web/index.html /var/www/html

EXPOSE 80

CMD ["httpd", "-DFOREGROUND"]
```

When building PHP images

`Dockerfile`
```
# Use official PHP with Apache image
FROM php:8.0-apache

# Install PDO MySQL extension
RUN docker-php-ext-install pdo_mysql \
    && docker-php-source delete
```

MariaDB Dockerfile

`Dockerfile`
```
FROM mariadb:10.7

ADD ./db/db_setup.sql /docker-entrypoint-initdb.d/init.sql
```