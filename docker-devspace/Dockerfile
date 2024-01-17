FROM docker:19

RUN wget --output-document=/usr/local/bin/devspace https://github.com/loft-sh/devspace/releases/latest/download/devspace-linux-amd64 \
      && chmod +x /usr/local/bin/devspace \

RUN apk update && apk add bash