FROM node:20-alpine

ENV NODE_ENV=production
WORKDIR /app

COPY package*.json ./
RUN npm install --omit=dev && npm cache clean --force

COPY app.js ./

EXPOSE 3000

CMD ["node", "app.js"]
