FROM        debian:jessie
MAINTAINER  Dmitry Shapovalov

ARG         hostuid=966
ARG         hostgid=966
ARG         guestport=8006
ARG         guestappname=artifactory

ENV         DEBIAN_FRONTEND noninteractive
ENV         APACHE_RUN_USER artifactory
ENV         APACHE_RUN_GROUP artifactory
ENV         GUEST_APP_NAME $guestappname
ENV         DB_NAME artdb
ENV         DB_USER_NAME artifactory
ENV         DOWNLOAD_VERSION 5.11.0
ENV         DOWNLOAD_NAME jfrog-artifactory-pro-$DOWNLOAD_VERSION
ENV         DOWNLOAD_URL https://bintray.com/jfrog/artifactory-pro/download_file?file_path=org%2Fartifactory%2Fpro%2Fjfrog-artifactory-pro%2F$DOWNLOAD_VERSION%2F$DOWNLOAD_NAME.zip

COPY        . /tmp

# Install packages
RUN         echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" >> /etc/apt/sources.list.d/java.list && \
            echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" >> /etc/apt/sources.list.d/java.list && \
            echo "oracle-java8-installer shared/accepted-oracle-license-v1-1 select true" | /usr/bin/debconf-set-selections && \
            apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys EEA14886 && \
            apt-get update && \
            apt-get install --assume-yes \
                apache2 \
                oracle-java8-installer \
                oracle-java8-set-default \
                vim \
                wget && \
            apt-get clean

# Create user and group
RUN         groupadd -g $hostgid artifactory && \
            useradd -u $hostuid -g $hostgid -M artifactory

# Create folders for artifactory
RUN         mkdir /artifactory && \
            mkdir /artifactory/db && \
            mkdir /artifactory/data && \
            mkdir /artifactory/scripts && \
            mkdir /artifactory/logs && \
            mkdir /artifactory/backups && \
            chown -R artifactory:artifactory /artifactory
