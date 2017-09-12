# Pull base image
FROM networkstatic/iperf3

# Add our scripts for client and server
ADD iperf3* /usr/local/bin/

# Make them executable
RUN /bin/chmod +x /usr/local/bin/iperf3*

# Add a few utils.
RUN apt-get update -y \
    && apt-get install -y net-tools
