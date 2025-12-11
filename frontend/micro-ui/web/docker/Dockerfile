FROM egovio/alpine-node-builder-14:yarn AS build
#FROM ghcr.io/egovernments/alpine-node-builder-14:yarn AS build

RUN apk update && apk upgrade
RUN apk add --no-cache git>2.30.0

ARG WORK_DIR
WORKDIR /app

ENV NODE_OPTIONS="--max-old-space-size=8500"
ENV GENERATE_SOURCEMAP="false"

# Copy the micro-ui TL source into /app
COPY ${WORK_DIR} .
RUN ls -lah

# Build the micro-ui bundle
RUN cd web/ \
    && node envs.js \
    && node -e 'console.log(v8.getHeapStatistics().heap_size_limit/(1024*1024))' \
    && ./install-deps.sh \
    && yarn install \
    && yarn build:webpack

# ---------------- Runtime image ----------------
FROM nginx:mainline-alpine
#FROM ghcr.io/egovernments/nginx:mainline-alpine

# Serve TL UI from /var/web/digit-ui-tl (matches homepage="/digit-ui-tl")
ENV WORK_DIR=/var/web/digit-ui-tl

RUN mkdir -p ${WORK_DIR}

# Copy the built TL UI bundle
COPY --from=build /app/web/build ${WORK_DIR}/

# Nginx vhost configuration for TL UI
COPY --from=build /app/web/docker/nginx.conf /etc/nginx/conf.d/default.conf

