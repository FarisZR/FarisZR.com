---
title: تثبيت ownCloud infinite scale مع docker compose
categories:
  - guides
keywords:
  - Owncloud infinite scale
  - OCIS
  - nextcloud alternative
  - LibreOffice
image: thumbnail.jpg
slug: owncloud-infinite-scale-docker-setup
description: شرح بسيط حول تثبيت OCIS باستخدام docker compose مع خدمات اضافية apache Taki لبحث افضل و LibreOffice.
date: 2024-07-31
canonicalurl: https://discourse.aosus.org/t/topic/3387
---

برمجية ownCloud infinite scale هي برمجية جديدة لتستبدل الإصدار العادي من ownCloud و Nextcloud المكتوب بPHP.
OCIS مكتوب بلغة Go واسرع بكثير, لدرجة انه لا يحتاج لقاعدة بيانات!

وجدت ان شروحاتهم ترتيبها غريب قليلا, لذلك كتبت هذا الشرح ليسهل تثبيت OCIS على الخوادم المنزلية مع docker

## طريقة التثبيت
كما اسم ownCloud infinite scale يوضح, OCIS قابل للتوسع و يمكن تثبيته على عده خوادم لتحمل عدد اكبر من المستخدمين, مع فصل الخِدْمَات الداخلية ليتم توسيعها حسب الاستخدام.
في هذا الشرح سوف اركز على طريقة التثبيت المناسبة للخوادم المنزلية و الاستخدام المحلي.

حتى اذا كان هدفك تثبيت OCIS في بيئة عمل كبيرة, هذه الطريقة بداية جيده لفهم طريقة عمل OCIS.

## مِلَفّ docker-compose لOCIS

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
هذا الأعداد الاولي لOCIS مع استخدام مجلد منفصل للthumbnails, ما يمكنك من تخزينها بشكل منفصل من البيانات الأساسية على SSD مثلا.

ستحتاج لتثبيت Reverse proxy لتشغيل OCIS, مثل Caddy او Nginx او [Nginx proxy manager](https://discourse.aosus.org/t/topic/2148)

هذه الإعدادات تتضمن كل خِدْمَات OCIS الأساسية, بإمكانك ان تستخدمه هكذا دون أصافات اذا كان كل ما تحتاجه هو فقط تخزين الملفات.

لكن بإمكانك أضافه بعض المميزات والتحسينات عبر اضافه برمجيات اختياريه:

## تفعيل البحث داخل محتوى الملفات
افتراضيا يبني OCIS فِهْرِس البحث بناءا على اسم الملف او المجلد ومحتوى الملفات النصية.
لكن بإمكانك تحسين البحث ليشمل محتوى ملفات عبر إضافة Apache tika الذي يستخدم OCR لقرائه محتوى ملفات PDF عبر التعرف تلقائيا على لغة المحتوى, للأسف التعرف التلقائي لا يدعم العربية ([قائمة اللغات المدعومة](https://www.javatpoint.com/tika-language-detection)).

**هذا المِلَفّ يحتوي فقط على التغييرات التي يجب اضافتها للاعدادات الأولية في القسم السابق**
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
    # command: apt update && apt install tesseract-ocr-xxx -y # add additional languages (only works if Tika supports auto-detection for that language)
    logging:
      driver: local
```

## فحص تلقائي ضد الفيروسات
لحذف الفيروسات تِلْقائيًا عند التعرف عليها في الملفات المرفوعة, يمكن إضافة ClamAV كخدمة إضافية 
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
احد اكبر الميزات في الإصدار السادس من OCIS هي إضافة أمكانية تضمين حُزْمَة تطبيقات المكتبية مثل Collabora (LibreOffice) و OnlyOffice. 

لكن هذا الإصدار ليس اصدار عادي, بل اصدار متدحرج يستهدف المستخدمين الذين يبحثون عن ميزات اكثر, ولا يحصل على نفس مستوى دعم الإصدارات العادية.
لذلك لإضافة تطبيقات المكتبية ستحتاج الى تغيير اصدار OCIS من الإصدار العادي الى الإصدار المتدحرج.

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

أضفنا خدمتين جديده تحتاج لReverse proxy
الاولى OCIS-collaboration التي تكشف المنفذ(port) 9300 و هي المسئولة عن خادم wopi و برمجية Collabora التي تستخدم منفذ 9980 لlibreoffice.

**هناك خطوة إضافية**, سنحتاج لتعديل سياسة امن المحتوى (Content security policy, CSP) للسماح بفتح LibreOffice داخل واجهه الويب لOCIS.


أنشئ مِلَفّ باسم csp.yaml و استخدم [القالب الرسمي](https://github.com/owncloud/ocis/blob/master/deployments/examples/ocis_full/config/ocis/csp.yaml) ليقوم بإضافة رابط collabora للقائمة الروابط المسموحة, تأكد من تغير `https://${COLLABORA_DOMAIN|collabora.owncloud.test}/'` الى الرابط الخاص بك لcollabora.

والآن اصبح لديك خادم OCIS كامل.
هناك خِدْمَات اخرى لم اغطيها بالمقال مثل خدمه cloud import لتسهيل نقل الملفات من خِدْمَات تخزين سحابي اخرى.
توجد قوالب اضافيه لكل الخِدْمَات الأخرى تحت مِلَفّ [ocis_full](https://github.com/owncloud/ocis/tree/master/deployments/examples/ocis_full) في مستودع ownCloud infinite Scale.

الترتيب غريب قليلا, كل مِلَفّ yml هو عباره عن docker-compose واسم المِلَفّ يمثل الخدمة.
تبدأ من مِلَفّ ocis.yml و بقية الملفات هي تكمله, والقوالب تفترض انك تستخدم trafeik.

تحت مجلد config تجد كل الملفات الإضافية للخدمات.

## المصادر 
https://github.com/owncloud/ocis/tree/master/deployments/examples/ocis_full

https://doc.owncloud.com/ocis/next/deployment/services/services.html

https://doc.owncloud.com/ocis/next/deployment/general/general-info.html#infinite-scale-supervised-services