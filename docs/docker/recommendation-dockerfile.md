Recommendation Service Dockerfile – Explanation

This Dockerfile builds a Python-based microservice for recommendations and prepares a containerized runtime environment.

🧱 Stage: Base Image + Environment Setup
FROM python:3.12-slim-bookworm AS base


Uses Python 3.12 slim image on Debian Bookworm as the base.

Slim variant keeps the image small by including only essentials.

WORKDIR /usr/src/app


Sets the working directory inside the container where all files and commands will run.

COPY requirements.txt ./


Copies the Python dependencies file first.

Enables Docker caching: dependencies are only reinstalled if requirements.txt changes.

RUN pip install --upgrade pip
RUN pip install -r requirements.txt


Upgrades pip to the latest version.

Installs all Python dependencies required for the recommendation service.

COPY . .


Copies the rest of the application source code into the container.

RUN opentelemetry-bootstrap -a install


Installs OpenTelemetry auto-instrumentation for monitoring and tracing.

Prepares the service for observability in distributed environments.

ENV RECOMMENDATION_PORT 1010


Sets an environment variable to define the service port.

Useful for Docker, Kubernetes, or configuration overrides.

ENTRYPOINT ["python", "recommendation_server.py"]


Defines the entrypoint for the container.

When the container starts, it runs the Python script recommendation_server.py.