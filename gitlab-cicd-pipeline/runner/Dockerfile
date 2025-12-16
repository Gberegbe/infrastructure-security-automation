# Custom GitLab Runner with Docker and Ansible support
# Base image: Docker with Alpine Linux
#
# Build:
#   DOCKER_GID=$(getent group docker | cut -d: -f3)
#   docker build --build-arg DOCKER_GID=$DOCKER_GID -t gitlab-runner-custom ./runner

FROM docker:20.10.16

# Docker group ID argument (required for socket access)
ARG DOCKER_GID

# Install system dependencies
RUN apk add --no-cache \
    python3 \
    py3-pip \
    ansible \
    git \
    bash \
    docker-compose \
    shadow \
    openssh-client \
    sshpass

# Install GitLab Runner binary
RUN wget -O /usr/local/bin/gitlab-runner \
    https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64 && \
    chmod +x /usr/local/bin/gitlab-runner

# Create required directories
RUN mkdir -p /etc/gitlab-runner /builds /shared

# Create gitlab-runner user and configure Docker socket access
RUN addgroup -g ${DOCKER_GID} docker && \
    useradd --system --create-home gitlab-runner && \
    usermod -aG docker gitlab-runner && \
    chown -R gitlab-runner:gitlab-runner /home/gitlab-runner /etc/gitlab-runner /builds /shared

# Default entrypoint
ENTRYPOINT ["gitlab-runner", "run", "--user=gitlab-runner"]
