# Copy https://hub.docker.com/r/sameersbn/squid/~/dockerfile/ 
# so we can tweak it.
FROM ubuntu:16.04

ENV SQUID_VERSION=3.5.12 \
    SQUID_CACHE_DIR=/var/spool/squid3 \
    SQUID_LOG_DIR=/var/log/squid3 \
    SQUID_USER=proxy

RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 05A314B7EF2B0AE21A50C9E91577D4D5BA3FF786 \
 && echo "deb http://ppa.launchpad.net/dnmvisser/squid-mitm/ubuntu xenial main" >> /etc/apt/sources.list \
 && apt-get update -y \
 && apt-get install -y software-properties-common python-software-properties \
 && add-apt-repository ppa:dnmvisser/squid-mitm \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y squid3 \
 && mv /etc/squid/squid.conf /etc/squid/squid.conf.dist \
 && rm -rf /var/lib/apt/lists/*

# Copy our config
COPY squid.conf /etc/squid/squid.conf
COPY entrypoint.sh /sbin/entrypoint.sh
RUN chmod 755 /sbin/entrypoint.sh

EXPOSE 3128/tcp
VOLUME ["${SQUID_CACHE_DIR}"]
ENTRYPOINT ["/sbin/entrypoint.sh"]

# Install some debugging tools
RUN apt-get update -y \
        && apt-get install -y net-tools netcat less tcpdump vim

# Initialize the TLS certificates cache directory
# http://wiki.squid-cache.org/ConfigExamples/Intercept/SslBumpExplicit
RUN /usr/lib/squid/ssl_crtd -c -s /var/lib/ssl_db
RUN chown -R proxy:proxy /var/lib/ssl_db

# install OpenJDK 8 JRE so we can use its keytool
RUN apt-get install -y openjdk-8-jre
