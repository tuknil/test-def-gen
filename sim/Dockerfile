# syntax=docker/dockerfile:1.7
ARG NODE_IMAGE=artifact.it.att.com/att-dkr-proxy-group/node:20

FROM ${NODE_IMAGE} AS dependencies
WORKDIR /app
COPY package.json package-lock.json ./

ENV http_proxy=http://proxy.conexus.svc.local:3128
ENV https_proxy=http://proxy.conexus.svc.local:3128
ENV HTTP_PROXY=http://proxy.conexus.svc.local:3128
ENV HTTPS_PROXY=http://proxy.conexus.svc.local:3128
ENV NO_PROXY=localhost,127.0.0.1,0.0.0.0,vault.azure.net,169.254.169.254
ENV no_proxy=localhost,127.0.0.1,0.0.0.0,vault.azure.net,169.254.169.254

RUN --mount=type=secret,id=jfrog_username \
    --mount=type=secret,id=jfrog_access_token \
    export NODE_ENV=development \
    && export VITE_DEPLOY_ENV=development \
    && echo "registry=https://artifact.it.att.com/artifactory/api/npm/apm0012256-npm-group/" > .npmrc \
    && echo "//artifact.it.att.com/artifactory/api/npm/apm0012256-npm-group/:_authToken=$(cat /run/secrets/jfrog_access_token)" >> .npmrc \
    && echo "email=$(cat /run/secrets/jfrog_username)" >> .npmrc \
    && echo "always-auth=true" >> .npmrc \
    && echo "strict-ssl=false" >> .npmrc \
    && echo "NODE_TLS_REJECT_UNAUTHORIZED=0" >> .npmrc \
    && npm ci


FROM dependencies AS build
COPY . .
RUN npm run build

FROM ${NODE_IMAGE} AS runtime
USER root
RUN --mount=type=secret,id=corporate_ca,required=false \
    if [ -s /run/secrets/corporate_ca ]; then \
      cat /run/secrets/corporate_ca >> /etc/ssl/certs/ca-certificates.crt; \
    fi

WORKDIR /app
ENV HOSTNAME=0.0.0.0 \
    NODE_ENV=production \
    PORT=8099

COPY --from=build --chown=node:node /app/package.json ./package.json
COPY --from=build --chown=node:node /app/package-lock.json ./package-lock.json
COPY --from=build --chown=node:node /app/lib ./lib
COPY --from=build --chown=node:node /app/server.js ./server.js
COPY --from=build --chown=node:node /app/src ./src

USER node
EXPOSE 8099
HEALTHCHECK --interval=15s --timeout=5s --start-period=20s --retries=3 \
  CMD node -e "fetch('http://127.0.0.1:8099/health').then(r => { if (!r.ok) process.exit(1) }).catch(() => process.exit(1))"

CMD ["node", "server.js"]