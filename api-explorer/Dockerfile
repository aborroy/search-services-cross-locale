FROM tomcat:8.5.20-jre8-alpine

RUN apk update && apk add wget && rm -rf /var/cache/apk/*

RUN cd /usr/local/tomcat/webapps && \
wget https://artifacts.alfresco.com/nexus/service/local/repositories/releases/content/org/alfresco/api-explorer/6.1.0/api-explorer-6.1.0.war \
-O api-explorer.war
