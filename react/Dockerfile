FROM ubuntu:latest

# Set build arguments
ARG DEBIAN_FRONTEND=noninteractive
ARG TZ=Asia/Taipei
ARG NODE_VERSION=22
ARG NVIM_VERSION=0.11.2
ARG TARGETARCH

# Set environment variables
ENV LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8 \
    TZ=${TZ}

# Install base system packages and development tools first
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    wget curl git gpg ca-certificates openssh-client \
    locales build-essential unzip luarocks sqlite3 libsqlite3-dev && \
    echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen && \
    locale-gen

# Install Python and Aider dependencies
RUN apt-get install -y --no-install-recommends \
    python3 python3-pip python3-venv \
    # Aider core dependencies
    libportaudio2 pandoc \
    && ln -s /usr/bin/python3 /usr/bin/python

# Install shell and terminal utilities
RUN apt-get install -y --no-install-recommends \
    zsh ripgrep fzf fd-find && \
    ln -s $(which fdfind) /usr/local/bin/fd

# Install Node.js and related tools
RUN curl -fsSL https://deb.nodesource.com/setup_${NODE_VERSION}.x | bash - && \
    apt-get update && \
    apt-get install -y nodejs && \
    npm install -g neovim tree-sitter-cli

# Install Neovim
RUN NVIM_ARCH="$([ "$TARGETARCH" = "arm64" ] && echo "arm64" || echo "x86_64")" && \
    wget -q https://github.com/neovim/neovim/releases/download/v${NVIM_VERSION}/nvim-linux-${NVIM_ARCH}.tar.gz && \
    tar -zxf nvim-linux-${NVIM_ARCH}.tar.gz && \
    cp -r nvim-linux-${NVIM_ARCH}/bin/nvim /usr/bin/ && \
    cp -r nvim-linux-${NVIM_ARCH}/lib/nvim /usr/lib/ && \
    cp -r nvim-linux-${NVIM_ARCH}/share/nvim /usr/share/ && \
    rm -rf nvim-linux-${NVIM_ARCH}.tar.gz nvim-linux-${NVIM_ARCH}

# Install Lazygit
RUN LG_ARCH="$([ "$TARGETARCH" = "arm64" ] && echo "arm64" || echo "x86_64")" && \
    LAZYGIT_VERSION=$(curl -s "https://api.github.com/repos/jesseduffield/lazygit/releases/latest" | grep -Po '"tag_name": "v\K[^"]*') && \
    curl -Lo lazygit.tar.gz "https://github.com/jesseduffield/lazygit/releases/latest/download/lazygit_${LAZYGIT_VERSION}_Linux_${LG_ARCH}.tar.gz" && \
    tar xf lazygit.tar.gz lazygit && \
    mv lazygit /usr/local/bin/ && \
    rm lazygit.tar.gz

# Install mods (CLI for AI models)
RUN mkdir -p /etc/apt/keyrings && \
    curl -fsSL https://repo.charm.sh/apt/gpg.key | gpg --dearmor -o /etc/apt/keyrings/charm.gpg && \
    echo "deb [signed-by=/etc/apt/keyrings/charm.gpg] https://repo.charm.sh/apt/ * *" | tee /etc/apt/sources.list.d/charm.list && \
    apt-get update && \
    apt-get install -y mods

# Install Aider AI with full browser support
RUN python3 -m pip install --no-cache-dir aider-chat[help,browser,playwright] --break-system-packages

# Install Playwright browsers (chromium only, following official)
RUN python3 -m playwright install --with-deps chromium

# Verify Aider and Playwright installation
RUN aider --version && python3 -m playwright --version

# Install zoxide (better cd)
RUN curl -sSfL https://raw.githubusercontent.com/ajeetdsouza/zoxide/main/install.sh | sh

# Install eza-community (modern ls replacement)
RUN mkdir -p /etc/apt/keyrings && \
    wget -qO- https://raw.githubusercontent.com/eza-community/eza/main/deb.asc | gpg --dearmor -o /etc/apt/keyrings/gierens.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/gierens.gpg] http://deb.gierens.de stable main" | tee /etc/apt/sources.list.d/gierens.list && \
    chmod 644 /etc/apt/keyrings/gierens.gpg /etc/apt/sources.list.d/gierens.list && \
    apt-get update && \
    apt-get install -y eza

# Install Oh-My-Zsh
RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended

# Install Pure theme
RUN mkdir -p ~/.zsh && \
    git clone https://github.com/sindresorhus/pure.git ~/.zsh/pure && \
    mkdir -p /usr/local/share/zsh/site-functions && \
    ln -s ~/.zsh/pure/pure.zsh /usr/local/share/zsh/site-functions/prompt_pure_setup && \
    ln -s ~/.zsh/pure/async.zsh /usr/local/share/zsh/site-functions/async

# Install zsh plugins
RUN git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions && \
    git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# Clone personal dotfiles
RUN git clone -b react https://github.com/caffbit/dotfiles.git ~/.config

# Set up gen-commit-lazygit script
RUN mkdir -p ~/.local/bin && \
    cp ~/.config/lazygit/gen-commit-lazygit ~/.local/bin/gen-commit-lazygit && \
    chmod +x ~/.local/bin/gen-commit-lazygit

# Link configuration files
RUN ln -sf ~/.config/zsh/ubuntu.zshrc ~/.zshrc
RUN ln -sf ~/.config/aider/.aider.conf.yml ~/.aider.conf.yml

# Default shell
RUN chsh -s /bin/zsh

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* ~/.cache

# Add metadata
LABEL description="React development environment with Neovim, Node.js, and Aider AI"

CMD ["zsh"]