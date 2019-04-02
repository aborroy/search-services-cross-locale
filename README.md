# Alfresco Search Services for Cross Locale Users

Deployment template based in official [Docker Composition](https://github.com/Alfresco/acs-community-deployment/tree/master/docker-compose) but configured for cross locale environments.

In a *cross locale* environment, content locale is multiple and users are uploading content using different browser locales.

You should review volumes, configuration, modules & tuning parameters before using this composition in **Production** environments.

## Source Images

* [alfresco-content-repository-community:6.1.2-ga](https://hub.docker.com/r/alfresco/alfresco-content-repository-community)
* [alfresco-share:6.1.0](https://hub.docker.com/r/alfresco/alfresco-share)
* [alfresco-search-services:1.3.0.1](https://hub.docker.com/r/alfresco/alfresco-search-services)
* [alfresco-activemq:5.15.6](https://hub.docker.com/r/alfresco/alfresco-activemq)
* [alfresco-content-app:latest](https://hub.docker.com/r/alfresco/alfresco-content-app/)
* **alfresco-api-explorer 6.1.0**: Custom build
* [nginx:stable-alpine](https://hub.docker.com/_/nginx)
* [postgres:10.1](https://hub.docker.com/_/postgres)

## Volumes

Three `volume` directories are available for configuration, data and log files.
By default only configuration and SOLR log configuration is provided.

```bash
config
├── nginx.conf
├── shared.properties     >>> Cross locale SOLR Configuration
└── solrconfig.xml        >>> Cross locale SOLR Configuration
logs
└── solr
    └── log4j.properties
```

**Data** wil be persisted automatically in `data` folder. Once launched, Docker will create three subfolders for following services:

* `alf-repo-data` for Content Store
* `postgres-data` for Database
* `solr-data` for Indexes

>> For Linux hosts, set `solr-data` folder permissions to user with UID 1000, as `alfresco-search-services` is using an container user named `solr` with UID 1000.

```bash
$ docker exec -it 9a11 sh

$ ls -l /opt/alfresco-search-services/
drwxr-xr-x 5 solr solr  160 Apr  2 09:12 data

$ id -u solr
1000
```

**Logs** folder includes log files for:

* `alfresco` contains Tomcat repository logs
* `nginx` contains HTTP Proxy logs
* `postgres` contains database logs
* `share` contains Tomcat share logs
* `solr` contains SOLR logs and `log4j.properties` configuration file

## SOLR Considerations

Alfresco SOLR API has been protected to be accessed from outside Docker network. You can enable this URLs removing following lines at [nginx.conf](https://github.com/keensoft/docker-alfresco/blob/master/volumes/config/nginx.conf)

```
    # Protect access to SOLR APIs
    location ~ ^(/.*/service/api/solr/.*)$ {return 403;}
    location ~ ^(/.*/s/api/solr/.*)$ {return 403;}
    location ~ ^(/.*/wcservice/api/solr/.*)$ {return 403;}
    location ~ ^(/.*/wcs/api/solr/.*)$ {return 403;}

    location ~ ^(/.*/proxy/alfresco/api/solr/.*)$ {return 403 ;}
    location ~ ^(/.*/-default-/proxy/alfresco/api/.*)$ {return 403;}  
```

SOLR Web Console (http://localhost/solr) access has **not** been protected.

## Locale configuration

This repository is pre-configured for English (en) and Spanish (es) locales, but different languages can be configured by modifying following block in `config/solrconfig.xml` file.

```xml
<queryParser name="afts" class="org.alfresco.solr.query.AlfrescoFTSQParserPlugin">
    <str name="rerankPhase">QUERY_PHASE</str>
    <str name="autoDetectQueryLocale">true</str>
    <str name="autoDetectQueryLocales">en,es</str>
</queryParser>
```

# How to use this composition

## Start Docker

Start docker and check the ports are correctly bound.

```bash
$ docker-compose up -d
$ docker ps --format '{{.Names}}\t{{.Image}}\t{{.Ports}}'
activemq_1      alfresco/alfresco-activemq:5.15.6                       0.0.0.0:5672->5672/tcp, 0.0.0.0:8161->8161/tcp, 0.0.0.0:61613->61613/tcp, 0.0.0.0:61616->61616/tcp
postgres_1      postgres:10.1                                           0.0.0.0:5432->5432/tcp
content-app_1   alfresco/alfresco-content-app:latest                    0.0.0.0:8084->80/tcp
alfresco_1      alfresco/alfresco-content-repository-community:6.1.2-ga 0.0.0.0:8082->8080/tcp
share_1         alfresco/alfresco-share:6.1.0                           8000/tcp, 0.0.0.0:8080->8080/tcp
proxy_1         nginx:stable-alpine                                     0.0.0.0:80->80/tcp
api-explorer_1  search-services-cross-locale_api-explorer               0.0.0.0:8085->8080/tcp
solr6_1         alfresco/alfresco-search-services:1.3.0.1               0.0.0.0:8083->8983/tcp
```

### Viewing System Logs

You can view the system logs by issuing the following.

```bash
$ docker-compose logs -f
```

Logs for every service are also available at `logs` folder.

## Access

Use the following username/password combination to login.

 - User: admin
 - Password: admin

Alfresco and related web applications can be accessed from the below URIs when the servers have started.

```
http://localhost/              - Alfresco Content Application (sample ADF application)
http://localhost/share         - Alfresco Share
http://localhost/alfresco      - Alfresco Repository
http://localhost/solr          - Alfresco Search Services
http://localhost/api-explorer  - OpenAPIs sample Web Client and documentation for Alfresco REST API
```
