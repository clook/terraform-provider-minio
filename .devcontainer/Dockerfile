FROM minio/minio:RELEASE.2020-01-25T02-50-51Z as minio
FROM hashicorp/terraform:0.12.20 as terraform
FROM golang:1

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Set data for non-root user creation
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Configure the behavior of Go Modules
ENV GO111MODULE=auto

# Copy and install Terraform binary
COPY --from=terraform /bin/terraform /usr/local/bin/

# Define environment variables for MinIO
ENV MINIO_ACCESS_KEY=minio
ENV MINIO_SECRET_KEY=minio123
ENV MINIO_HTTP_TRACE=/dev/stdout
ENV MINIO_VOLUMES=/data

# Copy and install MinIO binary
COPY --from=minio /usr/bin/minio /usr/local/bin/

# Update packages lists
RUN apt-get update \
  # Install utils
  && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
  #
  # Verify git, process tools, lsb-release (common in install instructions for CLIs) installed
  && apt-get -y install git openssh-client iproute2 procps lsb-release \
  #
  # Install Go tools w/module support
  && mkdir -p /tmp/gotools \
  && cd /tmp/gotools \
  && GO111MODULE=on go get -v -u golang.org/x/tools/gopls@latest 2>&1 \
  && GO111MODULE=on go get -v \
  honnef.co/go/tools/...@latest \
  golang.org/x/tools/cmd/gorename@latest \
  golang.org/x/tools/cmd/goimports@latest \
  golang.org/x/tools/cmd/guru@latest \
  golang.org/x/lint/golint@latest \
  github.com/mdempsky/gocode@latest \
  github.com/cweill/gotests/...@latest \
  github.com/haya14busa/goplay/cmd/goplay@latest \
  github.com/sqs/goreturns@latest \
  github.com/josharian/impl@latest \
  github.com/davidrjenni/reftools/cmd/fillstruct@latest \
  github.com/uudashr/gopkgs/cmd/gopkgs@latest  \
  github.com/ramya-rao-a/go-outline@latest  \
  github.com/acroca/go-symbols@latest  \
  github.com/godoctor/godoctor@latest  \
  github.com/rogpeppe/godef@latest  \
  github.com/zmb3/gogetdoc@latest \
  github.com/fatih/gomodifytags@latest  \
  github.com/mgechev/revive@latest  \
  github.com/go-delve/delve/cmd/dlv@latest 2>&1 \
  #
  # Give all permissions to Go Path
  && chmod -R 777 $GOPATH \
  #
  # Install Go tools w/o module support
  && go get -v github.com/alecthomas/gometalinter 2>&1 \
  #
  # Install gocode-gomod
  && go get -x -d github.com/stamblerre/gocode 2>&1 \
  && go build -o gocode-gomod github.com/stamblerre/gocode \
  && mv gocode-gomod $GOPATH/bin/ \
  #
  # Install golangci-lint
  && curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOPATH)/bin 2>&1 \
  #
  # Create a non-root user - see https://aka.ms/vscode-remote/containers/non-root-user
  && groupadd --gid $USER_GID $USERNAME \
  && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
  #
  # Add sudo support
  && apt-get install -y sudo \
  && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
  && chmod 0440 /etc/sudoers.d/$USERNAME \
  #
  # Create folders for MinIO
  && mkdir -p ${MINIO_VOLUMES} && chmod -R 777 ${MINIO_VOLUMES} \
  #
  # Create folders for Terraform
  && mkdir -p /home/${USERNAME}/.terraform.d/plugins \
  && chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/.terraform.d \
  #
  # Clean up
  && apt-get autoremove -y \
  && apt-get clean -y \
  && rm -rf /var/lib/apt/lists/* /go/src /tmp/gotools

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog

# Define the default command to run
CMD bash -c "minio server ${MINIO_VOLUMES} &" \
  && sh -c "while sleep 1000; do :; done"
