FROM        debian:jessie
MAINTAINER  Dmitry Shapovalov

ARG         hostuid=966
ARG         hostgid=966

ENV         DEBIAN_FRONTEND noninteractive
ENV         RUN_USER artifactory
ENV         RUN_GROUP artifactory
ENV         RUN_PORT 8006
ENV         JAVA_HOME /usr/lib/jvm/java-8-oracle
ENV         DB_NAME artdb
ENV         DB_USER_NAME artifactory
ENV         CATALINA_HOME /usr/share/tomcat8
ENV         CATALINA_BASE /var/lib/tomcat8
ENV         JDBC_DOWNLOAD_VERSION 8.0.11
ENV         JDBC_DOWNLOAD_NAME mysql-connector-java-$JDBC_DOWNLOAD_VERSION
ENV         JDBC_DOWNLOAD_URL https://dev.mysql.com/get/Downloads/Connector-J/$JDBC_DOWNLOAD_NAME.zip
ENV         ARTIFACTORY_HOME /etc/artifactory
ENV         ARTIFACTORY_DOWNLOAD_VERSION 5.3.0
ENV         ARTIFACTORY_DOWNLOAD_NAME jfrog-artifactory-oss-$ARTIFACTORY_DOWNLOAD_VERSION
ENV         ARTIFACTORY_ARCHIVED_FOLDER_NAME artifactory-oss-$ARTIFACTORY_DOWNLOAD_VERSION
ENV         ARTIFACTORY_DOWNLOAD_URL https://bintray.com/jfrog/artifactory/download_file?file_path=$ARTIFACTORY_DOWNLOAD_NAME.zip

COPY        . /tmp

# Install packages
RUN         echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" >> /etc/apt/sources.list.d/java.list && \
            echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" >> /etc/apt/sources.list.d/java.list && \
            echo "oracle-java8-installer shared/accepted-oracle-license-v1-1 select true" | /usr/bin/debconf-set-selections && \
            apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys EEA14886 && \
            apt-get update && \
            apt-get install --assume-yes \
                mysql-server \
                oracle-java8-installer \
                oracle-java8-set-default \
                tomcat8 \
                unzip \
                vim \
                wget && \
            apt-get clean

# Create user and group
RUN         groupadd -g $hostgid $RUN_GROUP && \
            useradd -u $hostuid -g $hostgid -M $RUN_USER

# Create folders for artifactory
RUN         mkdir /artifactory && \
            mkdir /artifactory/db && \
            mkdir /artifactory/data && \
            mkdir /artifactory/access && \
            mkdir /artifactory/scripts && \
            mkdir /artifactory/logs && \
            mkdir /artifactory/backups && \
            chown -R $RUN_USER:$RUN_GROUP /artifactory

# Configure MySQL
RUN         sed -i 's|user.*|user            = '"$RUN_USER"'|' /etc/mysql/my.cnf && \
            sed -i 's|datadir.*|datadir         = /artifactory/db|' /etc/mysql/my.cnf && \
            sed -i 's|key_buffer.*|key_buffer_size         = 16M|' /etc/mysql/my.cnf && \
            sed -i 's|max_allowed_packet.*|max_allowed_packet      = 8M|' /etc/mysql/my.cnf && \
            sed -i 's|myisam-recover.*|myisam-recover-options  = BACKUP|' /etc/mysql/my.cnf && \
            sed -i 's|log_error.*|log_error                = /var/log/mysql/dberror.log|' /etc/mysql/my.cnf && \
            sed -i '\|\[mysqld_safe\]|a log_error       = /var/log/mysql/dberror.log' /etc/mysql/my.cnf && \
            sed -i '\|# Read the manual for more InnoDB related options..*|r /tmp/mysql.conf' /etc/mysql/my.cnf && \
            rm /tmp/mysql.conf && \
            sed -i 's|syslog.*|# syslog|' /etc/mysql/conf.d/mysqld_safe_syslog.cnf && \
            rm -rf /var/lib/mysql && \
            rm -rf /var/log/mysql && \
            ln -s /artifactory/logs /var/log/mysql && \
            chown -h $RUN_USER:$RUN_GROUP /var/log/mysql && \
            mkdir -p /var/run/mysqld && \
            chown -R $RUN_USER:$RUN_GROUP /var/run/mysqld

# Configure Tomcat
RUN         mkdir -p /etc/tomcat8 && \
            chown -R $RUN_USER:$RUN_GROUP /etc/tomcat8 && \
            sed -i 's|port="8080"|port="'"$RUN_PORT"'"|' /etc/tomcat8/server.xml && \
            ln -s /etc/tomcat8 $CATALINA_HOME/conf && \
            chown -h $RUN_USER:$RUN_GROUP $CATALINA_HOME/conf && \
            mkdir $CATALINA_HOME/common && \
            mkdir $CATALINA_HOME/common/classes && \
            chown -R $RUN_USER:$RUN_GROUP $CATALINA_HOME/common && \
            mkdir $CATALINA_HOME/server && \
            mkdir $CATALINA_HOME/server/classes && \
            chown -R $RUN_USER:$RUN_GROUP $CATALINA_HOME/server && \
            mkdir $CATALINA_HOME/shared && \
            mkdir $CATALINA_HOME/shared/classes && \
            chown -R $RUN_USER:$RUN_GROUP $CATALINA_HOME/shared && \
            ln -s /artifactory/logs $CATALINA_HOME/logs && \
            chown -h $RUN_USER:$RUN_GROUP $CATALINA_HOME/logs && \
            mkdir -p /var/cache/tomcat8 && \
            chown -R $RUN_USER:$RUN_GROUP /var/cache/tomcat8 && \
            chown -h $RUN_USER:$RUN_GROUP $CATALINA_BASE/conf && \
            mkdir -p $CATALINA_BASE/lib && \
            chown -R $RUN_USER:$RUN_GROUP $CATALINA_BASE/lib && \
            rm -rf $CATALINA_BASE/logs && \
            ln -s /artifactory/logs $CATALINA_BASE/logs && \
            chown -h $RUN_USER:$RUN_GROUP $CATALINA_BASE/logs && \
            ln -s /tmp $CATALINA_BASE/temp && \
            chown -h $RUN_USER:$RUN_GROUP $CATALINA_BASE/temp && \
            mkdir -p $CATALINA_BASE/webapps && \
            chown -R $RUN_USER:$RUN_GROUP $CATALINA_BASE/webapps && \
            chown -h $RUN_USER:$RUN_GROUP $CATALINA_BASE/work && \
            rm -rf /var/log/tomcat8 && \
            ln -s /artifactory/logs /var/log/tomcat8 && \
            chown -h $RUN_USER:$RUN_GROUP /var/log/tomcat8

# Configure artifactory
RUN         cd /tmp && \
            wget -O $JDBC_DOWNLOAD_NAME.zip $JDBC_DOWNLOAD_URL && \
            unzip -q $JDBC_DOWNLOAD_NAME.zip -d /tmp && \
            rm $JDBC_DOWNLOAD_NAME.zip && \
            mv $JDBC_DOWNLOAD_NAME/$JDBC_DOWNLOAD_NAME.jar $CATALINA_BASE/lib && \
            chown $RUN_USER:$RUN_GROUP $CATALINA_BASE/lib/$JDBC_DOWNLOAD_NAME.jar && \
            rm -rf $JDBC_DOWNLOAD_NAME && \
            wget -O $ARTIFACTORY_DOWNLOAD_NAME.zip $ARTIFACTORY_DOWNLOAD_URL && \
            unzip -q $ARTIFACTORY_DOWNLOAD_NAME.zip -d /tmp && \
            rm $ARTIFACTORY_DOWNLOAD_NAME.zip && \
            mkdir $CATALINA_BASE/webapps/artifactory && \
            unzip -q $ARTIFACTORY_ARCHIVED_FOLDER_NAME/webapps/artifactory.war -d $CATALINA_BASE/webapps/artifactory && \
            chown -R $RUN_USER:$RUN_GROUP $CATALINA_BASE/webapps/artifactory && \
            mkdir $ARTIFACTORY_HOME && \
            mv $ARTIFACTORY_ARCHIVED_FOLDER_NAME/etc $ARTIFACTORY_HOME && \
            mv $ARTIFACTORY_ARCHIVED_FOLDER_NAME/misc $ARTIFACTORY_HOME && \
            rm -rf $ARTIFACTORY_ARCHIVED_FOLDER_NAME && \
            ln -s /artifactory/data $ARTIFACTORY_HOME/data && \
            ln -s /artifactory/access $ARTIFACTORY_HOME/access && \
            ln -s /artifactory/logs $ARTIFACTORY_HOME/logs && \
            ln -s /artifactory/backups $ARTIFACTORY_HOME/backup && \
            chown -R -h $RUN_USER:$RUN_GROUP $ARTIFACTORY_HOME && \
            cp $ARTIFACTORY_HOME/misc/db/mysql.properties $ARTIFACTORY_HOME/etc/db.properties && \
            sed -i 's|url=.*|url=jdbc:mysql://localhost:3306/'"$DB_NAME"'?characterEncoding=UTF-8\&elideSetAutoCommitts=true\&serverTimezone=UTC|' $ARTIFACTORY_HOME/etc/db.properties && \
            sed -i 's|username.*|username='"$DB_USER_NAME"'|' $ARTIFACTORY_HOME/etc/db.properties

# Configure helper scripts
RUN         mv /tmp/initialize /artifactory/scripts && \
            chmod a+x /artifactory/scripts/initialize && \
            mv /tmp/changeRootPassword /artifactory/scripts && \
            chmod a+x /artifactory/scripts/changeRootPassword && \
            mv /tmp/changeUserPassword /artifactory/scripts && \
            chmod a+x /artifactory/scripts/changeUserPassword && \
            mv /tmp/backup /artifactory/scripts && \
            chmod a+x /artifactory/scripts/backup && \
            mv /tmp/restore /artifactory/scripts && \
            chmod a+x /artifactory/scripts/restore && \
            mv /tmp/startup /artifactory/scripts && \
            chmod a+x /artifactory/scripts/startup && \
            mv /tmp/shutdown /artifactory/scripts && \
            chmod a+x /artifactory/scripts/shutdown

USER        $RUN_USER

EXPOSE      $RUN_PORT

ENTRYPOINT  ["/artifactory/scripts/startup"]
