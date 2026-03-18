FROM mcr.microsoft.com/playwright:v1.52.0-jammy AS base

RUN apt-get update && apt-get install -y \
    xvfb ffmpeg x11-utils pulseaudio x11-xserver-utils fluxbox \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Install bun
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:$PATH"

# Copy workspace config from repo root
COPY package.json bun.lock ./
COPY src/bots/package.json ./src/bots/
COPY src/bots/meet/package.json src/bots/meet/entrypoint.sh ./src/bots/meet/

RUN sed -i 's/\r$//' ./src/bots/meet/entrypoint.sh

# Install dependencies
RUN bun install --frozen-lockfile

# Install Playwright
RUN bunx playwright@1.52.0 install-deps
RUN bunx playwright@1.52.0 install --with-deps
RUN mkdir -p /root/ms-playwright && chmod -R 777 /root/ms-playwright

# ======================================================
FROM mcr.microsoft.com/playwright:v1.52.0-jammy AS runtime

ENV DOCKER_MEETING_PLATFORM=meet
WORKDIR /app

RUN useradd -ms /bin/bash meetbot && chown -R meetbot:meetbot /app

RUN apt-get update && apt-get install -y --no-install-recommends \
    xvfb pulseaudio pulseaudio-utils xserver-xephyr x11-xserver-utils \
    && rm -rf /var/lib/apt/lists/*

# Install bun in runtime
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:$PATH"

COPY --from=base /app/node_modules /app/node_modules
COPY --from=base /app/package.json /app/package.json
COPY --from=base /app/src/bots/node_modules /app/src/bots/node_modules
COPY --from=base /app/src/bots/package.json /app/src/bots/package.json
COPY --from=base /app/src/bots/meet/node_modules /app/src/bots/meet/node_modules
COPY --from=base /app/src/bots/meet/package.json /app/src/bots/meet/package.json
COPY --from=base /root/ms-playwright /root/ms-playwright
COPY --from=base /app/src/bots/meet/entrypoint.sh /app/entrypoint.sh

RUN chown -R meetbot:meetbot /app
RUN chown -R meetbot:meetbot /root/ms-playwright

COPY src/bots/src ./src/bots/src
COPY src/bots/meet/src ./src/bots/meet/src
COPY src/bots/test.env ./
RUN chown -R meetbot:meetbot ./src

ENV DISPLAY=:99
USER meetbot
RUN chmod +x ./entrypoint.sh
CMD ["./entrypoint.sh"]
