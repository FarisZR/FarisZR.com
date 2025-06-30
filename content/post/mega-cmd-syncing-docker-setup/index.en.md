---
title: Automatically sync files to Mega.io with dockerized mega-cmd
categories:
- quick-snippets
tags:
- docker
- megacmd
- docker images
- mega
date: 2025-06-30
slug: dockerized-mega-cmd-automatic-file-sync
image: thumbnail.webp
---

I self-host my own private cloud, but sometimes I have to make files available on external clouds like Mega.

In this post I will explain how to create a dockerized mega-cmd with an entrypoint script that will automatically sync files and watch for changes.

## dockerfile

To make it easier to build the image, i embedded the entrypoint script inside the dockerfile.

```dockerfile
FROM debian:12

# Install dependencies and MegaCMD
RUN apt-get update && apt-get install -y \
    wget \
    curl \
    gnupg \
    apt-transport-https \
    && mkdir -p /etc/apt/keyrings \
    && curl -fsSL https://mega.nz/keys/MEGA_signing.key | gpg --dearmor -o /etc/apt/keyrings/mega.nz.gpg \
    && echo "deb [arch=amd64,arm64 signed-by=/etc/apt/keyrings/mega.nz.gpg] https://mega.nz/linux/repo/Debian_12/ ./" | tee /etc/apt/sources.list.d/mega.nz.list > /dev/null \
    && apt-get update \
    && apt-get install -y megacmd \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Create directories for configuration and data
RUN mkdir -p /root/.megaCmd /data

# Create startup script
RUN echo '#!/bin/bash\n\
set -e\n\
\n\
# Function to check if already logged in\n\
check_login() {\n\
  mega-whoami > /dev/null 2>&1\n\
  return $?\n\
}\n\
\n\
# Check if already logged in\n\
if check_login; then\n\
  echo "Existing session found. Skipping login."\n\
else\n\
  # Check if credentials are provided\n\
  if [ -z "$MEGA_EMAIL" ] || [ -z "$MEGA_PASSWORD" ]; then\n\
    echo "Error: MEGA_EMAIL and MEGA_PASSWORD environment variables must be set"\n\
    exit 1\n\
  fi\n\
\n\
  # Login to MEGA\n\
  if [ -n "$MEGA_2FA" ]; then\n\
    mega-login $MEGA_EMAIL $MEGA_PASSWORD --auth-code=$MEGA_2FA\n\
  else\n\
    mega-login $MEGA_EMAIL $MEGA_PASSWORD\n\
  fi\n\
fi\n\
\n\
# Show account info to confirm login\n\
echo "Logged in as:"\n\
mega-whoami\n\
\n\
# Set up sync if MEGA_SYNC_LOCAL_PATH and MEGA_SYNC_REMOTE_PATH are provided\n\
if [ ! -z "$MEGA_SYNC_LOCAL_PATH" ] && [ ! -z "$MEGA_SYNC_REMOTE_PATH" ]; then\n\
  echo "Setting up sync from $MEGA_SYNC_LOCAL_PATH to $MEGA_SYNC_REMOTE_PATH"\n\
  mega-sync $MEGA_SYNC_LOCAL_PATH $MEGA_SYNC_REMOTE_PATH\n\
fi\n\
\n\
# Keep the container running\n\
echo "MegaCMD is now running and syncing..."\n\
tail -f /dev/null\n\
' > /entrypoint.sh && chmod +x /entrypoint.sh

# Set the entrypoint
ENTRYPOINT ["/entrypoint.sh"]
```

## docker-compose.yml

```yaml
services:
  megacmd:
    build: .
    restart: always
    environment:
      - MEGA_EMAIL=your@email.com
      - "MEGA_PASSWORD=pass"
      - MEGA_2FA=10101
      - MEGA_SYNC_LOCAL_PATH=/data # directory to sync inside the container
      - MEGA_SYNC_REMOTE_PATH=/remote-dir # directory on mega
      - MEGA_DEVICE_ID=dockerized-mega #device name
    volumes:
      - ~/files:/data
      - ./megacmd-config:/root/.megaCmd
      - /etc/machine-id:/etc/machine-id:ro
```

the authentication variables are only needed for the first run, after that the token is stored in the .megaCmd folder.

And now your files should show up on mega!

Keep in mind mega support only two-way sync, mounting the data as read only isn't an option due to megacmd checking for write-access on startup.