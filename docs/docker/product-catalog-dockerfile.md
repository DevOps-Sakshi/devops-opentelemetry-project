# Product Catalog Dockerfile – Explanation

This Dockerfile uses a multi-stage build to compile a Go-based microservice and then produce a lightweight runtime image.

## Stage 1: Build Stage (Go + Build Environment)
`FROM golang:1.22-alpine AS builder`
- Uses Go 1.22 on Alpine Linux as the base image.
- Tagged as builder to reference in the runtime stage.
- Alpine provides a small Linux environment with Go installed.

`WORKDIR /usr/src/app/`
Sets the working directory inside the container where all build files will reside.

## Use Go build cache for dependencies
```hcl
RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    mkdir -p /root/.cache/go-build
```
- Creates cache directories to speed up Go module downloads and builds.
- --mount=type=cache allows caching dependencies between builds for faster rebuilds.

## Copy
`COPY go.mod go.sum ./`
- Copies Go module files first (go.mod and go.sum).
- Allows Docker to cache dependency installation separately from source code changes.

## Download
`RUN go mod download`
- Downloads all Go module dependencies.
- This ensures all required libraries are available before building the application.

## Copy the rest of the source code
`COPY . .`
- Copies the full source code into the container.

`RUN go build -o product-catalog .`
- Compiles the Go application into a single executable named product-catalog.

## Stage 2: Runtime Stage (Lightweight Alpine Image)
`FROM alpine AS release`
- Uses a minimal Alpine Linux image to reduce the final image size.
- No Go compiler is needed here because the application is already compiled.

`WORKDIR /usr/src/app/`
- Sets the working directory for runtime execution.

```hcl
COPY ./products/ ./products/
COPY --from=builder /usr/src/app/product-catalog/ ./
```
- Copies any static product data from the local products/ directory.
- Copies the compiled executable product-catalog from the builder stage.
- Keeps only what’s needed for runtime → small and efficient image.

`ENV PRODUCT_CATALOG_PORT=8088`
- Sets an environment variable to define the service port.
- Useful for container orchestration (Docker Compose, Kubernetes).

`ENTRYPOINT [ "./product-catalog" ]`
- Defines the entrypoint for the container.
- When the container starts, it runs the compiled Go executable.