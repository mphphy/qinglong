FROM node:18-slim as nodebuilder

FROM python:3.10-slim-bullseye as builder
COPY . /tmp/build
COPY --from=nodebuilder /usr/local/bin/node /usr/local/bin/
COPY --from=nodebuilder /usr/local/lib/node_modules/. /usr/local/lib/node_modules/
RUN set -x && \
  ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm && \
  apt update && \
  apt install --no-install-recommends -y libatomic1 && \
  npm i -g pnpm@8.3.1 && \
  cd /tmp/build && \
  pnpm --registry https://registry.npmmirror.com install && \
  pnpm run build:front && \
  pnpm run build:back

FROM python:3.10-slim-bullseye

ARG QL_MAINTAINER="whyour"
LABEL maintainer="${QL_MAINTAINER}"
ARG QL_URL=https://github.com/${QL_MAINTAINER}/qinglong.git
ARG QL_BRANCH=develop

ENV PNPM_HOME=/root/.local/share/pnpm \
  PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/root/.local/share/pnpm:/root/.local/share/pnpm/global/5/node_modules:$PNPM_HOME \
  NODE_PATH=/usr/local/bin:/usr/local/pnpm-global/5/node_modules:/usr/local/lib/node_modules:/root/.local/share/pnpm/global/5/node_modules \
  LANG=C.UTF-8 \
  SHELL=/bin/bash \
  PS1="\u@\h:\w \$ " \
  QL_DIR=/ql \
  QL_BRANCH=${QL_BRANCH}

COPY --from=nodebuilder /usr/local/bin/node /usr/local/bin/
COPY --from=nodebuilder /usr/local/lib/node_modules/. /usr/local/lib/node_modules/
COPY . ${QL_DIR}

WORKDIR ${QL_DIR}

RUN set -x && \
  ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm && \
  apt update && \
  apt upgrade -y && \
  apt install --no-install-recommends -y git \
  curl \
  wget \
  tzdata \
  perl \
  openssl \
  openssh-client \
  nginx \
  jq \
  procps \
  netcat \
  libatomic1 && \
  apt clean && \
  ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
  echo "Asia/Shanghai" >/etc/timezone && \
  git config --global user.email "qinglong@@users.noreply.github.com" && \
  git config --global user.name "qinglong" && \
  git config --global http.postBuffer 524288000 && \
  npm install -g pnpm@8.3.1 && \
  pnpm config set registry https://registry.npmmirror.com && \
  pnpm add -g pm2 tsx && \
  pnpm install --prod && \
  rm -rf /root/.pnpm-store && \
  rm -rf /root/.local/share/pnpm/store && \
  rm -rf /root/.cache && \
  rm -rf /root/.npm

RUN chmod +x ${QL_DIR}/shell/*.sh && \
    chmod +x ${QL_DIR}/docker/*.sh && \
    mkdir -p ${QL_DIR}/static

COPY --from=builder /tmp/build/static ${QL_DIR}/static

ENTRYPOINT ["./docker/docker-entrypoint.sh"]
