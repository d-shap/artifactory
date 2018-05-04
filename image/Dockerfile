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
RUN         sudo add-apt-repository "deb http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" && \
            apt-get update && \
            apt-get install --assume-yes \
                apache2 \
                oracle-java8-installer \
                software-properties-common \
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
