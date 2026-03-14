ARG BASE_IMAGE=lmsysorg/sglang:dev
FROM ${BASE_IMAGE}

ENV DEBIAN_FRONTEND=noninteractive
ARG TARGETARCH
ARG VSCODE_QUALITY=stable
ARG VSCODE_SERVER_DIR=/root/.vscode-server

# Add the development tools that should always exist on top of the upstream image.
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    ca-certificates \
    curl \
    dnsutils \
    git \
    htop \
    iproute2 \
    iputils-ping \
    jq \
    less \
    net-tools \
    procps \
    tmux \
    vim \
    wget \
    && rm -rf /var/lib/apt/lists/*

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
    curl -fsSL "${vscode_url}" -o /tmp/vscode-server.tar.gz; \
    tar -xzf /tmp/vscode-server.tar.gz -C "${VSCODE_SERVER_DIR}/bin/${vscode_commit}" --strip-components=1; \
    rm /tmp/vscode-server.tar.gz; \
    "${VSCODE_SERVER_DIR}/bin/${vscode_commit}/bin/code-server" \
      --force \
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
