FROM alpine/k8s:1.26.12

RUN wget --output-document=/usr/local/bin/devspace https://github.com/loft-sh/devspace/releases/latest/download/devspace-linux-amd64 \
      && chmod +x /usr/local/bin/devspace

RUN apk add --update --no-cache bash

