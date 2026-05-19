FROM node:20-slim

RUN apt-get update \
    && apt-get install -y --no-install-recommends git ca-certificates \
    && rm -rf /var/lib/apt/lists/*

RUN npm install -g @anthropic-ai/claude-code

WORKDIR /workspace
COPY package.json tsconfig.json ./
RUN npm install

COPY . .

CMD ["bash"]
