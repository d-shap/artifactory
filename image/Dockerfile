FROM        debian:buster
MAINTAINER  Dmitry Shapovalov

ARG         hostuid=966
ARG         hostgid=966

ENV         DEBIAN_FRONTEND=noninteractive \
            RUN_USER=artifactory \
            RUN_GROUP=artifactory \
            RUN_PORT=8006 \
            JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64 \
            ARTIFACTORY_HOME=/etc/artifactory \
            DB_NAME=artdb \
            DB_USER_NAME=artifactory \
            ARTIFACTORY_DOWNLOAD_URL="https://bintray.com/jfrog/artifactory/download_file?file_path=jfrog-artifactory-oss-6.23.7.zip" \
            ARTIFACTORY_DOWNLOAD_NAME="artifactory-oss-6.23.7" \
            ARTIFACTORY_FOLDER_NAME="artifactory-oss-6.23.7" \
            JDBC_DOWNLOAD_URL="https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-8.0.22.zip" \
            JDBC_DOWNLOAD_NAME="mysql-connector-java-8.0.22" \
            JDBC_FOLDER_NAME="mysql-connector-java-8.0.22" \
            JDBC_JAR_NAME="mysql-connector-java-8.0.22"

COPY        . /tmp

# Install common packages and configure environment
RUN         echo "deb http://ftp.debian.org/debian buster-backports main" >> /etc/apt/sources.list.d/backports.list && \
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
            mkdir /artifactory/accessd && \
            mkdir /artifactory/accesse && \
            mkdir /artifactory/security && \
            mkdir /artifactory/scripts && \
            mkdir /artifactory/logs && \
            mkdir /artifactory/backups && \
            chown -R $RUN_USER:$RUN_GROUP /artifactory

# Install Java
RUN         apt-get update && \
            apt-get install --assume-yes -t buster-backports \
                ca-certificates-java \
                openjdk-11-jdk-headless \
                openjdk-11-jre-headless && \
            apt-get clean

# Install MySQL
RUN         apt-get update && \
            apt-get install --assume-yes \
                default-mysql-client \
                default-mysql-server && \
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
            sed -i '\|innodb_file_per_table|a max_heap_table_size     = 512M' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i '\|innodb_file_per_table|a tmp_table_size          = 512M' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i '\|innodb_file_per_table|a innodb_buffer_pool_size = 1536M' /etc/mysql/mariadb.conf.d/50-server.cnf && \
            sed -i 's|syslog.*|# syslog|' /etc/mysql/mariadb.conf.d/50-mysqld_safe.cnf && \
            rm -rf /var/lib/mysql && \
            rm -rf /var/log/mysql && \
            ln -s /artifactory/logs /var/log/mysql && \
            chown -h $RUN_USER:$RUN_GROUP /var/log/mysql && \
            mkdir -p /var/run/mysqld && \
            chown -R $RUN_USER:$RUN_GROUP /var/run/mysqld

# Configure app
RUN         cd /tmp && \
            mkdir $ARTIFACTORY_HOME && \
            wget -O $ARTIFACTORY_DOWNLOAD_NAME.zip $ARTIFACTORY_DOWNLOAD_URL && \
            unzip -q $ARTIFACTORY_DOWNLOAD_NAME.zip -d /tmp && \
            rm $ARTIFACTORY_DOWNLOAD_NAME.zip && \
            mv $ARTIFACTORY_FOLDER_NAME/* $ARTIFACTORY_HOME && \
            rm -rf $ARTIFACTORY_FOLDER_NAME && \
            wget -O $JDBC_DOWNLOAD_NAME.zip $JDBC_DOWNLOAD_URL && \
            unzip -q $JDBC_DOWNLOAD_NAME.zip -d /tmp && \
            rm $JDBC_DOWNLOAD_NAME.zip && \
            mv $JDBC_FOLDER_NAME/$JDBC_JAR_NAME.jar $ARTIFACTORY_HOME/tomcat/lib && \
            rm -rf $JDBC_FOLDER_NAME && \
            rm -rf $ARTIFACTORY_HOME/data && \
            ln -s /artifactory/data $ARTIFACTORY_HOME/data && \
            rm -rf $ARTIFACTORY_HOME/access && \
            mkdir $ARTIFACTORY_HOME/access && \
            ln -s /artifactory/accessd $ARTIFACTORY_HOME/access/data && \
            ln -s /artifactory/accesse $ARTIFACTORY_HOME/access/etc && \
            ln -s /artifactory/logs $ARTIFACTORY_HOME/access/logs && \
            ln -s /artifactory/backups $ARTIFACTORY_HOME/access/backup && \
            ln -s /tmp $ARTIFACTORY_HOME/access/tmp && \
            rm -rf $ARTIFACTORY_HOME/etc/security && \
            ln -s /artifactory/security $ARTIFACTORY_HOME/etc/security && \
            rm -rf $ARTIFACTORY_HOME/logs && \
            ln -s /artifactory/logs $ARTIFACTORY_HOME/logs && \
            rm -rf $ARTIFACTORY_HOME/backup && \
            ln -s /artifactory/backups $ARTIFACTORY_HOME/backup && \
            sed -i 's|port="8081"|port="'"$RUN_PORT"'"|' $ARTIFACTORY_HOME/tomcat/conf/server.xml && \
            rm -rf $ARTIFACTORY_HOME/tomcat/logs && \
            ln -s /artifactory/logs $ARTIFACTORY_HOME/tomcat/logs && \
            rm -rf $ARTIFACTORY_HOME/tomcat/temp && \
            ln -s /tmp $ARTIFACTORY_HOME/tomcat/temp && \
            cp $ARTIFACTORY_HOME/misc/db/mysql.properties $ARTIFACTORY_HOME/etc/db.properties && \
            sed -i 's|driver.*|driver=com.mysql.cj.jdbc.Driver|' $ARTIFACTORY_HOME/etc/db.properties && \
            sed -i 's|url=.*|url=jdbc:mysql://localhost:3306/'"$DB_NAME"'?characterEncoding=UTF-8\&elideSetAutoCommitts=true\&serverTimezone=UTC|' $ARTIFACTORY_HOME/etc/db.properties && \
            sed -i 's|username.*|username='"$DB_USER_NAME"'|' $ARTIFACTORY_HOME/etc/db.properties && \
            chown -R -h $RUN_USER:$RUN_GROUP $ARTIFACTORY_HOME

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
