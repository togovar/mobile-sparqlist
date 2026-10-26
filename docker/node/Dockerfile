# Supported LTS release. Pin the major line so security patch releases are
# picked up by regular image rebuilds.
FROM node:24-bookworm-slim

# 作業ディレクトリを設定
WORKDIR /usr/src/app

# package.json と package-lock.json をコピー
COPY --chown=node:node package*.json ./

# 依存関係をインストール
RUN npm ci --omit=dev --ignore-scripts \
    && npm cache clean --force

USER node

# ポート3000を開放
EXPOSE 8080

# アプリケーションを起動
CMD ["npm", "start"]
