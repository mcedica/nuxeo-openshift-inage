# vim:set ft=dockerfile:
FROM registry.access.redhat.com/rhel7
MAINTAINER Nuxeo <packagers@nuxeo.com>

# install java
RUN yum install -y \
       java-1.8.0-openjdk 

# install wget
RUN yum -y install wget 
# install unzip
RUN yum -y install unzip 


# Create Nuxeo user
ENV NUXEO_USER nuxeo
RUN useradd -m -d /home/$NUXEO_USER -s /bin/bash $NUXEO_USER

#Add repositories need it for ffmpeg2theora and ffmpeg
RUN wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm \
    && rpm -Uvh epel-release-latest-7*.rpm \
    && rpm --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro \
    && rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-1.el7.nux.noarch.rpm

## must find another way to install it, need it for ffmpeg
RUN rpm -Uvh ftp://ftp.pbone.net/mirror/li.nux.ro/download/nux/dextop/retired/libfribidi-0.19.2-3.el7.nux.x86_64.rpm

# enable repo for libreoffice
RUN yum-config-manager --enable rhel-7-server-optional-rpms
# Add needed convert tools
RUN yum -y upgrade && yum -y install \
    perl \
    ImageMagick \
    ffmpeg \
    ffmpeg2theora \
    ufraw \
    poppler-utils \
    libreoffice-headless  \
    libwpd-tools \
    perl-Image-ExifTool \
    ghostscript \
 && rm -rf /var/lib/apt/lists/*

ENV NUXEO_VERSION 8.10
ENV NUXEO_MD5 29e67a19bba54099093b51d892926be1
ENV NUXEO_HOME /opt/nuxeo/server

# Add distribution
RUN curl -fsSL "http://cdn.nuxeo.com/nuxeo-${NUXEO_VERSION}/nuxeo-server-${NUXEO_VERSION}-tomcat.zip" -o /tmp/nuxeo-distribution-tomcat.zip \
    && echo "$NUXEO_MD5 /tmp/nuxeo-distribution-tomcat.zip" | md5sum -c - \
    && mkdir -p /tmp/nuxeo-distribution $(dirname $NUXEO_HOME) \
    && unzip -q -d /tmp/nuxeo-distribution /tmp/nuxeo-distribution-tomcat.zip \
    && DISTDIR=$(/bin/ls /tmp/nuxeo-distribution | head -n 1) \
    && mv /tmp/nuxeo-distribution/$DISTDIR $NUXEO_HOME \
    && sed -i -e "s/^org.nuxeo.distribution.package.*/org.nuxeo.distribution.package=docker/" $NUXEO_HOME/templates/common/config/distribution.properties \
    && rm -rf /tmp/nuxeo-distribution* \
    && chmod +x $NUXEO_HOME/bin/*ctl $NUXEO_HOME/bin/*.sh

RUN mkdir /docker-entrypoint-initnuxeo.d
ENV PATH $NUXEO_HOME/bin:$PATH

USER root

RUN mkdir /var/lib/nuxeo \
    && chmod a+rw /var/lib/nuxeo \
    && mkdir /var/log/nuxeo \
    && chmod a+rw /var/log/nuxeo \
    && mkdir /var/run/nuxeo \
    && chmod a+rw /var/run/nuxeo \
    && chmod a+rw $NUXEO_HOME \
    && chmod -R a+rw $NUXEO_HOME/* \
&& chmod a+x $NUXEO_HOME/bin/*ctl $NUXEO_HOME/bin/*.sh

USER nuxeo
WORKDIR $NUXEO_HOME
COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
EXPOSE 8080
EXPOSE 8787
CMD ["nuxeoctl","console"]


