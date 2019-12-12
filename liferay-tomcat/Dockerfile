FROM openjdk:8u171-jdk-alpine3.8

ENV LIFERAY_HOME=/opt/liferay
ENV DOWNLOADS=/etc/liferay/downloads

ENV BUNDLE_BASE=liferay-dxp-7.1.10-ga1
ENV BUNDLE_FILE=liferay-dxp-tomcat-7.1.10-ga1-20180703090613030.zip

ENV TOMCAT_HOME=$LIFERAY_HOME/tomcat-9.0.6

RUN apk update && \
	apk add curl bash tar tree unzip && \
	adduser -D -h /etc/liferay liferay && \ 
	addgroup liferay liferay && \
	mkdir -p $LIFERAY_HOME && \
	chown -R liferay:liferay $LIFERAY_HOME

WORKDIR $LIFERAY_HOME 

USER liferay:liferay

RUN mkdir -p $DOWNLOADS

COPY ./$BUNDLE_FILE $DOWNLOADS

RUN unzip -q $DOWNLOADS/$BUNDLE_FILE -d $DOWNLOADS && \
	mv $DOWNLOADS/$BUNDLE_BASE/* . && \
	rm -rf $DOWNLOADS

COPY --chown=liferay:liferay ./config/portal-ext.properties $LIFERAY_HOME/portal-ext.properties
COPY --chown=liferay:liferay ./config/activation-key-digitalenterprisedevelopment-7.1-liferaycom.xml $LIFERAY_HOME/deploy/activation-key-digitalenterprisedevelopment-7.1-liferaycom.xml

# Configure Liferay to point to an external Elasticsearch container for production use
#COPY --chown=liferay:liferay ./config/com.liferay.portal.search.elasticsearch6.configuration.ElasticsearchConfiguration.config $LIFERAY_HOME/osgi/configs/com.liferay.portal.search.elasticsearch6.configuration.ElasticsearchConfiguration.config

# Configure the AdvancedFileSystemStore
#COPY ./config/com.liferay.portal.store.file.system.configuration.AdvancedFileSystemStoreConfiguration.config $LIFERAY_HOME/osgi/configs/com.liferay.portal.store.file.system.configuration.AdvancedFileSystemStoreConfiguration.config

# Configure cluster link to use JDBC PING so that the cluster network works in the cloud
#COPY --chown=liferay:liferay ./config/jdbc_ping.xml $TOMCAT_HOME/webapps/ROOT/WEB-INF/classes/jdbc_ping.xml

# Configure the TunnelAuthVerifier for live remote staging
COPY ./config/com.liferay.portal.security.auth.verifier.internal.tunnel.module.configuration.TunnelAuthVerifierConfiguration-default.cfg $LIFERAY_HOME/osgi/configs/com.liferay.portal.security.auth.verifier.internal.tunnel.module.configuration.TunnelAuthVerifierConfiguration-default.cfg

COPY --chown=liferay:liferay ./config/wait-for-it.sh /etc/liferay/wait-for-it.sh
RUN chmod +x /etc/liferay/wait-for-it.sh

CMD /etc/liferay/wait-for-it.sh -s -t 60 mysql:3306 -- $TOMCAT_HOME/bin/catalina.sh run
