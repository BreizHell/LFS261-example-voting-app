# ========== base ===========
FROM node:22-alpine AS base

# add curl for healthcheck
RUN apk add --no-cache curl

WORKDIR /usr/src/app

COPY package*.json .

RUN npm ci
RUN npm cache clean --force

COPY . .

ENV PORT=80
EXPOSE 80

# ========== dev ===========
FROM base AS dev

# TODO: use npm script to launch with nodemon

CMD ["node", "server.js"]

# ========== dist ===========
FROM base AS dist

CMD ["node", "server.js"]