FROM        debian:stretch
MAINTAINER  Dmitry Shapovalov

ARG         hostuid=966
ARG         hostgid=966

ENV         DEBIAN_FRONTEND=noninteractive \
            RUN_USER=artifactory \
            RUN_GROUP=artifactory \
            RUN_PORT=8006 \
            JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 \
            CATALINA_HOME=/usr/share/tomcat8 \
            CATALINA_BASE=/var/lib/tomcat8 \
            ARTIFACTORY_HOME=/etc/artifactory \
            DB_NAME=artdb \
            DB_USER_NAME=artifactory \
            JDBC_DOWNLOAD_URL="https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-8.0.18.zip" \
            JDBC_DOWNLOAD_NAME="mysql-connector-java-8.0.11" \
            JDBC_JAR_NAME="mysql-connector-java-8.0.11" \
            ARTIFACTORY_DOWNLOAD_URL="https://bintray.com/jfrog/artifactory/download_file?file_path=jfrog-artifactory-oss-5.3.0.zip" \
            ARTIFACTORY_DOWNLOAD_NAME="artifactory-oss-5.3.0" \
            ARTIFACTORY_WAR_NAME="artifactory"

COPY        . /tmp

# Install common packages and configure environment
RUN         echo "deb http://ftp.debian.org/debian stretch-backports main" >> /etc/apt/sources.list.d/backports.list && \
            apt-get update && \
            apt-get install --assume-yes \
                apt-utils \
                build-essential \
                software-properties-common \
                unzip \
                vim \
                wget && \
            apt-get clean

# Create user and group
RUN         groupadd -g $hostgid $RUN_GROUP && \
            useradd -u $hostuid -g $hostgid -M $RUN_USER

# Configure application folders
RUN         mkdir /artifactory && \
            mkdir /artifactory/db && \
            mkdir /artifactory/data && \
            mkdir /artifactory/security && \
            mkdir /artifactory/access && \
            mkdir /artifactory/scripts && \
            mkdir /artifactory/logs && \
            mkdir /artifactory/backups && \
            chown -R $RUN_USER:$RUN_GROUP /artifactory

# Install Java
RUN         apt-get update && \
            apt-get install --assume-yes -t stretch-backports \
                ca-certificates-java \
                openjdk-8-jdk-headless \
                openjdk-8-jre-headless && \
            apt-get clean

# Install MySQL
RUN         apt-get update && \
            apt-get install --assume-yes \
                mysql-client \
                mysql-server && \
            apt-get clean && \
            sed -i '\|\[mysqld\]|a ft_min_word_len = 2' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i 's|user.*|user            = '"$RUN_USER"'|' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i 's|datadir.*|datadir         = /artifactory/db|' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i 's|key_buffer.*|key_buffer_size         = 16M|' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i 's|max_allowed_packet.*|max_allowed_packet      = 8M|' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i 's|myisam-recover.*|myisam-recover-options  = BACKUP|' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i 's|log_error.*|log_error                = /var/log/mysql/dberror.log|' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i '\|\[mysqld_safe\]|a log_error       = /var/log/mysql/dberror.log' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i '\|# Read the manual for more InnoDB related options..*|a innodb_file_per_table' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i '\|innodb_file_per_table|a innodb_log_buffer_size  = 4M' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i '\|innodb_file_per_table|a innodb_log_file_size    = 256M' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i '\|innodb_file_per_table|a max_heap_table_size     = 128M' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i '\|innodb_file_per_table|a tmp_table_size          = 128M' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i '\|innodb_file_per_table|a innodb_buffer_pool_size = 256M' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i 's|syslog.*|# syslog|' /etc/mysql/mariadb.conf.d/50-mysqld_safe.cnf && \
            rm -rf /var/lib/mysql && \
            rm -rf /var/log/mysql && \
            ln -s /artifactory/logs /var/log/mysql && \
            chown -h $RUN_USER:$RUN_GROUP /var/log/mysql && \
            mkdir -p /var/run/mysqld && \
            chown -R $RUN_USER:$RUN_GROUP /var/run/mysqld

# Configure Tomcat
RUN         apt-get update && \
            apt-get install --assume-yes \
                tomcat8 && \
            apt-get clean && \
            mkdir -p /etc/tomcat8 && \
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
            mv $JDBC_DOWNLOAD_NAME/$JDBC_JAR_NAME.jar $CATALINA_BASE/lib && \
            chown $RUN_USER:$RUN_GROUP $CATALINA_BASE/lib/$JDBC_JAR_NAME.jar && \
            rm -rf $JDBC_DOWNLOAD_NAME && \
            wget -O $ARTIFACTORY_DOWNLOAD_NAME.zip $ARTIFACTORY_DOWNLOAD_URL && \
            unzip -q $ARTIFACTORY_DOWNLOAD_NAME.zip -d /tmp && \
            rm $ARTIFACTORY_DOWNLOAD_NAME.zip && \
            mkdir $CATALINA_BASE/webapps/artifactory && \
            unzip -q $ARTIFACTORY_DOWNLOAD_NAME/webapps/$ARTIFACTORY_WAR_NAME.war -d $CATALINA_BASE/webapps/artifactory && \
            chown -R $RUN_USER:$RUN_GROUP $CATALINA_BASE/webapps/artifactory && \
            mkdir $ARTIFACTORY_HOME && \
            mv $ARTIFACTORY_DOWNLOAD_NAME/etc $ARTIFACTORY_HOME && \
            mv $ARTIFACTORY_DOWNLOAD_NAME/misc $ARTIFACTORY_HOME && \
            rm -rf $ARTIFACTORY_DOWNLOAD_NAME && \
            ln -s /artifactory/data $ARTIFACTORY_HOME/data && \
            ln -s /artifactory/security $ARTIFACTORY_HOME/etc/security && \
            ln -s /artifactory/access $ARTIFACTORY_HOME/access && \
            ln -s /artifactory/logs $ARTIFACTORY_HOME/logs && \
            ln -s /artifactory/backups $ARTIFACTORY_HOME/backup && \
            chown -R -h $RUN_USER:$RUN_GROUP $ARTIFACTORY_HOME && \
            cp $ARTIFACTORY_HOME/misc/db/mysql.properties $ARTIFACTORY_HOME/etc/db.properties && \
            sed -i 's|url=.*|url=jdbc:mysql://localhost:3306/'"$DB_NAME"'?characterEncoding=UTF-8\&elideSetAutoCommitts=true\&serverTimezone=UTC|' $ARTIFACTORY_HOME/etc/db.properties && \
            sed -i 's|username.*|username='"$DB_USER_NAME"'|' $ARTIFACTORY_HOME/etc/db.properties

# Configure helper scripts
RUN         mv /tmp/startup /artifactory/scripts && \
            mv /tmp/shutdown /artifactory/scripts && \
            mv /tmp/initialize /artifactory/scripts && \
            mv /tmp/changeRootPassword /artifactory/scripts && \
            mv /tmp/changeUserPassword /artifactory/scripts && \
            mv /tmp/backup /artifactory/scripts && \
            mv /tmp/restore /artifactory/scripts && \
            chmod 755 /artifactory/scripts/*

USER        $RUN_USER

EXPOSE      $RUN_PORT

ENTRYPOINT  ["/artifactory/scripts/startup"]
