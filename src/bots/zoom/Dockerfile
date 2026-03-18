FROM --platform=linux/amd64 ghcr.io/puppeteer/puppeteer:24.7.2

ENV LANG=en_US.UTF-8 \
    PPTRUSER_UID=10042 \
    DOCKER_MEETING_PLATFORM=zoom

USER root

# Install bun
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:$PATH"

WORKDIR /home/pptruser

COPY package.json bun.lock ./
COPY src/bots/package.json ./src/bots/
COPY src/bots/zoom/package.json ./src/bots/zoom/

RUN bun install --frozen-lockfile
RUN bunx puppeteer browsers install chrome

RUN chown -R pptruser:pptruser /home/pptruser

COPY src/bots/src ./src/bots/src
COPY src/bots/zoom/src ./src/bots/zoom/src

RUN chown -R pptruser:pptruser /home/pptruser/src && chmod -R 755 /home/pptruser/src

USER pptruser
EXPOSE 3000
CMD ["bun", "run", "dev"]
