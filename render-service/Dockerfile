# Base images are pinned by digest (@sha256): the tag is only a label.
# To update one: docker buildx imagetools inspect <tag> and paste the new digest.
FROM node:18-bookworm-slim@sha256:f9ab18e354e6855ae56ef2b290dd225c1e51a564f87584b9bd21dd651838830e
COPY --from=python:3.11-slim@sha256:65bd3d9f1d7fa268dd1909973ba7eb4bdf41f6dcf2cfbe76ba9528b170f3fd79 /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt

# Install Chromium and FFmpeg for Remotion rendering
RUN find /etc/apt -type f \( -name '*.list' -o -name '*.sources' \) -exec sed -i 's|http://deb.debian.org|https://deb.debian.org|g' {} + && apt-get update && \
    apt-get install -y --no-install-recommends chromium ffmpeg && \
    rm -rf /var/lib/apt/lists/*

ENV PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium
ENV REMOTION_BUNDLE_PATH=/app/remotion
ENV OUTPUT_DIR=/output

WORKDIR /app

# Copy render service source
COPY render-service/package.json render-service/package-lock.json ./
RUN npm ci --no-audit --no-fund

COPY render-service/tsconfig.json ./
COPY render-service/src/ ./src/

# Copy remotion source for bundling
COPY remotion/package.json remotion/package-lock.json /app/remotion/
COPY remotion/tsconfig.json /app/remotion/tsconfig.json
COPY remotion/src/ /app/remotion/src/
COPY remotion/public/ /app/remotion/public/

# Install remotion deps
RUN cd /app/remotion && npm ci --no-audit --no-fund

# Build TypeScript
RUN npm run build

EXPOSE 3100
CMD ["node", "dist/server.js"]
