# Neovim + Node 開發環境

這個 Docker 映像提供了一個完整配置的 Neovim 開發環境，內建 Node.js 支援。它專為跨多架構（AMD64 和 ARM64）工作而設計，可以在 Windows、macOS（Intel 或 M1/M2）和 Linux 系統上使用。

## 使用方式

### 參數說明
- `<name>`: 容器名稱，例如 my-project
- `<host_port>`: 主機端口（可選），例如 5173
- `<container_port>`: 容器內端口（可選），例如 5173

### 構建映像

單一架構構建

```
# 為您當前的架構構建
docker build --platform linux/arm64 -t react .
```

多架構構建

```
# 設置 Docker Buildx（如果尚未完成）
docker buildx create --name mybuilder --use
# 構建並推送多架構映像
docker buildx build --platform linux/amd64,linux/arm64 \
  -t your-registry/dotfiles:react \
  --push .
```

### 執行容器

此映像設計為將所有專案檔案保留在容器內，使用 Docker 磁碟區來實現持久化，避免在主機檔案系統中產生任何雜亂。

```bash
# 建立並啟動具有持久磁碟區的容器
docker run -d --name <name> [-p <host_port>:<container_port>] -v application:/apps react tail -f /dev/null

# 進入容器
docker exec -it <name> bash
```

### SSH 設定與授權

```
# 在容器內執行
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Windows PowerShell
docker cp ${env:USERPROFILE}/.ssh/id_ed25519 <name>:/root/.ssh/

# Linux/macOS
docker cp ~/.ssh/id_ed25519 <name>:/root/.ssh/

# 在容器內執行
chmod 600 /root/.ssh/id_ed25519

# 測試 SSH 連接
ssh -T git@github.com
```

