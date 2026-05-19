FROM php:8.2-cli

RUN apt-get update \
    && apt-get install -y --no-install-recommends curl ca-certificates git unzip \
    && curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && apt-get install -y --no-install-recommends nodejs \
    && rm -rf /var/lib/apt/lists/*

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

RUN npm install -g @anthropic-ai/claude-code

WORKDIR /workspace
COPY composer.json ./
RUN composer install --no-interaction

COPY . .

CMD ["bash"]
