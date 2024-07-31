---
title: Setup Owncloud infinite scale with docker compose
categories:
  - Guides
keywords:
  - Owncloud infinite scale
  - OCIS
  - nextcloud alternative
  - Libreoffice
  - Full text search
image: thumbnail.jpg
slug: owncloud-infinite-scale-docker-setup
summary: a quick guide on setting up ownCloud infinite scale using docker compose with additional services like Apache Taki for better full search and Libreoffice.
date: 2024-07-31
---

ownCloud infinite scale is a complete rewrite of the classic ownCloud/Nextcloud server in go, it's much faster and much more efficient, to the point it doesn't even need a database!

I found their docs to be a bit confusing, but I managed to figure it out and wrote this guide to save you the hassle!

## Setup plan
ownCloud infinite scale is, as the name says, very scalable, you could deploy multiple instances, with services separated from the main program to be able to scale depending on usage and so on.

This guide is focused on the most common setup for Homelabs and small scale NAS's, local UNIX storage with one instance using the embedded supervisor.
It's also a great starting point if you are planning a bigger deployment.

## Initial ownCloud setup

```yaml
services:
  ocis:
    image: owncloud/ocis:5
    user: 1000:1000
    ports:
      - 127.0.0.1:9200:9200
    entrypoint:
      - /bin/sh
    # run ocis init to initialize a configuration file with random secrets
    # it will fail on subsequent runs, because the config file already exists
    # therefore we ignore the error and then start the ocis server
    command: ["-c", "ocis init || true; ocis server"]
    environment:
      OCIS_URL: https://owncloud.yourdomain.com
      OCIS_LOG_LEVEL: error # make oCIS less verbose
      PROXY_TLS: false # do not use SSL between reverse proxy and oCIS
      OCIS_INSECURE: true
      # basic auth (not recommended, but needed for eg. WebDav clients that do not support OpenID Connect)
      PROXY_ENABLE_BASIC_AUTH: false
      # admin user password
      IDM_ADMIN_PASSWORD: "admin" # this overrides the admin password from the configuration file
      # make settings service available to oCIS Hello
      SETTINGS_GRPC_ADDR: 0.0.0.0:9191
      GATEWAY_GRPC_ADDR: 0.0.0.0:9142 # make the REVA gateway accessible to the app drivers
      # email server (if configured)
      NOTIFICATIONS_SMTP_HOST: "xxxxxx"
      NOTIFICATIONS_SMTP_PORT: "xxxx"
      NOTIFICATIONS_SMTP_SENDER: "xxxxx"
      NOTIFICATIONS_SMTP_USERNAME: "xxxxxxxx"
      NOTIFICATIONS_SMTP_INSECURE: "xxxxxxx"
      # PROXY_TLS is set to "false", the download url has no https
      STORAGE_USERS_DATA_GATEWAY_URL: http://ocis:9200/data
      # separate directory for thumbnails
      THUMBNAILS_FILESYSTEMSTORAGE_ROOT: /var/lib/ocis-thumbnails
    volumes:
      - ./ocis-config:/etc/ocis
      - ./ocis-data:/var/lib/ocis
      - ./thumbnails:/var/lib/ocis-thumbnails
    logging:
      driver: "local"
    restart: always
```

This is the basic setup for OCIS running on port 9200, with a separate directory for thumbnails (useful for speeding up thumbnail loading by storing them on an SSD) and override for the admin password.

You need a reverse proxy for this to work properly, you could use Caddy or Nginx for this.

This includes all the basic ownCloud services, you could just continue to use it like this if you feel it's adequate for your use case.

But you could add a few bells and whistles to your instance by adding a few optional services:

## Enable full text search
By default the text search in OCIS only indexes file and folder names and the content of raw and Markdown text files.
You can enable Full text search by adding Apache Tika, which will use OCR to read PDF file content and auto-detect its language ([list of supported languages](https://www.javatpoint.com/tika-language-detection)) to build a search index of file contents.

```yaml
services:
  ocis:
    environment: #add this to your yaml file!
      # better fulltext search
      SEARCH_EXTRACTOR_TYPE: tika
      SEARCH_EXTRACTOR_TIKA_TIKA_URL: http://tika:9998
      FRONTEND_FULL_TEXT_SEARCH_ENABLED: "true"

# better full text search
  tika:
    image: apache/tika:latest-full
    restart: always
    # command: apt update && apt install tesseract-ocr-xxx -y add additional languages (only works if Tika supports auto-detection for that language)
    logging:
      driver: local
```

This is a partial compose file, you should add the environment variables to the OCIS template at the start of the article and the Tika container.

## Automatic anti virus scanning
If you don't like viruses casually hanging up with your files on your server, you can enable the antivirus service using ClamAV as an antivirus scanner.

```yaml
services:
  ocis:
    enviroment:
      # ClamAV setup
      ANTIVIRUS_SCANNER_TYPE: clamav
      ANTIVIRUS_INFECTED_FILE_HANDLING: delete
      ANTIVIRUS_CLAMAV_SOCKET: "/var/run/clamav/clamd.sock"
      # enable the antivirus service
      OCIS_ADD_RUN_SERVICES: "antivirus"
      # configure the antivirus service
      POSTPROCESSING_STEPS: "virusscan"
    volumes:
      - clamav-socket:/var/run/clamav

  clamav:
    image: clamav/clamav:latest
    container_name: clamav
    # restart: always
    # ports:
    #   - '127.0.0.1:3310:3310'
    volumes:
      - ./clamav:/var/lib/clamav
      - clamav-socket:/tmp

volumes:
  clamav-socket:
```

## Online office
One of the latest features in OCIS 6 is the addition of support for online office suites like Collabora (Libreoffice) and OnlyOffice.

However, OCIS 6 is a rolling release, which doesn't enjoy full production support and is intended for "Early adopters".

So you will have to switch to the rolling release for this:

```yaml
services:
  ocis:
    image: owncloud/ocis-rolling:latest # use OCIS-rolling releases
    enviroment:
      NATS_NATS_HOST: 0.0.0.0 # make NATS accessible to ocis-collaboration
      NATS_NATS_PORT: 9233
      # make collabora the secure view app
      FRONTEND_APP_HANDLER_SECURE_VIEW_APP_ADDR: com.owncloud.api.collaboration.Collabora
      # fix CSP for collabora
      PROXY_CSP_CONFIG_FILE_LOCATION: /etc/ocis/csp.yaml
    volumes:
      - ./csp.yaml:/etc/ocis/csp.yaml # modifiy CSP policy to allow loading of Collabora in OCIS web ui
      
# the collaboration service isn't included in the embedded supervisor and needs to be run separately
  ocis-collaboration:
    image: owncloud/ocis-rolling:latest
    user: 1000:1000
    restart: always
    ports:
      - 127.0.0.1:9300:9300 #woopi server port
    depends_on:
      ocis:
        condition: service_started
      collabora:
        condition: service_healthy
    entrypoint:
      - /bin/sh
    command: [ "-c", "ocis collaboration server" ]
    environment:
      COLLABORATION_GRPC_ADDR: 0.0.0.0:9301
      COLLABORATION_HTTP_ADDR: 0.0.0.0:9300
      MICRO_REGISTRY: "nats-js-kv"
      MICRO_REGISTRY_ADDRESS: "ocis:9233"
      COLLABORATION_WOPI_SRC: https://wopiserver.owncloud.yourdomain.com
      COLLABORATION_APP_NAME: "Collabora"
      COLLABORATION_APP_ADDR: https://collabora.owncloud.yourdomain.com
      COLLABORATION_APP_ICON: https://collabora.owncloud.yourdomain.com/favicon.ico
      # COLLABORATION_APP_INSECURE: true
      # COLLABORATION_CS3API_DATAGATEWAY_INSECURE: true
      COLLABORATION_LOG_LEVEL: ${LOG_LEVEL:-info}
    volumes:
      - ./ocis-config:/etc/ocis
    logging:
      driver: local

  collabora:
    image: collabora/code:24.04.5.1.1
    ports:
      - 127.0.0.1:9980:9980
    environment:
      aliasgroup1: https://wopiserver.owncloud.yourdomain.com:443
      DONT_GEN_SSL_CERT: "YES"
      extra_params: --o:ssl.enable=false --o:ssl.termination=true --o:welcome.enable=false --o:net.frame_ancestors=owncloud.yourdomain.com
      username: admin
      password: admin
    cap_add:
      - MKNOD
    logging:
      driver: local
    restart: always
    command: ["bash", "-c", "coolconfig generate-proof-key ; /start-collabora-online.sh"]
    healthcheck:
      test: [ "CMD", "curl", "-f", "http://localhost:9980/hosting/discovery" ]
```

This adds two services which need to be reverse proxied, the OCIS-collaboration service which exposes port 9300 for wopiserver and Collabora which uses port 9980.

**However, this is not all**, we have to also modify the content security policy of ownCloud to allow Libreoffice to load inside ownCloud's web interface.

Create a csp.yaml file and use the [template](https://github.com/owncloud/ocis/blob/master/deployments/examples/ocis_full/config/ocis/csp.yaml) prepared from ownCloud to modify the CSP policy accordingly, make sure to replace `https://${COLLABORA_DOMAIN|collabora.owncloud.test}/'` with the public URL of your Collabora server.

That should be it! You now have an OCIS server ready for small production use in your Homelab!

There are other services I didn't cover like the import service to make it easier to import files from other cloud providers.
You can find more templates under the [ocis_full](https://github.com/owncloud/ocis/tree/master/deployments/examples/ocis_full) directory in ownCloud's OCIS repo.

It's a bit oddly organized, but each yml file is a docker-compose template, named under the service it adds.
You start with OCIS.yml and the other files are the things you have to add to enable the service.
Under the config directory you will find all the additional files used in the docker compose templates.

## sources
https://github.com/owncloud/ocis/tree/master/deployments/examples/ocis_full

https://doc.owncloud.com/ocis/next/deployment/services/services.html

https://doc.owncloud.com/ocis/next/deployment/general/general-info.html#infinite-scale-supervised-services