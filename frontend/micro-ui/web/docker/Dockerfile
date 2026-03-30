FROM node:14.21.3-alpine AS build

RUN apk update && apk upgrade \
    && apk add --no-cache git bash

ARG WORK_DIR
WORKDIR /app

ENV NODE_OPTIONS="--max-old-space-size=8500"
ENV GENERATE_SOURCEMAP="false"

COPY ${WORK_DIR} .
RUN ls -lah

RUN cd web/ \
    && node --version \
    && yarn --version \
    && node envs.js \
    && node -e 'console.log(v8.getHeapStatistics().heap_size_limit/(1024*1024))' \
    && chmod +x install-deps.sh \
    && ./install-deps.sh \
    && yarn install \
    && yarn build:webpack

FROM nginx:mainline-alpine

ENV WORK_DIR="/var/web/digit-ui"

RUN mkdir -p "${WORK_DIR}"

COPY --from=build /app/web/build "${WORK_DIR}/"
COPY --from=build /app/web/docker/nginx.conf /etc/nginx/conf.d/default.conf
