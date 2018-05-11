FROM        debian:jessie
MAINTAINER  Dmitry Shapovalov

ARG         hostuid=966
ARG         hostgid=966
ARG         guestport=8006
ARG         guestappname=artifactory

ENV         DEBIAN_FRONTEND noninteractive
ENV         RUN_USER artifactory
ENV         RUN_GROUP artifactory
ENV         APACHE_RUN_USER $RUN_USER
ENV         APACHE_RUN_GROUP $RUN_GROUP
ENV         GUEST_PORT $guestport
ENV         GUEST_APP_NAME $guestappname
ENV         DB_NAME artdb
ENV         DB_USER_NAME artifactory
ENV         JAVA_HOME /usr/lib/jvm/java-8-oracle
ENV         CATALINA_HOME /usr/share/tomcat8
ENV         CATALINA_BASE /var/lib/tomcat8
ENV         ARTIFACTORY_DOWNLOAD_VERSION 5.11.0
ENV         ARTIFACTORY_DOWNLOAD_NAME jfrog-artifactory-pro-$ARTIFACTORY_DOWNLOAD_VERSION
ENV         ARTIFACTORY_DOWNLOAD_URL https://bintray.com/jfrog/artifactory-pro/download_file?file_path=org%2Fartifactory%2Fpro%2Fjfrog-artifactory-pro%2F$ARTIFACTORY_DOWNLOAD_VERSION%2F$ARTIFACTORY_DOWNLOAD_NAME.zip

COPY        . /tmp

# Install packages
RUN         echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" >> /etc/apt/sources.list.d/java.list && \
            echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" >> /etc/apt/sources.list.d/java.list && \
            echo "oracle-java8-installer shared/accepted-oracle-license-v1-1 select true" | /usr/bin/debconf-set-selections && \
            apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys EEA14886 && \
            apt-get update && \
            apt-get install --assume-yes \
                apache2 \
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
RUN         chown -R $RUN_USER:$RUN_GROUP /etc/tomcat8 && \
            ln -s /etc/tomcat8 /usr/share/tomcat8/conf && \
            chown -h $RUN_USER:$RUN_GROUP /usr/share/tomcat8/conf && \
            mkdir /usr/share/tomcat8/common && \
            mkdir /usr/share/tomcat8/common/classes && \
            chown -R $RUN_USER:$RUN_GROUP /usr/share/tomcat8/common && \
            mkdir /usr/share/tomcat8/server && \
            mkdir /usr/share/tomcat8/server/classes && \
            chown -R $RUN_USER:$RUN_GROUP /usr/share/tomcat8/server && \
            mkdir /usr/share/tomcat8/shared && \
            mkdir /usr/share/tomcat8/shared/classes && \
            chown -R $RUN_USER:$RUN_GROUP /usr/share/tomcat8/shared && \
            ln -s /artifactory/logs /usr/share/tomcat8/logs && \
            chown -h $RUN_USER:$RUN_GROUP /usr/share/tomcat8/logs && \
            chown -R $RUN_USER:$RUN_GROUP /var/cache/tomcat8 && \
            chown -h $RUN_USER:$RUN_GROUP /var/lib/tomcat8/conf && \
            chown -R $RUN_USER:$RUN_GROUP /var/lib/tomcat8/lib && \
            rm -rf /var/lib/tomcat8/logs && \
            ln -s /artifactory/logs /var/lib/tomcat8/logs && \
            chown -h $RUN_USER:$RUN_GROUP /var/lib/tomcat8/logs && \
            ln -s /tmp /var/lib/tomcat8/temp && \
            chown -h $RUN_USER:$RUN_GROUP /var/lib/tomcat8/temp && \
            chown -R $RUN_USER:$RUN_GROUP /var/lib/tomcat8/webapps && \
            chown -h $RUN_USER:$RUN_GROUP /var/lib/tomcat8/work && \
            rm -rf /var/log/tomcat8 && \
            ln -s /artifactory/logs /var/log/tomcat8 && \
            chown -h $RUN_USER:$RUN_GROUP /var/log/tomcat8

# Configure apache
RUN         sed -i 's|:80>|:'"$GUEST_PORT"'>|' /etc/apache2/sites-available/000-default.conf && \
            sed -i 's|#ServerName www.example.com|ServerName localhost|' /etc/apache2/sites-available/000-default.conf && \
            mv /tmp/artifactory.conf /etc/apache2/sites-available && \
            sed -i 's|Listen 80|Listen '"$GUEST_PORT"'|' /etc/apache2/ports.conf && \
            echo "ServerName localhost" > /etc/apache2/conf-available/servername.conf && \
            a2enconf servername && \
            a2enmod proxy proxy_http && \
            a2ensite artifactory && \
            rm -rf /var/log/apache2 && \
            ln -s /artifactory/logs /var/log/apache2 && \
            chown -h $RUN_USER:$RUN_GROUP /var/log/apache2 && \
            mkdir -p /var/lock/apache2 && \
            chown -R $RUN_USER:$RUN_GROUP /var/lock/apache2 && \
            mkdir -p /var/run/apache2 && \
            chown -R $RUN_USER:$RUN_GROUP /var/run/apache2

# Configure artifactory
RUN         cd /tmp && \
            wget -q -O $ARTIFACTORY_DOWNLOAD_NAME.zip ${ARTIFACTORY_DOWNLOAD_URL} && \
            unzip $ARTIFACTORY_DOWNLOAD_NAME.zip -d /tmp

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

EXPOSE      $GUEST_PORT

ENTRYPOINT  ["/artifactory/scripts/startup"]
