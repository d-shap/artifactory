FROM        debian:buster
MAINTAINER  Dmitry Shapovalov

ARG         hostuid=966
ARG         hostgid=966

ENV         DEBIAN_FRONTEND=noninteractive \
            RUN_USER="artifactory" \
            RUN_GROUP="artifactory" \
            RUN_ARTIFACTORY_PORT=8051 \
            RUN_TOMCAT_MAINTENANCE_PORT=8052 \
            RUN_ACCESS_HTTP_PORT=8053 \
            RUN_ACCESS_GRPC_PORT=8054 \
            RUN_FRONTEND_PORT=8055 \
            RUN_METADATA_PORT=8056 \
            RUN_ROUTER_INTERNAL_PORT=8057 \
            RUN_ROUTER_EXTERNAL_PORT=8058 \
            RUN_ROUTER_TRAEFIC_PORT=8059 \
            RUN_ROUTER_GRPC_PORT=8060 \
            RUN_EVENT_PORT=8061 \
            JFROG_HOME="/jfrog" \
            ARTIFACTORY_HOME="/jfrog/artifactory" \
            DB_NAME="artdb" \
            DB_USER_NAME="artifactory" \
            ARTIFACTORY_DOWNLOAD_URL="https://releases.jfrog.io/artifactory/bintray-artifactory/org/artifactory/oss/jfrog-artifactory-oss/7.77.7/jfrog-artifactory-oss-7.77.7-linux.tar.gz" \
            ARTIFACTORY_DOWNLOAD_NAME="artifactory-oss-7.77.7.tar.gz" \
            ARTIFACTORY_FOLDER_NAME="artifactory-oss-7.77.7" \
            JDBC_DOWNLOAD_URL="https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-8.0.24.tar.gz" \
            JDBC_DOWNLOAD_NAME="mysql-connector-java-8.0.24.tar.gz" \
            JDBC_FOLDER_NAME="mysql-connector-java-8.0.24" \
            JDBC_JAR_NAME="mysql-connector-java-8.0.24.jar"

COPY        . /tmp

# Install GPG and keys
RUN         apt-get update && \
            apt-get upgrade --assume-yes && \
            apt-get install --assume-yes \
                gnupg && \
            apt-get clean && \
            gpg --keyserver pgp.mit.edu --recv-keys 7638D0442B90D010 8B48AD6246925553 && \
            gpg --armor --export 7638D0442B90D010 | apt-key add - && \
            gpg --armor --export 8B48AD6246925553 | apt-key add -

# Install common packages and configure environment
RUN         echo "deb http://ftp.debian.org/debian buster-backports main" >> /etc/apt/sources.list.d/backports.list && \
            apt-get update && \
            apt-get upgrade --assume-yes && \
            apt-get install --assume-yes \
                apt-utils \
                build-essential \
                cron \
                curl \
                net-tools \
                procps \
                software-properties-common \
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
            mkdir /artifactory/scripts && \
            mkdir /artifactory/logs && \
            mkdir /artifactory/backups && \
            chown -R $RUN_USER:$RUN_GROUP /artifactory

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
            mkdir $JFROG_HOME && \
            wget -O $ARTIFACTORY_DOWNLOAD_NAME $ARTIFACTORY_DOWNLOAD_URL && \
            tar -C /tmp -xzf $ARTIFACTORY_DOWNLOAD_NAME && \
            rm $ARTIFACTORY_DOWNLOAD_NAME && \
            mv $ARTIFACTORY_FOLDER_NAME $ARTIFACTORY_HOME && \
            wget -O $JDBC_DOWNLOAD_NAME $JDBC_DOWNLOAD_URL && \
            tar -C /tmp -xzf $JDBC_DOWNLOAD_NAME && \
            rm $JDBC_DOWNLOAD_NAME && \
            mv $JDBC_FOLDER_NAME/$JDBC_JAR_NAME $ARTIFACTORY_HOME/var/bootstrap/artifactory/tomcat/lib && \
            rm -rf $JDBC_FOLDER_NAME && \
            rm -rf $ARTIFACTORY_HOME/var/data && \
            ln -s /artifactory/data $ARTIFACTORY_HOME/var/data && \
            rm -rf $ARTIFACTORY_HOME/var/etc/security && \
            ln -s /artifactory/security $ARTIFACTORY_HOME/var/etc/security && \
            rm -rf $ARTIFACTORY_HOME/var/log && \
            ln -s /artifactory/logs $ARTIFACTORY_HOME/var/log && \
            rm -rf $ARTIFACTORY_HOME/var/backup && \
            ln -s /artifactory/backups $ARTIFACTORY_HOME/var/backup && \
            rm -rf $ARTIFACTORY_HOME/app/artifactory/tomcat/logs && \
            ln -s /artifactory/logs $ARTIFACTORY_HOME/app/artifactory/tomcat/logs && \
            cp $ARTIFACTORY_HOME/var/etc/system.basic-template.yaml $ARTIFACTORY_HOME/var/etc/system.yaml && \
            sed -i 's|\(\s*\)#javaHome: .*|\1javaHome: "'"$ARTIFACTORY_HOME"'/app/third-party/java"|' $ARTIFACTORY_HOME/var/etc/system.yaml && \
            sed -i 's|\(\s*\)#joinKeyFile: .*|\1joinKeyFile: "'"$ARTIFACTORY_HOME"'/var/etc/security/join.key"|' $ARTIFACTORY_HOME/var/etc/system.yaml && \
            sed -i 's|\(\s*\)#masterKeyFile: .*|\1masterKeyFile: "'"$ARTIFACTORY_HOME"'/var/etc/security/master.key"|' $ARTIFACTORY_HOME/var/etc/system.yaml && \
            sed -i 's|\(\s*\)#haEnabled: .*|\1haEnabled: false|' $ARTIFACTORY_HOME/var/etc/system.yaml && \
            sed -i 's|\(\s*\)## Example for postgresql|\1## Example for mysql|' $ARTIFACTORY_HOME/var/etc/system.yaml && \
            sed -i 's|\(\s*\)#type: .*|\1type: mysql|' $ARTIFACTORY_HOME/var/etc/system.yaml && \
            sed -i 's|\(\s*\)#driver: .*|\1driver: "com.mysql.jdbc.Driver"|' $ARTIFACTORY_HOME/var/etc/system.yaml && \
            sed -i 's|\(\s*\)#url: .*|\1url: "jdbc:mysql://localhost:3306/'"$DB_NAME"'?characterEncoding=UTF-8\&elideSetAutoCommits=true\&useSSL=false\&serverTimezone=UTC"|' $ARTIFACTORY_HOME/var/etc/system.yaml && \
            sed -i 's|\(\s*\)#username: .*|\1username: "'"$DB_USER_NAME"'"|' $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "  user: $RUN_USER" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "  group: $RUN_GROUP" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "  logging:" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "    consoleLog:" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "      enabled: false" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "artifactory:" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "  port: $RUN_ARTIFACTORY_PORT" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "  tomcat:" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "    maintenanceConnector: $RUN_TOMCAT_MAINTENANCE_PORT" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "    httpsConnector:" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "      enabled: false" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "access:" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "  http:" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "    port: $RUN_ACCESS_HTTP_PORT" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "  grpc:" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "    port: $RUN_ACCESS_GRPC_PORT" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "frontend:" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "  port: $RUN_FRONTEND_PORT" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "metadata:" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "  port: $RUN_METADATA_PORT" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "router:" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "  entrypoints:" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "    internalPort: $RUN_ROUTER_INTERNAL_PORT" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "    externalPort: $RUN_ROUTER_EXTERNAL_PORT" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "    traefikApiPort: $RUN_ROUTER_TRAEFIC_PORT" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "    grpcPort: $RUN_ROUTER_GRPC_PORT" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "event:" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            echo "  port: $RUN_EVENT_PORT" >> $ARTIFACTORY_HOME/var/etc/system.yaml && \
            chown -R -h $RUN_USER:$RUN_GROUP $JFROG_HOME

# Configure helper scripts
RUN         mv /tmp/containerConfig /artifactory/scripts && \
            mv /tmp/containerDummy /artifactory/scripts && \
            mv /tmp/containerStartup /artifactory/scripts && \
            mv /tmp/containerShutdown /artifactory/scripts && \
            mv /tmp/webStartup /artifactory/scripts && \
            mv /tmp/webShutdown /artifactory/scripts && \
            mv /tmp/utilInitialize /artifactory/scripts && \
            mv /tmp/utilChangeRootPassword /artifactory/scripts && \
            mv /tmp/utilChangeUserPassword /artifactory/scripts && \
            mv /tmp/utilBackup /artifactory/scripts && \
            mv /tmp/utilRestore /artifactory/scripts && \
            chmod 755 /artifactory/scripts/*

USER        $RUN_USER

EXPOSE      $RUN_ARTIFACTORY_PORT \
            $RUN_TOMCAT_MAINTENANCE_PORT \
            $RUN_ACCESS_HTTP_PORT \
            $RUN_ACCESS_GRPC_PORT \
            $RUN_FRONTEND_PORT \
            $RUN_METADATA_PORT \
            $RUN_ROUTER_INTERNAL_PORT \
            $RUN_ROUTER_EXTERNAL_PORT \
            $RUN_ROUTER_TRAEFIC_PORT \
            $RUN_ROUTER_GRPC_PORT \
            $RUN_EVENT_PORT

ENTRYPOINT  ["/artifactory/scripts/containerStartup"]
