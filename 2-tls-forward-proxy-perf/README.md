# KWIC TLS with Squid SSL passthrough forward proxy between gateways - PERF TEST

## Configuration

The configuration for the cloud KWIC instance is in [config/cloud-config.xml](config/cloud-config.xml).

The configuration for the on-prem KWIC instance is in [config/onprem-config.xml](config/onprem-config.xml).

These have been changed slightly from the original 2-tls tutorial:

- There is a squid proxy acting as a transparent forward proxy between onprem and example.com
- These have been replaced with iperf3:
-- The "service-A" (reverse) service
See docker-compose service iperf3-sink-reverse
-- The "service-B" (forward) service
See docker-compose service iperf3-sink-forward
-- The "service-A" client
See docker-compose service iperf3-source-reverse
-- The "service-B" client
See docker-compose service iperf3-source-forward

Note that if you want to do the perf test without a proxy, simply remove these from the onprem service GATEWAY_OPTS:

```
-Dhttp.proxyHost=squid -Dhttp.proxyPort=3128
```

# Running

docker-compose up

# Configuring perf sources

Here are the comments in services iperf3-source-reverse and iperf3-source-forward in the docker-compose.yml:

```
    # Params are:
    # $1 - SLEEP time for loop - sec.
    # $2 - iperf3 sink host
    # $3 - port to connect
    # $4 - parallel connections
    # $5 - report interval - sec.
    # $6 - duration of client - sec.
    # $7 - bandwidth - default is no throttle - values can have K|G|M as in 201M = 201mpbs per connection
```

By default I have provided no last parameter which in effect removes the throttle from the sources, i.e. they pump data as fast as they can.

You could tweak the test to test other things. Right now I am only tearing down the connections every 3600 seconds. If one wanted to test tear down and rebuilding of the connections in either direction - including the reverse WSS-over-SOCKS stack - one could shorten that 3600 second time to, say, 300 seconds.

# What the test does

It cranks data as fast as it can in both directions. In each direction it spawns 10 parallel threads for each sink (as currently configured). Here are the command lines provided in the docker-compose.yml for each direction:

```
iperf3-source-reverse:
    entrypoint: ["/usr/local/bin/iperf3client"]
    command: ["10", "example.com", "5551", "10", "60", "3600"]
```

So, sleep 10 seconds each time through the loop. Connect to example.com on port 5551 with 10 parallel connections. Report every 60 seconds (see output below) and run for 3600 seconds. Then repeat until shut down.

```
iperf3-source-forward:
    entrypoint: ["/usr/local/bin/iperf3client"]
    command: ["10", "onprem", "6661", "10", "60", "3600"]
```

Same as the above but for onprem:6661.
 
# Output

Every so often you will see the 60-second report output similar to the following in the console.
Note that even on the Ubuntu lab VM I am running on for this test I see throughput of ~75Mbps in each direction or a total of 150Mbps:

```
iperf3-sink-forward      | [ 23] 180.00-240.00 sec  43.7 MBytes  6.10 Mbits/sec        
iperf3-sink-forward      | [SUM] 180.00-240.00 sec   537 MBytes  75.0 Mbits/sec        
iperf3-sink-forward      | - - - - - - - - - - - - - - - - - - - - - - - - -
iperf3-sink-forward      | [  5] 240.00-300.00 sec  55.2 MBytes  7.71 Mbits/sec        
iperf3-sink-forward      | [  7] 240.00-300.00 sec  44.1 MBytes  6.16 Mbits/sec        
iperf3-sink-forward      | [  9] 240.00-300.00 sec  64.0 MBytes  8.95 Mbits/sec        
iperf3-sink-forward      | [ 11] 240.00-300.00 sec  55.2 MBytes  7.71 Mbits/sec        
iperf3-sink-forward      | [ 13] 240.00-300.00 sec  64.0 MBytes  8.95 Mbits/sec        
iperf3-sink-forward      | [ 15] 240.00-300.00 sec  53.7 MBytes  7.51 Mbits/sec        
iperf3-sink-forward      | [ 17] 240.00-300.00 sec  55.2 MBytes  7.71 Mbits/sec        
iperf3-sink-forward      | [ 19] 240.00-300.00 sec  53.7 MBytes  7.51 Mbits/sec        
iperf3-sink-forward      | [ 21] 240.00-300.00 sec  44.1 MBytes  6.16 Mbits/sec        
iperf3-sink-forward      | [ 23] 240.00-300.00 sec  44.1 MBytes  6.16 Mbits/sec        
iperf3-sink-forward      | [SUM] 240.00-300.00 sec   533 MBytes  74.6 Mbits/sec        
iperf3-sink-forward      | - - - - - - - - - - - - - - - - - - - - - - - - -
iperf3-sink-forward      | [  5] 300.00-360.00 sec  86.5 MBytes  12.1 Mbits/sec        
iperf3-sink-forward      | [  7] 300.00-360.00 sec  42.1 MBytes  5.89 Mbits/sec        
iperf3-sink-forward      | [  9] 300.00-360.00 sec  65.0 MBytes  9.08 Mbits/sec        
iperf3-sink-forward      | [ 11] 300.00-360.00 sec  55.4 MBytes  7.75 Mbits/sec        
iperf3-sink-forward      | [ 13] 300.00-360.00 sec  60.3 MBytes  8.43 Mbits/sec        
iperf3-sink-forward      | [ 15] 300.00-360.00 sec  51.3 MBytes  7.17 Mbits/sec        
iperf3-sink-forward      | [ 17] 300.00-360.00 sec  55.5 MBytes  7.76 Mbits/sec        
iperf3-sink-forward      | [ 19] 300.00-360.00 sec  51.3 MBytes  7.17 Mbits/sec        
iperf3-sink-forward      | [ 21] 300.00-360.00 sec  42.1 MBytes  5.89 Mbits/sec        
iperf3-sink-forward      | [ 23] 300.00-360.00 sec  42.1 MBytes  5.89 Mbits/sec        
iperf3-sink-forward      | [SUM] 300.00-360.00 sec   552 MBytes  77.1 Mbits/sec        
iperf3-sink-forward      | - - - - - - - - - - - - - - - - - - - - - - - - -
iperf3-sink-forward      | [  5] 360.00-420.00 sec   113 MBytes  15.8 Mbits/sec        
iperf3-sink-forward      | [  7] 360.00-420.00 sec  40.8 MBytes  5.71 Mbits/sec        
iperf3-sink-forward      | [  9] 360.00-420.00 sec  61.2 MBytes  8.56 Mbits/sec        
iperf3-sink-forward      | [ 11] 360.00-420.00 sec  58.4 MBytes  8.16 Mbits/sec        
iperf3-sink-forward      | [ 13] 360.00-420.00 sec  58.3 MBytes  8.15 Mbits/sec        
iperf3-sink-forward      | [ 15] 360.00-420.00 sec  49.2 MBytes  6.88 Mbits/sec        
iperf3-sink-forward      | [ 17] 360.00-420.00 sec  58.4 MBytes  8.16 Mbits/sec        
iperf3-sink-forward      | [ 19] 360.00-420.00 sec  49.2 MBytes  6.88 Mbits/sec        
iperf3-sink-forward      | [ 21] 360.00-420.00 sec  40.8 MBytes  5.70 Mbits/sec        
iperf3-sink-forward      | [ 23] 360.00-420.00 sec  40.8 MBytes  5.70 Mbits/sec        
iperf3-sink-forward      | [SUM] 360.00-420.00 sec   570 MBytes  79.7 Mbits/sec        
iperf3-sink-forward      | - - - - - - - - - - - - - - - - - - - - - - - - -
iperf3-sink-forward      | [  5] 420.00-480.01 sec   126 MBytes  17.6 Mbits/sec        
iperf3-sink-forward      | [  7] 420.00-480.01 sec  41.4 MBytes  5.78 Mbits/sec        
iperf3-sink-forward      | [  9] 420.00-480.01 sec  65.8 MBytes  9.20 Mbits/sec        
iperf3-sink-forward      | [ 11] 420.00-480.01 sec  53.5 MBytes  7.48 Mbits/sec        
iperf3-sink-forward      | [ 13] 420.00-480.01 sec  53.5 MBytes  7.49 Mbits/sec        
iperf3-sink-forward      | [ 15] 420.00-480.01 sec  51.1 MBytes  7.15 Mbits/sec        
iperf3-sink-forward      | [ 17] 420.00-480.01 sec  53.5 MBytes  7.48 Mbits/sec        
iperf3-sink-forward      | [ 19] 420.00-480.01 sec  51.1 MBytes  7.15 Mbits/sec        
iperf3-sink-forward      | [ 21] 420.00-480.01 sec  41.4 MBytes  5.78 Mbits/sec        
iperf3-sink-forward      | [ 23] 420.00-480.01 sec  41.4 MBytes  5.78 Mbits/sec        
iperf3-sink-forward      | [SUM] 420.00-480.01 sec   579 MBytes  80.9 Mbits/sec        
iperf3-sink-forward      | - - - - - - - - - - - - - - - - - - - - - - - - -
iperf3-sink-forward      | [  5] 480.01-540.00 sec  97.9 MBytes  13.7 Mbits/sec        
iperf3-sink-forward      | [  7] 480.01-540.00 sec  43.8 MBytes  6.13 Mbits/sec        
iperf3-sink-forward      | [  9] 480.01-540.00 sec  70.1 MBytes  9.80 Mbits/sec        
iperf3-sink-forward      | [ 11] 480.01-540.00 sec  50.2 MBytes  7.02 Mbits/sec        
iperf3-sink-forward      | [ 13] 480.01-540.00 sec  50.3 MBytes  7.03 Mbits/sec        
iperf3-sink-forward      | [ 15] 480.01-540.00 sec  53.5 MBytes  7.47 Mbits/sec        
iperf3-sink-forward      | [ 17] 480.01-540.00 sec  50.2 MBytes  7.02 Mbits/sec        
iperf3-sink-forward      | [ 19] 480.01-540.00 sec  53.5 MBytes  7.47 Mbits/sec        
iperf3-source-forward    | [ 18] 180.00-240.00 sec  54.6 MBytes  7.64 Mbits/sec    0   65.0 KBytes
iperf3-source-forward    | [ 20] 180.00-240.00 sec  44.1 MBytes  6.16 Mbits/sec    0    130 KBytes
iperf3-source-forward    | [ 22] 180.00-240.00 sec  54.4 MBytes  7.61 Mbits/sec    1    127 KBytes
iperf3-source-forward    | [SUM] 180.00-240.00 sec   539 MBytes  75.4 Mbits/sec    3   
iperf3-source-forward    | - - - - - - - - - - - - - - - - - - - - - - - - -
iperf3-source-forward    | [  4] 240.00-300.00 sec  55.9 MBytes  7.82 Mbits/sec    0   84.8 KBytes
iperf3-source-forward    | [  6] 240.00-300.00 sec  44.1 MBytes  6.16 Mbits/sec    0    134 KBytes
iperf3-source-forward    | [  8] 240.00-300.00 sec  55.1 MBytes  7.70 Mbits/sec    0   97.6 KBytes
iperf3-source-forward    | [ 10] 240.00-300.00 sec  63.3 MBytes  8.84 Mbits/sec    0    102 KBytes
iperf3-source-forward    | [ 12] 240.00-300.00 sec  64.4 MBytes  9.01 Mbits/sec    1   82.0 KBytes
iperf3-source-forward    | [ 14] 240.00-300.00 sec  44.6 MBytes  6.23 Mbits/sec    0    177 KBytes
iperf3-source-forward    | [ 16] 240.00-300.00 sec  53.9 MBytes  7.53 Mbits/sec    0   65.0 KBytes
iperf3-source-forward    | [ 18] 240.00-300.00 sec  54.9 MBytes  7.68 Mbits/sec    0   73.5 KBytes
iperf3-source-forward    | [ 20] 240.00-300.00 sec  43.9 MBytes  6.13 Mbits/sec    1    134 KBytes
iperf3-source-forward    | [ 22] 240.00-300.00 sec  53.7 MBytes  7.51 Mbits/sec    0    127 KBytes
iperf3-source-forward    | [SUM] 240.00-300.00 sec   534 MBytes  74.6 Mbits/sec    2   
iperf3-source-forward    | - - - - - - - - - - - - - - - - - - - - - - - - -
iperf3-source-forward    | [  4] 300.00-360.00 sec  78.6 MBytes  11.0 Mbits/sec    0    140 KBytes
iperf3-source-forward    | [  6] 300.00-360.00 sec  41.9 MBytes  5.86 Mbits/sec    1    136 KBytes
iperf3-source-forward    | [  8] 300.00-360.00 sec  53.2 MBytes  7.44 Mbits/sec    0    103 KBytes
iperf3-source-forward    | [ 10] 300.00-360.00 sec  58.5 MBytes  8.18 Mbits/sec    0   99.0 KBytes
iperf3-source-forward    | [ 12] 300.00-360.00 sec  64.0 MBytes  8.95 Mbits/sec    0   87.7 KBytes
iperf3-source-forward    | [ 14] 300.00-360.00 sec  41.9 MBytes  5.86 Mbits/sec    0    133 KBytes
iperf3-source-forward    | [ 16] 300.00-360.00 sec  50.6 MBytes  7.07 Mbits/sec    0   86.3 KBytes
iperf3-source-forward    | [ 18] 300.00-360.00 sec  53.4 MBytes  7.47 Mbits/sec    0   67.9 KBytes
iperf3-source-forward    | [ 20] 300.00-360.00 sec  41.8 MBytes  5.84 Mbits/sec    0    188 KBytes
iperf3-source-forward    | [ 22] 300.00-360.00 sec  51.0 MBytes  7.13 Mbits/sec    0    136 KBytes
iperf3-source-forward    | [SUM] 300.00-360.00 sec   535 MBytes  74.8 Mbits/sec    1   
iperf3-source-forward    | - - - - - - - - - - - - - - - - - - - - - - - - -
iperf3-source-forward    | [  4] 360.00-420.00 sec   109 MBytes  15.3 Mbits/sec    0    141 KBytes
iperf3-source-forward    | [  6] 360.00-420.00 sec  41.1 MBytes  5.75 Mbits/sec    2    127 KBytes
iperf3-source-forward    | [  8] 360.00-420.00 sec  58.4 MBytes  8.16 Mbits/sec    0   99.0 KBytes
iperf3-source-forward    | [ 10] 360.00-420.00 sec  58.6 MBytes  8.19 Mbits/sec    0   99.0 KBytes
iperf3-source-forward    | [ 12] 360.00-420.00 sec  61.9 MBytes  8.65 Mbits/sec    0    102 KBytes
iperf3-source-forward    | [ 14] 360.00-420.00 sec  40.9 MBytes  5.72 Mbits/sec    0    141 KBytes
iperf3-source-forward    | [ 16] 360.00-420.00 sec  49.7 MBytes  6.94 Mbits/sec    0   76.4 KBytes
iperf3-source-forward    | [ 18] 360.00-420.00 sec  58.1 MBytes  8.12 Mbits/sec    0   67.9 KBytes
iperf3-source-forward    | [ 20] 360.00-420.00 sec  41.3 MBytes  5.78 Mbits/sec    0    188 KBytes
iperf3-source-forward    | [ 22] 360.00-420.00 sec  49.3 MBytes  6.90 Mbits/sec    0    130 KBytes
iperf3-source-forward    | [SUM] 360.00-420.00 sec   569 MBytes  79.5 Mbits/sec    2   
iperf3-source-forward    | - - - - - - - - - - - - - - - - - - - - - - - - -
iperf3-source-forward    | [  4] 420.00-480.00 sec   126 MBytes  17.7 Mbits/sec    0    134 KBytes
iperf3-source-forward    | [  6] 420.00-480.00 sec  41.2 MBytes  5.76 Mbits/sec    0    127 KBytes
iperf3-source-forward    | [  8] 420.00-480.00 sec  54.1 MBytes  7.56 Mbits/sec    0   97.6 KBytes
iperf3-source-forward    | [ 10] 420.00-480.00 sec  53.2 MBytes  7.44 Mbits/sec    0   99.0 KBytes
iperf3-source-forward    | [ 12] 420.00-480.00 sec  66.4 MBytes  9.29 Mbits/sec    0   82.0 KBytes
iperf3-source-forward    | [ 14] 420.00-480.00 sec  41.3 MBytes  5.78 Mbits/sec    0    153 KBytes
iperf3-source-forward    | [ 16] 420.00-480.00 sec  51.3 MBytes  7.17 Mbits/sec    0   89.1 KBytes
iperf3-source-forward    | [ 18] 420.00-480.00 sec  53.8 MBytes  7.52 Mbits/sec    0   67.9 KBytes
iperf3-source-forward    | [ 20] 420.00-480.00 sec  41.0 MBytes  5.73 Mbits/sec    0    171 KBytes
iperf3-source-forward    | [ 22] 420.00-480.00 sec  51.1 MBytes  7.15 Mbits/sec    0    134 KBytes
iperf3-source-forward    | [SUM] 420.00-480.00 sec   580 MBytes  81.0 Mbits/sec    0   
iperf3-source-forward    | - - - - - - - - - - - - - - - - - - - - - - - - -
iperf3-source-forward    | [  4] 480.00-540.00 sec   106 MBytes  14.8 Mbits/sec    0    141 KBytes
iperf3-sink-reverse      | [ 23] 180.00-240.00 sec  56.0 MBytes  7.83 Mbits/sec        
iperf3-sink-reverse      | [SUM] 180.00-240.00 sec   555 MBytes  77.6 Mbits/sec        
iperf3-sink-reverse      | - - - - - - - - - - - - - - - - - - - - - - - - -
iperf3-sink-reverse      | [  5] 240.00-300.00 sec  65.7 MBytes  9.18 Mbits/sec        
iperf3-sink-reverse      | [  7] 240.00-300.00 sec  56.5 MBytes  7.89 Mbits/sec        
iperf3-sink-reverse      | [  9] 240.00-300.00 sec  45.6 MBytes  6.37 Mbits/sec        
iperf3-sink-reverse      | [ 11] 240.00-300.00 sec  66.3 MBytes  9.27 Mbits/sec        
iperf3-sink-reverse      | [ 13] 240.00-300.00 sec  55.4 MBytes  7.75 Mbits/sec        
iperf3-sink-reverse      | [ 15] 240.00-300.00 sec  56.5 MBytes  7.89 Mbits/sec        
iperf3-sink-reverse      | [ 17] 240.00-300.00 sec  55.5 MBytes  7.76 Mbits/sec        
iperf3-sink-reverse      | [ 19] 240.00-300.00 sec  45.6 MBytes  6.37 Mbits/sec        
iperf3-sink-reverse      | [ 21] 240.00-300.00 sec  45.4 MBytes  6.35 Mbits/sec        
iperf3-sink-reverse      | [ 23] 240.00-300.00 sec  55.4 MBytes  7.75 Mbits/sec        
iperf3-sink-reverse      | [SUM] 240.00-300.00 sec   548 MBytes  76.6 Mbits/sec        
iperf3-sink-reverse      | - - - - - - - - - - - - - - - - - - - - - - - - -
iperf3-sink-reverse      | [  5] 300.00-360.00 sec  67.0 MBytes  9.37 Mbits/sec        
iperf3-sink-reverse      | [  7] 300.00-360.00 sec  53.1 MBytes  7.43 Mbits/sec        
iperf3-sink-reverse      | [  9] 300.00-360.00 sec  43.2 MBytes  6.03 Mbits/sec        
iperf3-sink-reverse      | [ 11] 300.00-360.00 sec  67.1 MBytes  9.39 Mbits/sec        
iperf3-sink-reverse      | [ 13] 300.00-360.00 sec  52.6 MBytes  7.36 Mbits/sec        
iperf3-sink-reverse      | [ 15] 300.00-360.00 sec  53.1 MBytes  7.43 Mbits/sec        
iperf3-sink-reverse      | [ 17] 300.00-360.00 sec  52.6 MBytes  7.36 Mbits/sec        
iperf3-sink-reverse      | [ 19] 300.00-360.00 sec  43.1 MBytes  6.02 Mbits/sec        
iperf3-sink-reverse      | [ 21] 300.00-360.00 sec  43.0 MBytes  6.02 Mbits/sec        
iperf3-sink-reverse      | [ 23] 300.00-360.00 sec  52.7 MBytes  7.36 Mbits/sec        
iperf3-sink-reverse      | [SUM] 300.00-360.00 sec   528 MBytes  73.8 Mbits/sec        
iperf3-sink-reverse      | - - - - - - - - - - - - - - - - - - - - - - - - -
iperf3-sink-reverse      | [  5] 360.00-420.00 sec  63.6 MBytes  8.90 Mbits/sec        
iperf3-sink-reverse      | [  7] 360.00-420.00 sec  56.6 MBytes  7.91 Mbits/sec        
iperf3-sink-reverse      | [  9] 360.00-420.00 sec  42.5 MBytes  5.94 Mbits/sec        
iperf3-sink-reverse      | [ 11] 360.00-420.00 sec  63.3 MBytes  8.84 Mbits/sec        
iperf3-sink-reverse      | [ 13] 360.00-420.00 sec  51.2 MBytes  7.16 Mbits/sec        
iperf3-sink-reverse      | [ 15] 360.00-420.00 sec  56.6 MBytes  7.91 Mbits/sec        
iperf3-sink-reverse      | [ 17] 360.00-420.00 sec  51.2 MBytes  7.16 Mbits/sec        
iperf3-sink-reverse      | [ 19] 360.00-420.00 sec  42.6 MBytes  5.96 Mbits/sec        
iperf3-sink-reverse      | [ 21] 360.00-420.00 sec  42.8 MBytes  5.98 Mbits/sec        
iperf3-sink-reverse      | [ 23] 360.00-420.00 sec  51.2 MBytes  7.16 Mbits/sec        
iperf3-sink-reverse      | [SUM] 360.00-420.00 sec   522 MBytes  72.9 Mbits/sec        
iperf3-sink-reverse      | - - - - - - - - - - - - - - - - - - - - - - - - -
iperf3-sink-reverse      | [  5] 420.00-480.00 sec  67.5 MBytes  9.43 Mbits/sec        
iperf3-sink-reverse      | [  7] 420.00-480.00 sec  51.8 MBytes  7.25 Mbits/sec        
iperf3-sink-reverse      | [  9] 420.00-480.00 sec  42.8 MBytes  5.98 Mbits/sec        
iperf3-sink-reverse      | [ 11] 420.00-480.00 sec  67.9 MBytes  9.50 Mbits/sec        
iperf3-sink-reverse      | [ 13] 420.00-480.00 sec  52.8 MBytes  7.38 Mbits/sec        
iperf3-sink-reverse      | [ 15] 420.00-480.00 sec  51.8 MBytes  7.25 Mbits/sec        
iperf3-sink-reverse      | [ 17] 420.00-480.00 sec  52.9 MBytes  7.39 Mbits/sec        
iperf3-sink-reverse      | [ 19] 420.00-480.00 sec  42.7 MBytes  5.98 Mbits/sec        
iperf3-sink-reverse      | [ 21] 420.00-480.00 sec  42.8 MBytes  5.98 Mbits/sec        
iperf3-sink-reverse      | [ 23] 420.00-480.00 sec  52.8 MBytes  7.38 Mbits/sec        
iperf3-sink-reverse      | [SUM] 420.00-480.00 sec   526 MBytes  73.5 Mbits/sec        
iperf3-sink-reverse      | - - - - - - - - - - - - - - - - - - - - - - - - -
iperf3-sink-reverse      | [  5] 480.00-540.01 sec  68.1 MBytes  9.52 Mbits/sec        
iperf3-sink-reverse      | [  7] 480.00-540.01 sec  48.4 MBytes  6.76 Mbits/sec        
iperf3-sink-reverse      | [  9] 480.00-540.01 sec  45.3 MBytes  6.34 Mbits/sec        
iperf3-sink-reverse      | [ 11] 480.00-540.01 sec  72.5 MBytes  10.1 Mbits/sec        
iperf3-sink-reverse      | [ 13] 480.00-540.01 sec  55.5 MBytes  7.75 Mbits/sec        
iperf3-sink-reverse      | [ 15] 480.00-540.01 sec  48.4 MBytes  6.76 Mbits/sec        
iperf3-sink-reverse      | [ 17] 480.00-540.01 sec  55.2 MBytes  7.72 Mbits/sec        
iperf3-sink-reverse      | [ 19] 480.00-540.01 sec  45.4 MBytes  6.35 Mbits/sec        
iperf3-source-reverse    | [ 18] 180.02-240.00 sec  56.6 MBytes  7.92 Mbits/sec    0    181 KBytes
iperf3-source-reverse    | [ 20] 180.02-240.00 sec  45.2 MBytes  6.33 Mbits/sec    1   84.8 KBytes
iperf3-source-reverse    | [ 22] 180.02-240.00 sec  56.1 MBytes  7.85 Mbits/sec    0    103 KBytes
iperf3-source-reverse    | [SUM] 180.02-240.00 sec   555 MBytes  77.6 Mbits/sec    1   
iperf3-source-reverse    | - - - - - - - - - - - - - - - - - - - - - - - - -
iperf3-source-reverse    | [  4] 240.00-300.00 sec  45.4 MBytes  6.34 Mbits/sec    0   90.5 KBytes
iperf3-source-reverse    | [  6] 240.00-300.00 sec  55.3 MBytes  7.73 Mbits/sec    0   84.8 KBytes
iperf3-source-reverse    | [  8] 240.00-300.00 sec  65.9 MBytes  9.22 Mbits/sec    0    208 KBytes
iperf3-source-reverse    | [ 10] 240.00-300.00 sec  56.8 MBytes  7.94 Mbits/sec    0    191 KBytes
iperf3-source-reverse    | [ 12] 240.00-300.00 sec  45.6 MBytes  6.38 Mbits/sec    0   97.6 KBytes
iperf3-source-reverse    | [ 14] 240.00-300.00 sec  55.3 MBytes  7.73 Mbits/sec    0    120 KBytes
iperf3-source-reverse    | [ 16] 240.00-300.00 sec  66.4 MBytes  9.28 Mbits/sec    0   84.8 KBytes
iperf3-source-reverse    | [ 18] 240.00-300.00 sec  56.9 MBytes  7.95 Mbits/sec    0    178 KBytes
iperf3-source-reverse    | [ 20] 240.00-300.00 sec  45.7 MBytes  6.39 Mbits/sec    0   84.8 KBytes
iperf3-source-reverse    | [ 22] 240.00-300.00 sec  55.1 MBytes  7.70 Mbits/sec    0    103 KBytes
iperf3-source-reverse    | [SUM] 240.00-300.00 sec   548 MBytes  76.7 Mbits/sec    0   
iperf3-source-reverse    | - - - - - - - - - - - - - - - - - - - - - - - - -
iperf3-source-reverse    | [  4] 300.00-360.00 sec  43.4 MBytes  6.06 Mbits/sec    0   87.7 KBytes
iperf3-source-reverse    | [  6] 300.00-360.00 sec  53.1 MBytes  7.43 Mbits/sec    1   84.8 KBytes
iperf3-source-reverse    | [  8] 300.00-360.00 sec  67.6 MBytes  9.45 Mbits/sec    0    133 KBytes
iperf3-source-reverse    | [ 10] 300.00-360.00 sec  55.7 MBytes  7.78 Mbits/sec    0    157 KBytes
iperf3-source-reverse    | [ 12] 300.00-360.00 sec  43.3 MBytes  6.06 Mbits/sec    0    107 KBytes
iperf3-source-reverse    | [ 14] 300.00-360.00 sec  53.1 MBytes  7.42 Mbits/sec    1    117 KBytes
iperf3-source-reverse    | [ 16] 300.00-360.00 sec  66.9 MBytes  9.36 Mbits/sec    0   70.7 KBytes
iperf3-source-reverse    | [ 18] 300.00-360.00 sec  55.9 MBytes  7.82 Mbits/sec    0    144 KBytes
iperf3-source-reverse    | [ 20] 300.00-360.00 sec  43.3 MBytes  6.05 Mbits/sec    0   84.8 KBytes
iperf3-source-reverse    | [ 22] 300.00-360.00 sec  53.4 MBytes  7.47 Mbits/sec    0    103 KBytes
iperf3-source-reverse    | [SUM] 300.00-360.00 sec   536 MBytes  74.9 Mbits/sec    2   
iperf3-source-reverse    | - - - - - - - - - - - - - - - - - - - - - - - - -
iperf3-source-reverse    | [  4] 360.00-420.00 sec  42.4 MBytes  5.93 Mbits/sec    0   89.1 KBytes
iperf3-source-reverse    | [  6] 360.00-420.00 sec  50.9 MBytes  7.12 Mbits/sec    0   84.8 KBytes
iperf3-source-reverse    | [  8] 360.00-420.00 sec  63.1 MBytes  8.83 Mbits/sec    0    167 KBytes
iperf3-source-reverse    | [ 10] 360.00-420.00 sec  56.7 MBytes  7.93 Mbits/sec    0    158 KBytes
iperf3-source-reverse    | [ 12] 360.00-420.00 sec  42.2 MBytes  5.90 Mbits/sec    0    112 KBytes
iperf3-source-reverse    | [ 14] 360.00-420.00 sec  50.8 MBytes  7.10 Mbits/sec    0    115 KBytes
iperf3-source-reverse    | [ 16] 360.00-420.00 sec  63.4 MBytes  8.86 Mbits/sec    0   73.5 KBytes
iperf3-source-reverse    | [ 18] 360.00-420.00 sec  56.1 MBytes  7.85 Mbits/sec    0    140 KBytes
iperf3-source-reverse    | [ 20] 360.00-420.00 sec  42.3 MBytes  5.91 Mbits/sec    0   84.8 KBytes
iperf3-source-reverse    | [ 22] 360.00-420.00 sec  50.7 MBytes  7.09 Mbits/sec    0    105 KBytes
iperf3-source-reverse    | [SUM] 360.00-420.00 sec   519 MBytes  72.5 Mbits/sec    0   
iperf3-source-reverse    | - - - - - - - - - - - - - - - - - - - - - - - - -
iperf3-source-reverse    | [  4] 420.00-480.00 sec  42.6 MBytes  5.96 Mbits/sec    0   87.7 KBytes
iperf3-source-reverse    | [  6] 420.00-480.00 sec  52.9 MBytes  7.39 Mbits/sec    1   84.8 KBytes
iperf3-source-reverse    | [  8] 420.00-480.00 sec  67.7 MBytes  9.47 Mbits/sec    0    139 KBytes
iperf3-source-reverse    | [ 10] 420.00-480.00 sec  51.6 MBytes  7.22 Mbits/sec    0    168 KBytes
iperf3-source-reverse    | [ 12] 420.00-480.00 sec  43.0 MBytes  6.01 Mbits/sec    0   87.7 KBytes
iperf3-source-reverse    | [ 14] 420.00-480.00 sec  53.1 MBytes  7.43 Mbits/sec    0    123 KBytes
iperf3-source-reverse    | [ 16] 420.00-480.00 sec  67.9 MBytes  9.50 Mbits/sec    0   73.5 KBytes
iperf3-source-reverse    | [ 18] 420.00-480.00 sec  51.4 MBytes  7.18 Mbits/sec    0    140 KBytes
iperf3-source-reverse    | [ 20] 420.00-480.00 sec  42.8 MBytes  5.98 Mbits/sec    1   84.8 KBytes
iperf3-source-reverse    | [ 22] 420.00-480.00 sec  52.9 MBytes  7.39 Mbits/sec    0    110 KBytes
iperf3-source-reverse    | [SUM] 420.00-480.00 sec   526 MBytes  73.5 Mbits/sec    2   
iperf3-source-reverse    | - - - - - - - - - - - - - - - - - - - - - - - - -
iperf3-source-reverse    | [  4] 480.00-540.00 sec  45.5 MBytes  6.36 Mbits/sec    0 
```

# Certs

I use a more advanced certificate generator than the original 2-tls. See the certs service for details.

