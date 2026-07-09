FROM ubuntu:26.04 AS build-stage


RUN --mount=target=/var/lib/apt/lists,type=cache,sharing=locked \
    --mount=target=/var/cache/apt,type=cache,sharing=locked \
    apt update 
RUN --mount=target=/var/lib/apt/lists,type=cache,sharing=locked \
    --mount=target=/var/cache/apt,type=cache,sharing=locked \ 
    apt install git wget curl quilt python3 build-essential gcc make libkrb5-dev pkg-config libx11-dev libxkbfile-dev jq -y



# https://github.com/nvm-sh/nvm
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.5/install.sh | bash
ENV NVM_DIR=/root/.nvm
# install node
ARG NODE_VERSION=24.18.0
RUN bash -c "source $NVM_DIR/nvm.sh && nvm install $NODE_VERSION"
RUN bash -c "source $NVM_DIR/nvm.sh && nvm use $NODE_VERSION"

ENV NODE_PATH=$NVM_DIR/versions/node/v$NODE_VERSION/lib/node_modules
ENV PATH=$NVM_DIR/versions/node/v$NODE_VERSION/bin:$PATH
WORKDIR /code-server


COPY ci ./ci
COPY test test
COPY typings typings
COPY package*.json .
COPY lib/vscode lib/vscode
RUN --mount=type=cache,target=/root/.npm npm i
COPY .git .git
RUN VERSION='0.0.0' npm run build:vscode

COPY . .

RUN npm run build



FROM ubuntu:26.04 AS vscode-image


RUN apt update 
RUN apt install git wget curl quilt python3 build-essential gcc make -y
# https://github.com/nvm-sh/nvm
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.5/install.sh | bash
ENV NVM_DIR=/root/.nvm
# install node
ARG NODE_VERSION=24.18.0
RUN bash -c "source $NVM_DIR/nvm.sh && nvm install $NODE_VERSION"
RUN bash -c "source $NVM_DIR/nvm.sh && nvm use $NODE_VERSION"

ENV NODE_PATH=$NVM_DIR/versions/node/v$NODE_VERSION/lib/node_modules
ENV PATH=$NVM_DIR/versions/node/v$NODE_VERSION/bin:$PATH

WORKDIR /code-tug

COPY --from=build-stage /code-server/lib/vscode-web lib/vscode-web
COPY --from=build-stage /code-server/out out
COPY docker docker
EXPOSE 8080

ENTRYPOINT [ "docker/entrypoint.sh" ] 