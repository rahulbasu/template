# Use Ubuntu 24.04 LTS as base image for stability and security
FROM ubuntu:24.04

# ------------------------------ Basic setup ------------------------------

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV SHELL=/bin/zsh
ENV USER=dev
ENV HOME=/home/$USER

# Update package lists and install essential packages
RUN apt-get update && apt-get install -y \
    # Essential utilities
    apt-transport-https \
    build-essential \
    ca-certificates \
    curl \
    git \
    gnupg \
    lsb-release \
    software-properties-common \
    wget \
    # Shell and terminal utilities
    zsh \
    # sudo for the dev user
    sudo \
    # SSL/TLS support
    openssl \
    # Process management
    procps \
    # File utilities
    file \
    unzip \
    # Network utilities (useful for debugging)
    iputils-ping \
    # Clean up apt cache to reduce image size
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Create the non-root user with sudo privileges
RUN groupadd --gid 1000 $USER 2>/dev/null || groupmod -n $USER $(getent group 1000 | cut -d: -f1) \
    && (useradd --uid 1000 --gid 1000 --shell /bin/zsh --create-home $USER 2>/dev/null || usermod -l $USER -d /home/$USER -m $(getent passwd 1000 | cut -d: -f1)) \
    && echo "$USER ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/$USER \
    && chmod 0440 /etc/sudoers.d/$USER

# Switch to the dev user for installations
USER $USER
WORKDIR $HOME

# ------------------------------ Shell setup ------------------------------
ARG ZSH_IN_DOCKER_VERSION=1.2.0
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v${ZSH_IN_DOCKER_VERSION}/zsh-in-docker.sh)" -- \
  -p git \
  -p fzf \
  -a "source /usr/share/doc/fzf/examples/key-bindings.zsh" \
  -a "source /usr/share/doc/fzf/examples/completion.zsh" \
  -a "export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
  -x

# ------------------------------ Python3 & uv ------------------------------
ARG PYTHON_VERSION=3.13

# Install using deadsnakes, then remove files
RUN sudo add-apt-repository ppa:deadsnakes/ppa -y && \
    sudo apt-get update
RUN sudo apt-get install -y --no-install-recommends python${PYTHON_VERSION} && \
    sudo rm -rf /var/lib/apt/lists/*
RUN sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python${PYTHON_VERSION} 1

# Install uv (fast Python package manager)
RUN curl -LsSf https://astral.sh/uv/install.sh | sh \
    && echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> $HOME/.zshrc

# ------------------------------ Rust ------------------------------
ARG RUST_VERSION=1.88.0

# Install Rust and Cargo globally
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --default-toolchain ${RUST_VERSION} --profile default
ENV PATH="$HOME/${USERNAME}/.cargo/bin:$PATH"

# Install common Rust tools
RUN $HOME/.cargo/bin/cargo install cargo-watch cargo-edit

# Create cargo cache directory for bind mount and make it accessible
RUN sudo mkdir -p /usr/local/cargo/registry && sudo chmod 755 /usr/local/cargo/registry

# ----------------------------- npm/pnpm -------------------
RUN curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash - \
    && sudo apt-get install -y nodejs
RUN sudo npm install -g pnpm

# ----------------------------- Coding agents -------------------
# Install Claude Code CLI
RUN sudo npm install -g @anthropic-ai/claude-code
RUN sudo npm install -g @google/gemini-cli

# ------------------------------ Finishing touches ---------------------

# Ensure all tools are in PATH for current session
ENV PATH="$HOME/.cargo/bin:$PATH"

# Set up some useful aliases and environment variables
RUN echo 'alias ll="ls -alF"' >> $HOME/.zshrc \
    && echo 'alias la="ls -A"' >> $HOME/.zshrc \
    && echo 'alias l="ls -CF"' >> $HOME/.zshrc \
    && echo 'alias python="python3"' >> $HOME/.zshrc \
    && echo 'alias pip="python -m pip"' >> $HOME/.zshrc

# Create a workspace directory
RUN mkdir -p $HOME/workspace

# Security: Remove any potentially sensitive files and set proper permissions
RUN sudo find /tmp -type f -delete 2>/dev/null || true \
    && sudo find /var/tmp -type f -delete 2>/dev/null || true

# Set proper ownership and permissions for the user's home directory
RUN sudo chown -R $USER:$USER $HOME \
    && chmod 755 $HOME

# Set the working directory to workspace
WORKDIR $HOME/workspace

# Default command
CMD ["/bin/zsh"]