FROM node:22-bookworm-slim

ENV DEBIAN_FRONTEND=noninteractive \
    PLAYWRIGHT_BROWSERS_PATH=/ms-playwright \
    LANGSMITH_TRACING=false \
    LANGCHAIN_TRACING_V2=false

RUN apt-get update \
    && apt-get install -y --no-install-recommends ca-certificates git python3 python3-venv build-essential \
    && rm -rf /var/lib/apt/lists/* \
    && npm install --global pnpm@11.25.0

# Retain source, tests and development dependencies for an offline audit.
COPY . /src
WORKDIR /src
RUN pnpm install --frozen-lockfile \
    && pnpm exec playwright install --with-deps chromium --only-shell \
    && python3 -m venv /opt/aop-sdk \
    && /opt/aop-sdk/bin/python -m pip install --no-cache-dir ./sdk/python

RUN pnpm typecheck && pnpm build && pnpm test && pnpm test:browser

# The runtime starts only when the auditor explicitly invokes it.
CMD ["bash"]
