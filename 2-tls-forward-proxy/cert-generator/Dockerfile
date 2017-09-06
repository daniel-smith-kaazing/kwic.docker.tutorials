#FROM alpine:latest
FROM openjdk:8

# Install/update openssl
RUN apt-get update -y && \
    apt-get install openssl -y

COPY entrypoint-ca2.sh /
# WARNING - use only one of the following two!
# To do a negative test with a cert that allows serverAuth only:
# Uncomment the following line.
# COPY openssl.cnf.server.only /openssl.cnf
# To do a positive test with a cert that allows both serverAuth and clientAuth:
# Uncomment the following line.
COPY openssl.cnf.server.client /openssl.cnf

RUN mkdir /x509 && chmod +x /entrypoint-ca2.sh

#CMD echo "Sleeping" && sleep 300

ENTRYPOINT ["/entrypoint-ca2.sh"]
CMD ["--help"]

# ENTRYPOINT ["/entrypoint-ca2.sh"]
# CMD ["--command",     "create-ca", \
#      "--ca-key",      "rootca.example.com.key.pem", \
#      "--ca-cert",     "rootca.example.com.cert.pem", \
#      "--ca-password", "capass", \
#      "--days",        "10", \
#      "--country",     "US", \
#      "--state",       "California", \
#      "--org",         "Kaazing", \
#      "--org-unit",    "Kaazing Demo Certificate Authority", \
#      "--common-name", "Kaazing Demo Root CA", \
#      "--overwrite",   "false" \
# ]

ENTRYPOINT ["/entrypoint-ca2.sh"]
CMD ["--command",           "create-ca", \
     "--ca-key",            "rootca.key.pem", \
     "--ca-cert",           "rootca.cert.pem", \
     "--ca-password",       "capass", \
     "--ca-days",           "1", \
     "--ca-subject-dn",     "/C=US/ST=California/O=Kaazing/OU=Kaazing Demo Certificate Authority/CN=Kaazing Demo Root CA", \
     "--server-hostname",   "example.com", \
     "--server-days",       "7", \
     "--server-subject-dn", "/C=US/ST=California/O=Kaazing/OU=Kaazing Demo/CN=*.example.com", \
     "--keystore",          "keystore.jceks", \
     "--keystore-pw",       "ab987c", \
     "--truststore",        "truststore.jceks", \
     "--truststore-pw",     "changeit", \
     "--trust-ca-alias",    "kaazingdemoca" \
]
