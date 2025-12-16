# GitLab CI/CD Pipeline

Complete CI/CD pipeline for PHP application deployment using GitLab, Docker, and Ansible.

## Features

- **3-Stage Pipeline**: Build → Test → Deploy
- **Custom GitLab Runner**: Docker-in-Docker with Ansible support
- **Automated Testing**: PHP syntax check + HTTP functional test
- **Healthcheck Integration**: Docker HEALTHCHECK + Ansible verification
- **Secure Credentials**: Ansible Vault for inventory encryption
- **SHA Versioning**: Images tagged with commit SHA for traceability

## Architecture

```
  GitLab Server
         │
         │ git push triggers pipeline
         ▼
  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
  │    BUILD     │ ──▶ │     TEST     │ ──▶ │    DEPLOY    │
  │ docker build │     │  php -l      │     │   ansible    │
  │ docker save  │     │  curl test   │     │  healthcheck │
  └──────────────┘     └──────────────┘     └──────────────┘
                                                   │
                                                   ▼
                                        Target VM
                                        PHP/Apache Application
                                        with Docker healthcheck
```

## Quick Start

### 1. Setup GitLab Runner

```bash
# Build custom runner
cd runner
DOCKER_GID=$(getent group docker | cut -d: -f3)
docker build --build-arg DOCKER_GID=$DOCKER_GID -t gitlab-runner-custom .

# Start runner
docker run -d --name gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /shared:/shared \
  gitlab-runner-custom

# Register runner
docker exec -it gitlab-runner gitlab-runner register \
  --non-interactive \
  --url "http://YOUR_GITLAB_URL" \
  --registration-token "YOUR_TOKEN" \
  --description "docker-runner" \
  --tag-list "docker" \
  --executor "shell"
```

### 2. Configure Inventory

```bash
# Create inventory from example
cp inventory.ini.example inventory.ini

# Edit with your target VM credentials
nano inventory.ini

# Encrypt with Vault
ansible-vault encrypt inventory.ini
```

### 3. Push to GitLab

```bash
git add .
git commit -m "Initial commit"
git push origin main
```

Pipeline will automatically:
1. Build Docker image
2. Run PHP syntax check and HTTP test
3. Deploy to target VM if tests pass

## Project Structure

```
gitlab-cicd-pipeline/
├── src/
│   └── index.php              # PHP application
├── runner/
│   └── Dockerfile             # Custom GitLab Runner
├── Dockerfile                 # Application image (with healthcheck)
├── playbook.yml               # Ansible deployment playbook
├── .gitlab-ci.yml             # Pipeline configuration
└── inventory.ini.example      # Inventory template
```

## Pipeline Stages

### Build Stage

- Builds Docker image from Dockerfile
- Tags with commit SHA for versioning
- Saves image to shared directory

### Test Stage

- **Syntax check**: `php -l` validates PHP code
- **Functional test**: Starts container and verifies HTTP response
- Blocks deployment if any test fails

### Deploy Stage

- Uses Ansible to deploy to target VM
- Loads Docker image from archive
- Performs healthcheck after deployment

## Configuration

### GitLab CI Variables

| Variable | Description |
|----------|-------------|
| `APP_IMAGE` | Docker image name with SHA tag |
| `DOCKER_ARCHIVE` | Archive filename for image transfer |
| `SHARED_PATH1` | Path on target VM |
| `SHARED_PATH2` | Path in runner container |

### Customization

Edit `.gitlab-ci.yml` to:

- Deploy only from main branch (uncomment `only: main`)
- Add additional test stages
- Configure notifications

## Security

- **Vault encryption**: Inventory with credentials is encrypted
- **Key file**: Store vault password securely
- **Production recommendation**: Use GitLab CI/CD Variables instead of file-based keys

## Requirements

- GitLab with CI/CD enabled
- Docker on runner host
- Target VM with Docker installed
- SSH access to target VM

## License

MIT License - See [LICENSE](../LICENSE)
