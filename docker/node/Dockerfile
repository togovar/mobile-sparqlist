# ベースイメージとしてNode.jsを使用
FROM node:18

# 作業ディレクトリを設定
WORKDIR /usr/src/app

# 依存関係をインストール
RUN npm install

# ポート3000を開放
EXPOSE 8080

# アプリケーションを起動
CMD ["npm", "start"]
