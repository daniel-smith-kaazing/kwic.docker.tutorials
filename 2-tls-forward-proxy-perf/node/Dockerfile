# Add base image.
FROM openjdk:8

# Download KWIC
ENV KWIC_VERSION 5.7.2
ENV KWIC_URL https://cdn.kaazing.com/releases/kwic/$KWIC_VERSION/kwic-$KWIC_VERSION-linux.tar.gz

RUN curl -L -o /kwic-$KWIC_VERSION.tar.gz $KWIC_URL
RUN cd /
RUN tar axvf /kwic-$KWIC_VERSION.tar.gz
RUN ls -latr /
RUN mv /kwic-$KWIC_VERSION /kaazing-kwic-5

WORKDIR /kaazing-kwic-5

# Add a few utils.
RUN apt-get update -y \
    && apt-get install -y net-tools netcat
