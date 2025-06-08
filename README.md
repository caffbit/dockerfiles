## About
There are some directories in this repository. Each directory has a Dockerfile, with different functionality.

|Directory|Function|
|:--:|:--|
|base|LazyVim + basic setup + git auto-commit (mods)|
|react|LazyVim + React/TS ecosystem + AI tools (Aider, Copilot, Avante) + git auto-commit (mods)|

## Usage

### Build Image
```bash
# Single architecture
docker build --platform linux/arm64 -t <directory> .

# Multi-architecture with Buildx
docker buildx build --platform linux/amd64,linux/arm64 
 -t your-registry/dotfiles:<directory> 
 --push .
```

### Run Container
```bash
# Create and start container with persistent volume
docker run -d --name <name> [-p <host_port>:<container_port>] <directory> tail -f /dev/null

# Enter container
docker exec -it <name> bash
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