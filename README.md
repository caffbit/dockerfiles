## About
There are some directories in this repository. Each directory has a Dockerfile, with different functionality.

|Directory|Function|
|:--:|:--|
|base|LazyVim + basic setup + git auto-commit (mods)|
|react|LazyVim + React/TS ecosystem + AI tools (Aider, Copilot, Avante) + git auto-commit (mods)|

## Usage

### Pull from Docker Hub
```bash
# Pull multi-architecture image (automatically selects your architecture)
docker pull your-registry/dotfiles:<directory>

# Pull specific architecture (if needed)
docker pull --platform linux/<arch> your-registry/dotfiles:<directory>
```

### Build Image
```bash
# Local build (current architecture)
docker build -t <directory> .

# Docker Hub build (current architecture)
docker buildx build -t your-registry/dotfiles:<directory> --push .

# Check current architecture
docker image inspect <directory> --format='{{.Architecture}}'

# Multi-architecture workflow (when needed)
# 1. Build for different architecture with temporary tag
docker buildx build -t your-registry/dotfiles:<directory>-<arch> --push .

# 2. Create manifest to combine architectures
docker manifest create your-registry/dotfiles:<directory> 
  --amend your-registry/dotfiles:<directory> 
  your-registry/dotfiles:<directory>-<arch>

# 3. Push combined manifest
docker manifest push your-registry/dotfiles:<directory>
```

### Run Container
```bash
# Create and start container
docker run -d --name <name> [-p <host_port>:<container_port>] <directory> tail -f /dev/null

# Enter container
docker exec -it <name> bash

# Add ports to existing container (preserves data)
docker commit <name> <name>-backup
docker stop <name> && docker rm <name>
docker run -d --name <name> -p <host_port>:<container_port> <name>-backup tail -f /dev/null
```

### SSH Setup (in container)
```bash
# Setup SSH directory
mkdir -p ~/.ssh && chmod 700 ~/.ssh

# Windows Copy SSH key from host 
docker cp ${env:USERPROFILE}/.ssh/id_ed25519 <name>:/root/.ssh/

# Linux/macOS Copy SSH key from host 
docker cp ~/.ssh/id_ed25519 <name>:/root/.ssh/

# Set permissions and test
chmod 600 /root/.ssh/id_ed25519
ssh -T git@github.com
```

### Parameters
- `<directory>`: base or react
- `<name>`: Container name (e.g., my-project)
- `<host_port>`: Host port (optional, e.g., 5173)
- `<container_port>`: Container port (optional, e.g., 5173)
- `<arch>`: Architecture (e.g., amd64, arm64)
