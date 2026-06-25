# Stage 1 — install dependencies
FROM node:20-alpine AS deps
WORKDIR /app
COPY package.json .
RUN npm install --production

# Stage 2 — runtime image
FROM node:20-alpine
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY index.js .
COPY package.json .
EXPOSE 3000
CMD ["node", "index.js"]
