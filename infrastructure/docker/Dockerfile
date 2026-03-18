FROM public.ecr.aws/amazonlinux/amazonlinux:2023

ARG HELM_VERSION=3.16.4
ARG CODER_VERSION=2.30.2
ARG KUBECTL_VERSION=latest
ARG TERRAFORM_VERSION=latest

# Core packages
RUN dnf install -y \
    python3.12 \
    python3.12-pip \
    git \
    curl \
    unzip \
    jq \
    tar \
    gzip \
    shadow-utils \
    && dnf clean all

# AWS CLI v2
RUN curl -fsSL "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip \
    && unzip -q awscliv2.zip \
    && ./aws/install \
    && rm -rf awscliv2.zip aws

# kubectl
RUN if [ "$KUBECTL_VERSION" = "latest" ]; then \
      KUBECTL_VERSION=$(curl -fsSL https://storage.googleapis.com/kubernetes-release/release/stable.txt); \
    fi \
    && curl -fsSL "https://storage.googleapis.com/kubernetes-release/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl" \
       -o /usr/local/bin/kubectl \
    && chmod +x /usr/local/bin/kubectl

# eksctl
RUN curl -fsSL "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" \
    | tar xz -C /usr/local/bin

# Helm
RUN curl -fsSL "https://get.helm.sh/helm-v${HELM_VERSION}-linux-amd64.tar.gz" \
    | tar xz -C /tmp \
    && mv /tmp/linux-amd64/helm /usr/local/bin/helm \
    && rm -rf /tmp/linux-amd64

# Terraform
RUN if [ "$TERRAFORM_VERSION" = "latest" ]; then \
      TERRAFORM_VERSION=$(curl -fsSL https://api.github.com/repos/hashicorp/terraform/releases/latest \
        | jq -r '.tag_name' | sed 's/v//'); \
    fi \
    && curl -fsSL "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip" \
       -o /tmp/terraform.zip \
    && unzip -q /tmp/terraform.zip -d /usr/local/bin \
    && chmod +x /usr/local/bin/terraform \
    && rm /tmp/terraform.zip

# Coder CLI
RUN curl -fsSL https://coder.com/install.sh | sh -s -- --version ${CODER_VERSION} --method standalone

# Verify all installs
RUN echo "=== Installed versions ===" \
    && aws --version \
    && kubectl version --client \
    && eksctl version \
    && helm version --short \
    && terraform version \
    && coder version \
    && python3.12 --version \
    && git --version

ENTRYPOINT ["/bin/bash", "-c"]
