# Build from the monorepo root: docker build --build-arg APP=<package-name> -f <this file> .
# Each service needs a `build` script that outputs dist/index.js.
FROM node:22-slim AS build
ARG APP
WORKDIR /repo
RUN corepack enable
COPY . .
RUN pnpm dlx turbo prune "$APP" --docker
WORKDIR /repo/out/json
RUN pnpm install --frozen-lockfile
WORKDIR /repo/out/full
RUN cp -R ../json/node_modules . && pnpm turbo run build --filter="$APP"
RUN pnpm --filter="$APP" deploy --prod /app

FROM node:22-slim
WORKDIR /app
ENV NODE_ENV=production PORT=8080
COPY --from=build /app ./
USER 1000
EXPOSE 8080
CMD ["node", "dist/index.js"]
