FROM oven/bun:1.4.4-slim
WORKDIR /app
ENV NODE_ENV=production PORT=8080
COPY package.json bun.lock ./
RUN bun install --frozen-lockfile --production
COPY . .
USER 1000
EXPOSE 8080
CMD ["bun", "run", "src/index.ts"]
