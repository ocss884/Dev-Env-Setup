ARG BASE_IMAGE=lmsysorg/sglang:dev
FROM ${BASE_IMAGE}

ENV DEBIAN_FRONTEND=noninteractive
ENV PATH=/root/.local/bin:${PATH}
ARG TARGETARCH
ARG VSCODE_QUALITY=stable
ARG VSCODE_SERVER_DIR=/root/.vscode-server
ARG VSCODE_EXTENSIONS_DIR=/root/.vscode-server/extensions

# Add the development tools that should always exist on top of the upstream image.
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    ca-certificates \
    curl \
    dnsutils \
    duf \
    git \
    htop \
    iproute2 \
    iputils-ping \
    jq \
    less \
    net-tools \
    npm \
    pipx \
    procps \
    tmux \
    unzip \
    vim \
    wget \
    && rm -rf /var/lib/apt/lists/*

RUN npm install -g @openai/codex @anthropic-ai/claude-code \
    && pipx install nvitop

RUN set -eux; \
    case "${TARGETARCH}" in \
      amd64) \
        yazi_arch="x86_64-unknown-linux-musl"; \
        lazygit_arch="x86_64"; \
        ;; \
      arm64) \
        yazi_arch="aarch64-unknown-linux-musl"; \
        lazygit_arch="arm64"; \
        ;; \
      *) echo "Unsupported TARGETARCH: ${TARGETARCH}" >&2; exit 1 ;; \
    esac; \
    yazi_version="$(curl -fsSL https://api.github.com/repos/sxyazi/yazi/releases/latest | jq -r '.tag_name')"; \
    lazygit_version="$(curl -fsSL https://api.github.com/repos/jesseduffield/lazygit/releases/latest | jq -r '.tag_name')"; \
    curl -fsSL "https://github.com/sxyazi/yazi/releases/download/${yazi_version}/yazi-${yazi_arch}.zip" -o /tmp/yazi.zip; \
    unzip -q /tmp/yazi.zip -d /tmp/yazi; \
    install -m 0755 "/tmp/yazi/yazi-${yazi_arch}/yazi" /usr/local/bin/yazi; \
    install -m 0755 "/tmp/yazi/yazi-${yazi_arch}/ya" /usr/local/bin/ya; \
    curl -fsSL "https://github.com/jesseduffield/lazygit/releases/download/${lazygit_version}/lazygit_${lazygit_version#v}_linux_${lazygit_arch}.tar.gz" -o /tmp/lazygit.tar.gz; \
    tar -xzf /tmp/lazygit.tar.gz -C /tmp lazygit; \
    install -m 0755 /tmp/lazygit /usr/local/bin/lazygit; \
    rm -rf /tmp/yazi /tmp/yazi.zip /tmp/lazygit /tmp/lazygit.tar.gz

# Install the latest VS Code Server for the requested quality and pre-install
# the editor extensions needed in the dev image.
RUN set -eux; \
    case "${TARGETARCH}" in \
      amd64) vscode_arch="x64" ;; \
      arm64) vscode_arch="arm64" ;; \
      *) echo "Unsupported TARGETARCH: ${TARGETARCH}" >&2; exit 1 ;; \
    esac; \
    mkdir -p "${VSCODE_SERVER_DIR}"; \
    vscode_json="$(curl -fsSL "https://update.code.visualstudio.com/api/update/linux-${vscode_arch}/${VSCODE_QUALITY}/latest")"; \
    vscode_commit="$(printf '%s' "${vscode_json}" | jq -r '.version')"; \
    vscode_url="$(printf '%s' "${vscode_json}" | jq -r '.url')"; \
    mkdir -p "${VSCODE_SERVER_DIR}/bin/${vscode_commit}"; \
    mkdir -p "${VSCODE_EXTENSIONS_DIR}"; \
    curl -fsSL "${vscode_url}" -o /tmp/vscode-server.tar.gz; \
    tar -xzf /tmp/vscode-server.tar.gz -C "${VSCODE_SERVER_DIR}/bin/${vscode_commit}" --strip-components=1; \
    rm /tmp/vscode-server.tar.gz; \
    "${VSCODE_SERVER_DIR}/bin/${vscode_commit}/bin/code" \
      --force \
      --user-data-dir "${VSCODE_SERVER_DIR}/data" \
      --extensions-dir "${VSCODE_EXTENSIONS_DIR}" \
      --install-extension ms-python.python \
      --install-extension ms-python.vscode-pylance \
      --install-extension ms-python.debugpy \
      --install-extension ms-python.vscode-python-envs \
      --install-extension charliermarsh.ruff \
      --install-extension ms-toolsai.jupyter \
      --install-extension ms-toolsai.vscode-jupyter-cell-tags \
      --install-extension ms-toolsai.vscode-jupyter-slideshow \
      --install-extension ms-toolsai.jupyter-keymap \
      --install-extension ms-toolsai.jupyter-renderers \
      --install-extension ms-vscode.cpptools \
      --install-extension ms-vscode.makefile-tools \
      --install-extension ms-vscode.cmake-tools \
      --install-extension eamodio.gitlens \
      --install-extension github.vscode-pull-request-github \
      --install-extension yzhang.markdown-all-in-one \
      --install-extension tamasfe.even-better-toml \
      --install-extension tht13.rst-vscode \
      --install-extension drain99.perfetto-trace \
      --install-extension openai.chatgpt \
      --install-extension github.copilot-chat

CMD ["/bin/bash"]
