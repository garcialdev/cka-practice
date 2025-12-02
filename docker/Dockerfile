FROM ubuntu:22.04

# Set noninteractive frontend
ENV DEBIAN_FRONTEND=noninteractive

# Install dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      wget \
      gnupg \
      socat \
      pass \
      libsecret-1-0 \
      ca-certificates \
      xz-utils \
      binutils \
      tar \
      gzip \
      sudo && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Create proton user
RUN useradd -ms /bin/bash proton
USER proton
WORKDIR /home/proton

# Download Proton Bridge deb
ARG VERSION=3.21.2
RUN wget -q https://proton.me/download/bridge/protonmail-bridge_${VERSION}_amd64.deb

# Install Proton Bridge
RUN sudo apt-get install -y --no-install-recommends ./protonmail-bridge_${VERSION}_amd64.deb

# Create password store (optional)
RUN gpg --batch --generate-key /home/proton/gpgparams || true

# Expose SMTP & IMAP
EXPOSE 1025 1143

# Start in CLI mode (interactive login required)
CMD ["bash"]
