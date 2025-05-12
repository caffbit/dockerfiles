FROM ubuntu:24.04

# Set build arguments
ARG DEBIAN_FRONTEND=noninteractive
ARG TZ=Asia/Taipei
ARG NODE_VERSION=22
ARG NVIM_VERSION=0.11.1
ARG TARGETARCH

# Set environment variables
ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8
ENV TZ=${TZ}

# Install base system packages
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    openssh-client \
    git \
    tmux \
    curl \
    wget \
    tar \
    fzf \
    gcc \
    fd-find \
    ripgrep \
    python3 \
    python3-pip \
    python3-pynvim \
    luarocks \
    libmsgpack-dev \
    unzip \
    gettext \
    xauth \
    fontconfig \
    locales \
    imagemagick \
    ghostscript \
    sqlite3 \
    libsqlite3-dev \
    xz-utils \
    gawk \
    make \
    gpg && \
    ln -s $(which fdfind) /usr/local/bin/fd && \
    echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen && \
    locale-gen

# Install eza-community (modern ls replacement)
RUN mkdir -p /etc/apt/keyrings && \
    wget -qO- https://raw.githubusercontent.com/eza-community/eza/main/deb.asc | gpg --dearmor -o /etc/apt/keyrings/eza.gpg && \
    echo "deb [signed-by=/etc/apt/keyrings/eza.gpg] http://deb.gierens.de stable main" | tee /etc/apt/sources.list.d/eza.list && \
    apt-get update && \
    apt-get install -y eza

# Install Tmux Plugin Manager (TPM)
RUN git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm

# Install Node.js and related tools
RUN curl -fsSL https://deb.nodesource.com/setup_${NODE_VERSION}.x | bash - && \
    apt-get update && \
    apt-get install -y nodejs && \
    npm install -g npm@latest && \
    npm install -g tree-sitter-cli && \
    npm install -g neovim

# Download and install Neovim
RUN if [ "$TARGETARCH" = "arm64" ]; then \
      export NVIM_ARCH="arm64"; \
    else \
      export NVIM_ARCH="x86_64"; \
    fi && \
    wget -q https://github.com/neovim/neovim/releases/download/v${NVIM_VERSION}/nvim-linux-${NVIM_ARCH}.tar.gz && \
    tar -zxf nvim-linux-${NVIM_ARCH}.tar.gz && \
    cp -r nvim-linux-${NVIM_ARCH}/bin/nvim /usr/bin/ && \
    cp -r nvim-linux-${NVIM_ARCH}/lib/nvim /usr/lib/ && \
    cp -r nvim-linux-${NVIM_ARCH}/share/nvim /usr/share/ && \
    rm -rf nvim-linux-${NVIM_ARCH}.tar.gz nvim-linux-${NVIM_ARCH}

# Download and install Lazygit
RUN if [ "$TARGETARCH" = "arm64" ]; then \
      export LG_ARCH="arm64"; \
    else \
      export LG_ARCH="x86_64"; \
    fi && \
    export LAZYGIT_VERSION=$(curl -s "https://api.github.com/repos/jesseduffield/lazygit/releases/latest" | grep -Po '"tag_name": "v\K[^"]*') && \
    curl -Lo lazygit.tar.gz "https://github.com/jesseduffield/lazygit/releases/latest/download/lazygit_${LAZYGIT_VERSION}_Linux_${LG_ARCH}.tar.gz" && \
    tar xf lazygit.tar.gz lazygit && \
    mv lazygit /usr/local/bin/ && \
    rm lazygit.tar.gz

# Install My personal dotfiles 
RUN git clone https://github.com/caffbit/dotfiles.git ~/.config && \
    rm -rf ~/.config/nvim/.git

# Clean up unnecessary files to reduce image size
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    rm -rf /tmp/* && \
    rm -rf ~/.cache

# Set default command
CMD ["bash"]

# Add metadata
LABEL description="Optimized Neovim ${NVIM_VERSION} with Node.js ${NODE_VERSION}"

# Use:
# - docker build -t nvim-node:0.11.1-node22 .
# - docker run -d --name <name> [-p <host_port>:<container_port>] -v application:/apps nvim-node:0.11.1-node22 tail -f /dev/null
# - docker exec -it <name> bash