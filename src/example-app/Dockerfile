FROM node:18-alpine
WORKDIR /app

RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:$PATH"

COPY package.json bun.lock ./
COPY src/example-app/package.json ./src/example-app/

RUN bun install --frozen-lockfile
COPY src/example-app/ ./src/example-app/

RUN cd src/example-app && bun run build

EXPOSE 3002
CMD ["bun", "run", "--cwd", "src/example-app", "start"]
