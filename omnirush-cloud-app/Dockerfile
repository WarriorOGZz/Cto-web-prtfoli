FROM node:24-bookworm-slim

ENV NODE_ENV=production \
    PORT=3000 \
    HOME=/data/home \
    DATA_DIR=/data/app \
    WORKSPACES_DIR=/data/workspaces \
    COOKIE_SECURE=1

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates curl git python3 make g++ bash unzip gnupg \
    && install -d -m 0755 /etc/apt/keyrings \
    && curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg -o /etc/apt/keyrings/githubcli-archive-keyring.gpg \
    && chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" > /etc/apt/sources.list.d/github-cli.list \
    && apt-get update && apt-get install -y --no-install-recommends gh \
    && rm -rf /var/lib/apt/lists/*

RUN npm install --global omnirush@latest
WORKDIR /app
COPY package.json ./
RUN npm install --omit=dev
COPY . .
RUN mkdir -p /data/home /data/app /data/workspaces \
    && chown -R node:node /app /data
USER node
EXPOSE 3000
CMD ["node", "server.js"]
