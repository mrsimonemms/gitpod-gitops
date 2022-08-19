FROM gitpod/workspace-base
ENV KUBESEAL_VERSION=0.18.1
WORKDIR /tmp
# Install kubeseal
RUN wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v${KUBESEAL_VERSION}/kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz \
  && tar -xvzf kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz kubeseal \
  && sudo install -m 755 kubeseal /usr/local/bin/kubeseal \
  && kubeseal --version
WORKDIR $HOME
# Install Helm
COPY --from=alpine/helm /usr/bin/helm /usr/bin/helm
RUN helm version
# Install YQ
COPY --from=mikefarah/yq:4 /usr/bin/yq /usr/bin/yq
RUN yq --version
