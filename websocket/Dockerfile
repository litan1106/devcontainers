# syntax=docker/dockerfile:1.4
# -------------------------------------------- Build Time Arguments ------------------------------------------------
ARG NODE_IMAGE_VERSION="16-alpine"

# ==================================================================================================================
#                                                  --- NODE ---
# ==================================================================================================================
FROM node:${NODE_IMAGE_VERSION}

ARG WORK_DIR="/var/www/app"

WORKDIR ${WORK_DIR}

ARG LARAVEL_ECHO_SERVER_AUTH_HOST="http://localhost"
ARG LARAVEL_ECHO_SERVER_REDIS_HOST="redis"
ENV LARAVEL_ECHO_SERVER_REDIS_HOST=${LARAVEL_ECHO_SERVER_REDIS_HOST}
ENV LARAVEL_ECHO_SERVER_AUTH_HOST=${LARAVEL_ECHO_SERVER_AUTH_HOST}
ENV NODE_ENV="production"

COPY package.json ${WORK_DIR}/package.json

RUN yarn

COPY laravel-echo-server.json ${WORK_DIR}/laravel-echo-server.json

EXPOSE 6001

CMD ["npm", "start"]
